---
title: "Marginal vs. Conditional Subtyping"
date: 2016-06-10
categories:
  - "programming"
---

In computer programming, a type is a just a set of objects—not a physical set of objects in memory, but a conceptual set of objects. The type `Integer` is the set `{0, 1, -1, ...}`. Types are used to reason about the correctness of programs. A function that accepts an argument of type `Integer`, will work for any value `0`, `1`, `-1`, etc., for some definition of "works". Subtyping is used to define relationships between types. Type `B` is a subset of a type `A` if the set of objects defined by `B` is a subset of the objects defined by `A`. Every function that works on an instance of `A` also works on an instance of `B`, for some definition of "works". If we were just doing math, subsets would be the end of subtyping. But types as pure sets exist only conceptually. Actual types must be concretely defined. It is not very efficient to define the `Integer` class by writing all possible `Integer`s! In most programs, the possible values of a type are constrained by their fields. Subtypes and supertypes typically differ by having more or fewer fields than the other. Sometimes, the subtype has more fields, and sometimes, the supertype has more fields. Many of you may be thinking, "What? It's always one way, not the other!" The funny thing is that some of you think the subtype always has more fields and others think the supertype always has more fields. That's because there are two kinds of subtyping. The first is what I call "marginal subtyping", which is encountered in application programming and is well modeled by inheritance. The second is what I call "conditional subtyping", which is encountered in mathematical programming and is well modeled by implicit conversions. Depending on the genre of programming you work in, the other kind of subtyping may be unknown and the language features needed to implement it may be maligned. But both needs are real and both features are necessary.

<!-- more -->

