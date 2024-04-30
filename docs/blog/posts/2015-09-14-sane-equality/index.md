---
title: "Sane Equality"
date: 2015-09-14
categories:
  - "programming"
---

Equality: every programming language has it, the `==` syntax is as universal as `1+1`, and it works almost the same in every language. When the left and right operands are the same type, equality is easy. No one questions that `1==1` evaluates to `true` or that `"a"=="b"` evaluates to `false`. This post is about what to do when the operands are different types. What should `1=="1"` be? Or what should `Circle(1,2,2)==Point(1,2)` be? Or what should `ColoredPoint(1,2,red)==Point(1,2)` be?

<!-- more -->

## Defining Equality

When defining how equality should work, it is helpful to think about what equality means. If two values are equal, that means that one value can be substituted for the other in _some_ sense. Some programming languages flagrantly disobey this rule; they will say that two values are equal even if they cannot be used in the same way at all. The classic member of this club is Javascript, which [returns equals for numerous unrelated values](https://dorey.github.io/JavaScript-Equality-Table/).

It may be tempting to restrict equality to absolute indistinguishability—two objects are equal only if they can replace each other in every possible case. And for immutable values without inheritance, this is a clean choice with little drawback. It can be generated automatically for user-defined types. But as I'll show later, mutable objects and values satisfying multiple types may actually need several parallel definitions of equality. It is often not sufficient to ask if two objects are equal. One must ask: equal in what way?

## Design Options

When two different types are compared, there are a few options available to the language designer:

1. Evaluate to `false`
2. Convert them to a common type
3. Compare the bits in memory
4. Throw an error
5. Return a sentinel value

## Unsafe equality

Most of the professional managed programming languages like Python, Java, C#, Scala, and Ceylon take option #1 all say that `1!="1"`. In a naive sense, this is entirely correct because an integer and a string are not even comparable. But if they are incomparable, why allow this construction at all? This is not a novel observation. This is a [common](https://stackoverflow.com/questions/19742527/why-has-scala-no-type-safe-equals-method) [source](https://rickyclarkson.blogspot.com/2006/12/making-equalsobject-type-safe.html) of hidden bugs, especially annoying in statically typed languages, which otherwise catch an incorrect mixing of types. Some IDEs will mark these with a warning, lessening the problem.

Returning `false` on incomparable types implies that equality is defined for all types. In these languages, the `Anything` type has an `equals` method and usually defaults to reference equality for user-defined classes.

Many scripting languages, like Perl, R, and Javascript take option #2 and say that `1` cannot be compared to `"1"`, but they convert the number to a string and then compare the strings, giving `1=="1"` but `1!="2"`. This is worse than always returning `false`. If equality is to mean anything, it has to mean that `a==b` implies that `a` and `b` can be used in place of each other in some sense. But a `"1"` cannot be used in the same way as a `1`. It is true that `1+1` is `2`, but `"1"+"1"` is not `2`. At least, [it better not be](https://www.xkcd.com/1537/)!

In rare cases, programming languages like Matlab will allow two types to be compared as if they are the same type even if they are not. They see option #3 not as a pitfall, but as an hammock. In Matlab, `1!="1"` but `49=="1"`, because `"1"` is the 49th ASCII character. It should not come as surprise to anyone following this blog, that I think such a design is an abomination. It will get no further attention here.

## Type-safe equality

Comparing things of different types requires that the types be available at runtime. Languages that are statically compiled without types, like C, C++, and Rust, must choose option #4 and fail to compile. This is called type-safe equality and it prevents comparing things that cannot be compared.

Assuming that we want a trait to define the interface for equality, there are several ways to design it. One way is with an instance method that only accepts an argument of the same type as the implementing class:

```
trait Equatable[in Self <: Equatable[Self]]:
  # Assume that == and != somehow desugar to equals and not_equals
  formal def equals(that: Self) -> Boolean
  def not_equals(that: Self) -> Boolean:
    return not equals(that)

class Point(x: Float, y: Float) extends Equatable[Point]:
  actual def equals(that: Point):
    return this.x == that.x and this.y == that.y

class Circle(x: Float, y: Float, r: Float) extends Equatable[Circle]:
  actual def equals(that: Circle):
    return this.x == that.x and this.y == that.y and this.r == that.r
```

[Like all shape traits](https://rosstate.org/publications/shapes/), it is up to the user to correctly pass the current type as the type parameter. Rust, which uses this design, also magically defaults the type parameter to the current class, which is a nice touch.

This provides a totally type-safe equality:

```
# All of these are true
Point(1,2) == Point(1,2)
Circle(1,2,2) == Circle(1,2,2)

# These all are compile errors
Point(1,2) == Circle(1,2)
Circle(1,2) == Point(1,2)
Point(1,2):Anything == Circle(1,2):Anything
```

One cannot accidentally compare two objects that are incomparable.

An alternative solution is accept option #5 and return a sentinel value, say `null`, when two objects are incomparable. This is also type-safe assuming that the `if` statement won't accept `null` (which it shouldn't!). This forces the programmer to recognize the incomparable case, while still leaving the possibility of converting the `null` into a `false` if he wants to. Some languages, like Ceylon, have a nice operator for converting `null`s called `else`. One could write `1=="1" else false` to recover the type-unsafe behavior, but in a type-safe way. The formal method `equals` above assumes that it is type-safe. If I provided a `null`-returning equality, I would bake it into the trait so that a user extending `Equatable` would only have to write one method.

```
trait Equatable[Self <: Equatable[Self]]:
  def equals_or_null[That](that: That) -> Boolean|Null:
    if That <: Self:
      return this == that
    else:
      return null
```

Note that this method grabs the static type of `that` and uses it to see if normal equals would work. Not all languages allow the type parameter to be queried at runtime like this. Ceylon allows it, but Scala does not. Some magical type computation could probably ensure that the return type was `Boolean` rather than `Boolean|Null` whenever the types were comparable, but that is beyond the scope of this post.

## Equal in what way?

One thing that is overlooked by every programming language I have used is how inheritance interacts with equality. Conceptually, when `B` inherits `A`, objects of type `B` now have two kinds of equality. They can be equal as `B`s or they can be equal as `A`s.

Let's add another class, this time with inheritance involved:

```
class ColoredPoint(x: Float, y: Float, color: Color)
    extends Point(x,y) and Equatable[ColoredPoint]:
  new def equals(that: ColoredPoint):
    return this.x == that.x and this.y == that.y and this.color == that.color
```

The `new` keyword works like it does in C#; it introduces a new method to hide an existing one. The hiding is partial; the new method is called only if the static type of the object and the argument are both `ColoredPoint`.

Before we continue, let's remember what inheritance is for. Inheritance allows us to pass in instances of `ColoredPoint` to code that is expecting instances of `Point` and that code needs to work exactly like it had received a real instance of `Point`. In most languages, the equality operator is a virtual method, which means that this is true:

```
# In most languages, but not the one espoused here
ColoredPoint(1,2,colors.red):Point != ColoredPoint(1,2,colors.blue):Point
```

Even when the static types of the objects are Points, they are not compared in their `Point`ness, but in their `ColoredPoint`ness. This matters because code that receives two `Point`s is expecting their equality to be determined entirely by the values of `x` and `y`, not some totally unrelated property that may not have even been defined when the code was first written.

If it is a virtual method, it is also not symmetric:

```
# In most languages, but not the one espoused here
ColoredPoint(1,2,colors.red) != Point(1,2)
Point(1,2) == ColoredPoint(1,2,colors.red)
```

This happens because each `equals` accepts `Anything` and then tests is the type is compatible for testing. When the method is on a `Point`, it is compatible with accepting a `ColoredPoint`, but not vice versa.

By using method hiding, my implementation has the expected behavior:

```
# narrow
ColoredPoint(1,2,colors.red):Point == ColoredPoint(1,2,colors.blue):Point

# symmetric
Point(1,2) == ColoredPoint(1,2,colors.blue)
ColoredPoint(1,2,colors.blue) == Point(1,2)
```

Doing the explicit casting in the narrow example above is rarely used. Where this really matters is functions that expect an instance of the base class like:

```
def do_points_overlap(p1: Point, p2: Point) -> Boolean:
  return p1 == p2
```

The typical behavior does not do the right thing here if `p1` and `p2` are `ColoredPoint`s.

## Mutable equality

In an inheritance tree, which version of `equals` gets chosen is determined by the static type we are currently working with. But mutable objects have two kinds of equality, reference equality and value equality. These two mutable arrays `a1 = Array(1,2,3)` and `a2 = Array(1,2,3)` have equal values, but are not indistinguishable. For all operations that read from the arrays, they are the same, but operations that write to the arrays reveal that `a1` and `a2` are not the same object. Many programming languages provide a separate operator, say `===`, to do reference equality. This operation can be, and probably should be, baked into the language. There is exactly one correct way to compare references (their memory pointers are equal) and it requires information often hidden from the user (their memory pointers).

The language could provide a trait that implements reference equality:

```
trait Identifiable[Self <: Identifiable[Self]] extends Equatable[Self]:
  actual def equals(that: Self) -> Boolean:
    # magical implementation

class MovablePoint(var x: Float, var y: Float)
    extends Identifiable[MovablePoint]:
  pass
```

Here there is no special syntax. It is a simple extension of `Equatable`. If one's type does not implement its own `equals` method, it will inherit it from `Identifiable` so that `MovablePoint(1,2) != MovablePoint(1,2)`. If it implements it's own `equals` to do value equality, we can recover the reference equality by casting to the `Identifiable` trait like this: `MovablePoint(1,2):Identifiable[MovablePoint] != MovablePoint(1,2)`.

Downcasting to `Identifiable` every time we want reference equality is very inconvenient. A better solution would be to move `Identifiable` out of the `Equatable` hierarchy entirely. Sure, reference equality can be used as the main equality, but that is not necessarily how the user wants it. Let the user specify how he wants equality to work on his object.

```
trait Identifiable[Self <: Identifiable[Self]]:
  # Assume that === desugars to identical
  def identical(that: Self) -> Boolean:
    # magical implementation

class MovablePoint(var x: Float, var y: Float)
    extends Identifiable[MovablePoint] & Equatable[MoveablePoint]:
  actual def equals(that: MoveablePoint):
    return this.x == that.x and this.y == that.y
```

We could have chosen to delegate to reference equality, but didn't. We now have two operators `==` and `===` that do different things for this type. We have chosen to have `==` on immutable objects and mutable objects do value comparison. In one way, this is a good choice because the user likely expects the operator to do the same thing. In another way, this is a bad choice because now `==` is not a stable property of mutable objects. It cannot be used in contexts where the equality must be stable, such as in sets, dictionary keys, or hash tables. Perhaps this is ok as long as `==` is used for transitory comparisons and the correct operation is aliased to a different operation, say `stable_equals`, which is used in collections. The language would be wise to provide metaclasses for immutable and mutable objects that implement this all correctly.

## Equality is everywhere

And speaking of collections, equality is not just about the behavior of the nice syntax. If it was just about the behavior of `==`, then we could just provide all the possibilities: `=!=` for an equals that returned `false` and `=?=` for an equals that returned `null`. The issue is so much bigger because equality is baked into collections, like `Sequence` and `Set`.

Consider `Set` first: all the elements are unique; none of the elements are equal to each other. But this begs the question, equal in what way? If I have a `Set[Point]`, I am probably expecting that `ColoredPoint(1,2,colors.red)` and `ColoredPoint(1,2,colors.blue)` are not both in there because those are equal as `Point`s. In Scala and languages like it, they both most certainly can be in there.

Consider the `contains` method on `Sequence`. The signature in Scala and friends is `contains(element: Anything)`. Here is the type-unsafe equals rearing its ugly head again. You can have `List(1,2,3).contains("foo")` quietly evaluate to `false`. ([Famously in Scala](https://scalapuzzlers.com/#pzzlr-040), `List(1,2,3).toSet()` will also evaluate to `false` due to a confluence of design flaws, one of which is type-unsafe `equals`.) A better design of `contains` takes a type parameter that answers that question: equal in what way?

```
class Sequence[Element]:
  def contains[That <: Equatable[Element]](that: That):
    for element in this:
      if element == that:
        return true
    return false
```

Note that `That` must be a supertype of `Element`, which is implied by it being a subtype of the contravariant `Equatable[Element]`. You can ask a sequence of colored points if it contains a particular point, but you can't ask a sequence of points if it contains a particular colored point, because its elements don't necessarily understand color.

## Unions require special care

One place where type-safe equals gets a little messy is when dealing with unions, like `Circle|Point`. Consider the following setup:

```
val circle:Circle|Point = Circle(1,2,2)
val point:Circle|Point = Point(1,2)
```

We would expect that `circle` and `point` could be compared because they have the same static type. However, neither `equals` method will accept an argument of type `Circle|Point`, because neither class knows how to deal with the other.

The solution is to remember that `Union` is itself a class. It is nice when it is treated specially by the language, but as a container type, it can have its own methods. The `equals` method on `Union` needs to check if the dynamic types fall into the same branch. If they don't, return `false`; otherwise, dispatch to the branch that they satisfy.

```
class Union[Left, Right](left: Left, right: Right) extends Equatable[Left|Right]:
  actual def equals(that: Left|Right):
    switch [this, that]:
      [Left&Right, Left&Right]:
        return this as Left == that as Left and this as Right == that as Right
      [Left, Left]:
        return this == that # this and that now have static type Left
      [Right, Right]:
        return this == that # this and that now have static type Right
      _:
        return false
```

The first condition covers the possibility that the arguments satisfy both `Left` and `Right`. Without this case, the union would be asymmetric. If `this` and `that` satisfied `A&B`, then `this:A|B==that:A|B` would call `A.equals(A)` while `this:B|A==that:B|A` would call `B.equals(B)`. My version always calls them both, though in different orders, but that is less problematic.

## Conclusions

The issue of syntax, such as whether `==` should ever mean value equality or whether `==` should always be a stable equality, is secondary to the problem of building a hierarchy of sane equals. I think that there should be traits with names like `Equatable` for value equality, `Identifiable` for reference equality, and `Collectible` for stable equality. Each of these should have members `value_equals`, `reference_equals`, and `stable_equals`. They are defined for whatever classes on which they make sense.

Once that is established, then we decide what to do with the syntax. In all the code where it really matters, the method name should be used directly. The `==` operator should point to whichever method is the most intuitive equality for that type. I think that `===` should point to `reference_equals`. I am on the fence about whether or not `value_equals` should get its own operator; it really depends on how many mutable classes I think should have `==` point to reference equality. I definitely don't think that `stable_equals` should gets its own operator because it should only be used with collections libraries, not user code.

And as for what to do with incomparables, I would throw an error for the main operators, but provide methods on the respective traits for returning `false` and `null`. Maybe not even methods on the traits, but as obscure library functions.

As difficult as this may be to believe, there is much I have left unsaid on this topic. Like all recursive types, `Equatable` and friends have deep problems integrating into a static type system, particularly because it is easy to get infinite recursion when dealing with generic container types (something I ignored while talking about union types). Ross Tate has an [interesting solution to this problem](https://www.cs.cornell.edu/~blg59/resources/doc/effing-bound-polymorphism.pdf), but I am not qualified to judge it.