<figure markdown="span">
    ![B is a subtype of A.](images/subtyping_light.svg#only-light)
    ![B is a subtype of A.](images/subtyping_dark.svg#only-dark)
</figure>

## Marginal subtyping

Marginal subtyping occurs when the programmer has one class, say `Square`, and then needs a new class, say `ColoredSquare`, that is like the old class but it has some additional functionality that is independent of the old functionality, usually in a totally separate domain. Marginal subtyping is well implemented through traditional object oriented inheritance:

```
class Square:
  x: Float
  y: Float
  width: Float
  new(x, y, width):
    self.x = x
    self.y = y
    self.width = width

class ColoredSquare extends Square:
  color: Color
  new(x, y, width, color):
    super(x, y, width)
    self.color = color
```

This is inheritance at its best. It has eliminated the need to copy the fields, methods, and constructors of Square into `ColoredSquare`. Every function that requires a `Square` can safely accept a `ColoredSquare`. The `Square` part of the `ColoredSquare` can even be mutated, the `x` and `y` and `width` fields can be changed while the `color` field is carried safely along.

I call this marginal subtyping because relationship between the subtype and the supertype is like that of a distribution and a [marginal distribution](https://en.wikipedia.org/wiki/Marginal_distribution). In marginal subtyping, the subtype has more dimensions, more degrees of freedom, more fields than the supertype. In some sense, the Venn diagram of subtyping is little misleading. There is not some universe of possible squares and then we carve out a subregion of squares and call them colored squares. It is more that there is a universe of possible squares and then there is an additional dimension, `color`, perpendicular to the diagram; the `ColoredSquare`s live in this space off the flat surface and each one can be projected down to its corresponding `Square`. It is a many-to-one mapping from subtype to supertype, with `ColoredSquare(0,1,2,Color.black)` and `ColoredSquare(0,1,2,Color.cornflower_blue)` both being interpreted as `Square(0,1,2)` whenever a `Square` is required.

## Conditional subtyping

Conditional subtyping occurs when one class is a specific kind of another class or, identically, one class is a generalization of the other. Conditional subtyping is well implemented through implicit conversions:

```
class Rectangle:
  x: Float
  y: Float
  width: Float
  height: Float
  new(x, y, width, height):
    self.x= x
    self.y = y
    self.width = width
    self.height = height

class Square:
  x: Float
  y: Float
  width: Float
  new(x, y, width):
    self.x= x
    self.y = y
    self.width = width
  implicit static def to_rectangle(r: Square) -> Rectangle:
    return Rectangle(r.x, r.y, r.width, r.width)
```

I am following Scala's implicit conversion design, where the implicit conversion can be defined in either the class being converted from or the class being converted to. By defining the implicit conversions as part of the class, it ensures that the user of the class does not have to import the conversion functions separately. Now anytime a function requires a `Rectangle`, like `intersect(rectangle1, rectangle2)`, one can actually pass in a `Square`, like `intersect(square, rectangle)`, and the implicit function will automatically be applied, like `intersect(Square.to_rectangle(square), rectangle)`.

I call the kind of subtyping required here conditional subtyping because the relationship between the supertype and the subtype is like the relationship between a distribution and a [conditional distribution](https://en.wikipedia.org/wiki/Conditional_probability_distribution). In conditional subtyping, the subtype has fewer degrees of freedom and, in an ideal world, fewer fields than the supertype. Here, the Venn diagram of subtyping is quite accurate, `Square`s really are nothing more than a special region within `Rectangle`s. `Square`s are `Rectangle`s under the condition that `width` and `height` are equal, a condition guaranteed by the `Square` constructor, assuming there is no mutability to spoil it later.

## Alternatives

Both inheritance and implicit conversions are often reviled as useless and complicated. This is not that surprising on its own. Both features are very powerful, often overused, and sadly, designed poorly in some languages. An additional difficulty is that marginal subtyping is often the only relationship model needed for application programming and conditional subtyping is often the only relationship needed by mathematical programmers. The unstoppable juggernaut of object oriented programming particularly encourages the use of inheritance everywhere, even on relationships that requires conditional subtyping. It may seem advisable to get rid of one or the other, but this leaves bad solutions for many problems, as shown below.

### Replace inheritance with membership

If one wanted to avoid the use of inheritance, one could make the `ColoredSquare` class like this:

```
class ColoredSquare:
  circle: Square
  color: Color
  new(x, y, width, color):
    self.circle = Square(x, y, width)
    self.color = color
```

But this would require one to access the circle member every time one wanted to use a `ColoredSquare` as a `Square`, writing `my_square.square.area()` instead of `my_square.area()` or `intersection(one_square.square, another_square.square)` instead of `intersection(one_square, another_square)`. Over an entire program, eliminating these "hasa" casts can save a lot of boilerplate and noise.

### Replace inheritance with implicits

One can try to replace inheritance with implicit conversions like this:

```
class Square:
  x: Float
  y: Float
  width: Float
  new(x, y, width):
    self.x = x
    self.y = y
    self.width = width

class ColoredSquare:
  x: Float
  y: Float
  color: Color
  new(x, y, width, color):
    self.x = x
    self.y = y
    self.color = color
  implicit static def to_square(s: ColoredSquare) -> Square:
    return Square(s.x, s.y, s.width)
```

The first problem with this is a large amount of repetition compared to the original above. There is no `super` constructor to which to delegate, so all fields must be set in each class. This is not that bad, because the user is also freed from _having_ to call the `super` constructor, a responsibility that can cause problems when one wants finer control over the order in which fields are initialized. There is more flexibility in this design and, while I have manually copied each constructor argument to its corresponding field in all my examples here, most modern languages have syntax that makes fields for some or all parameters of the constructor.

The second problem is fatal however. If these are mutable classes, then the object returned by the implicit conversion has no relationship to the original object. A change made to a field on the down-cast object is not reflected in the corresponding field on the original object, and vice versa:

```
val colored_square = ColoredSquare(1.0, 2.0, 3.0)
val square: Square = colored_square  # ColoredSquare.to_square(colored_square)

colored_square.x = 5.0
square.x  # Still 1.0

square.y = 4.0
colored_square.y  # Still 2.0

```

### Replace implicits with inheritance

Because of the popularity of object oriented programming languages, using inheritance where implicit conversions would be more appropriate is very common. It is also very easy to screw up. Consider this attempt to fit `Square` and `Rectangle` into an inheritance hierarchy:

```
class Square:
  x: Float
  y: Float
  width: Float
  new(x, y, width):
    self.x = x
    self.y = y
    self.width = width

class Rectangle extends Square:
  height: Float
  new(x, y, width, height):
    super(x, y, width)
    self.height = height
```

Like `ColoredSquare`, `Rectangle` has one additional field compared to a `Square`, so it becomes the child class. Programmatically, this is completely legal. Conceptually, this is wrong, wrong, wrong. Remember the diagram of a subtype from above? Subtyping must satisfy an "is a" relationship—`Rectangle` is only a subtype of `Square` if every `Rectangle` "is a" `Square`. This is mathematically wrong—the worst kind of wrong.

If I remember my middle school math correctly, a square is a rectangle with two equal sides. To do this properly with inheritance we need to flip the relationship:

```
class Rectangle:
  x: Float
  y: Float
  width: Float
  height: Float
  new(x, y, width, height):
    self.x= x
    self.y = y
    self.width = width
    self.height = height

class Square extends Rectangle:
  new(x, y, width):
    super(x, y, width, width)
```

It may seem a little strange for the subtype to add no fields, and we'll come back to that in a bit. But programmatically, this is completely legal. Conceptually, this also correct, but only if an important property holds—instances of `Square` and `Rectangle` must be immutable. That is, there cannot be a `stretch_horizontally` method that mutates an `Rectangle`, because `Square` would have to inherit it and calling such a method on a `Square` makes it no longer a `Square`. Variables typed as `Square` may not point to things that a reasonable person would consider a square:

```
val square: Square = Square(0.0, 1.0, 2.5)
square.stretch_horizontally(2.0)
# square is still type Square, but width does not equal height
```

This problem is unresolvable as long as the objects are mutable. When a rectangle is stretched in math, it is a new rectangle, but in programming, mutable objects can be changed and still remain the same object, just with a different values. The reality is that the mutable square type is disjoint with the mutable rectangle type; there is no object that is both a mutable square and a mutable rectangle. That is, there is no object that can change into any rectangle while also always remaining a square. If a mutable rectangle is the best model for your problem, by all means use it, but understand that there is no type relationship between it and the square version; they are totally unrelated classes.

Ok, so let's limit ourselves to immutable objects. Technically, the classes model the concepts they are supposed to. Something is still a little bit weird about modeling these concepts with inheritance. The problems are clearer when we go to add some more classes in this hierarchy. The rectangle can be generalized further into a parallelogram. Let's put that into the type hierarchy:

```
class Parallelogram:
  x: Float
  y: Float
  width: Float
  height: Float
  angle: Float
  new(x, y, width, height, angle):
    self.x= x
    self.y = y
    self.width = width
    self.height = height
    self.angle = angle

class Rectangle extends Parallelogram:
  new(x, y, width, height):
    super(x, y, width, height, tau/4)

class Square extends Rectangle:
  new(x, y, width, height):
    super(x, y, width, width)
```

The first problem with this is that adding a new generalization requires adding a supertype to an existing class. It would not be possible to add the `Parallelogram` class in this way if `Rectangle` and `Square` were in a different library. With PEP 3141, Python created the class hierarchy `Number >: Complex >: Real >: Rational >: Integral`. This is all correct, but if you ever wanted to make a `Quaternion` class, you'd be out of luck, because you can't "supertype" an existing class. If you wanted to add `Rhombus`, `Trapezoid`, `Quadrilateral`, or `Polygon` to our shape classes, you'd also be out of luck unless you owned the library that defined `Square` and `Rectangle`.

The second problem with this is that the simpler classes get progressively more complicated by the addition of the general base classes. When it was just `Rectangle` and `Square`, `Square` had both `width` and `height` even though it only needed one number to describe both. When `Parallelogram` was added, each class got `angle`. If we wanted to add `Polygon`, each class would need a variable length list, even though the simpler classes could be fixed-size. This wastes a ton of memory, which is why languages that do this usually turn the classes into interfaces and have only one concrete class for each interface. Python may have the `Complex` interface, but `float` doesn't have to actually have a field for the imaginary part. And there is a separate `complex` class that provides the actual implementation of `Complex`.

So modeling mathematical relationships with object-oriented inheritance means you need two classes for every concept or tons of extra state on simple classes, and you can't generalize existing classes no matter what. It is no wonder that object oriented programming is [often denigrated](https://crypto.stanford.edu/~blynn/c/object.html) for its excessive complexity and awkward boilerplate. The purists aren't wrong, traditional OOP really is bad for this kind of subtyping.

Implicit conversions make it easy to add additional generalizations. One can define `Parallelogram` separately from `Rectangle` and `Square` like this:

```
class Parallelogram:
  x: Float
  y: Float
  width: Float
  height: Float
  angle: Float
  new(x, y, width, height, angle):
    self.x= x
    self.y = y
    self.width = width
    self.height = height
    self.angle = angle
  implicit static def from_rectangle(r: Rectangle) -> Parallelogram:
    return Parallelogram(r.x, r.y, r.width, r.height, tau/4)
  implicit static def from_square(s: Square) -> Parallelogram:
    return Parallelogram(s.x, s.y, s.width, s.width, tau/4)
```

## Living in harmony

Both inheritance and implicit conversion are necessary in a programming language that intends to be universally useful because marginal subtyping and conditional subtyping both exist. In my projects, one kind or the other typically dominates. When I am writing graph algorithms, I have a lot of implicit conversions. When I am writing GUIs, I have a lot of inheritance. One thing I liked about Scala was that I could fit the algorithm and the user interface comfortably in the same program. This is probably not too surprising as the marriage of object oriented programming and functional programming was [the principle design goal of Scala](https://www.artima.com/scalazine/articles/goals_of_scala.html). If I were designing a language, I would start with Scala's design of inheritance and implicit conversion. They play together pretty well in practice, but they were not universally acclaimed. The developers recently put [user-defined implicit conversions behind a feature gate](https://docs.scala-lang.org/scala3/book/ca-implicit-conversions.html#beware-the-power-of-implicit-conversions). This may be a good idea, not because implicit conversions are bad, but because most new Scala programmers come from an object-oriented background and most programs are applications rather than math libraries. I think the problems that came from Scala's implicit conversions could have been avoided if they (1) had made them harder for new programmers to find and (2) didn't overuse implicit conversions in the standard library, of which the worst offender is the infamous and deprecated `any2StringAdd`.
