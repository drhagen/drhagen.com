---
title: "Difficulty of Pattern Matching Syntax"
date: 2015-03-07
categories:
  - "programming"
---

Pattern matching is compact syntax, originating in functional programming languages, that acts a super-charged switch statement, allowing one to concisely branch on the types and components of a value of interest. Like a traditional switch statement, a pattern match takes a single object and compares it to sequence of cases, running the code body associated with the matching case. There are many parts to a pattern matcher. and design concise and unambiguous syntax is a difficult endeavor, one failed by many popular programming languages.

<!-- more -->

There are five things that pattern matchers can do:

1. **Equality:** match if the value is equal to another particular value
2. **Instance**: match if the value is an instance of a particular type
3. **Destructuring**: split the value up into its components and match on each one
4. **Assignment**: assign the value to a name
5. **Otherwise**: match any value
6. **Guards**: after-the-fact test an arbitrary boolean expression

## Otherwise (_)

In some sense, the default case is the simplest. This is the pattern that matches any value. It is often given its own keyword, such as `default` or `otherwise` or `else`. In other languages, it gets its own symbol, such as `*` or `_`. In languages with full pattern matching, such as Scala, Haskell, and F#, `_` is most common, which is what will be used in this post.

```
print("What happened today?")
response = read()

switch response:
  _:
    print("And how does that make you feel?")
```

Because the rest of the tests can be combined, this really represents an empty list of tests. The best syntax for this may be simply listing nothing. The nothingness starts to become a little too invisible for my taste.

```
switch response:
  :
    print("And how does that make you feel?")
```

## Equality (eq)

With equality testing, a case is a match if the value of interest is equal to the value supplied in the case. In a switch statement, this is all there is. Because there are so many kinds of tests in full pattern matching, this post will use a two letter keyword to disambiguate each kind. For the equality test, the keyword `eq` will be used.

Here is an example of a pattern match used only like a switch statement:

```
print("Pick a number between 1 and 10")
number = Integer.parse(read()).or_die();

magic_number = random.integer(1,10)

switch number:
  eq 0:
    print("I said 1 and 10, not 0 and 9!")
  eq magic_number:
    print("You guessed my magic number!")
  _:
    print("You did not guess it.")
```

## Instance (is)

The first kind of pattern matching that you will not see in non-functional programming languages is instance testing—testing that the value is an instance of a given type. By itself, this is really only useful if the language also has union types and flow-sensitive static typing. Say variable `x` has type `Integer|String`. If a case tests for `x` being type `String`, then the body will only be executed when `x` actually is a `String`, making it safe to assume `x` has a static type of `String` within the body of the case. Ceylon, with its flow-sensitive typing and union types, is the only language I have found that has [this kind of test](https://web.archive.org/web/20141231224452/https://ceylon-lang.org/documentation/1.1/reference/statement/switch/), but not full pattern matching. This post will use the keyword `is` to indicate instance tests.

```
def double(x: Integer|String):
  switch x:
    is Integer:
      return 2*x
    is String:
      return x.concatenate_back(x)
```

It is rarely useful to test with `eq` and `is` in the same expression, because the type of the value being tested with `eq` is usually known. This assumes that the pattern matcher does not throw an error when comparing two incomparable types like String and Integer. In the example below, `is Integer` is pointless if `eq 0` is false for all values of `x` that are not `Integer`s. With implicit conversions, this could occasionally not be true.

```
switch x:
  # Don't write code where `is Integer` is meaningful here
  is Integer eq 0:
    print("It's really zero")
  _:
    pass
```

## Destructuring (to)

A quintessential part of pattern matching is destructuring. Destructuring breaks the value into components and then pattern matches on each of them. It is the recursive construct of pattern matching, but is not really a test in itself. The entire pattern matches only if each component matches its pattern. The value must be of a type that can be destructured. It is a type error to attempt to destructure a value that cannot be interpreted in this way, so usually destructuring is preceded by an instance check. This post will use the construct `to (...,...)` to perform destructuring.

Note the patterns testing for equality inside the destructuring construct.

```
class Point(x, y)
origin = Point(0.0, 0.0)

class Line(b, m)

if random.boolean():
  shape = Point(3.0, 0.0)
else:
  shape = Line(0.0, 1.0)

switch shape:
  eq origin:
    print("Point at origin")
  is Point to (eq 0.0, _):
    print("Point on x-axis")
  is Point to (_, eq 0.0):
    print("Point on y-axis")
  is Line to (eq 0.0, _):
    print("Line through origin")
  _:
    print("Something boring")
```

## Assignment (as)

Assignment is not a test at all, but binds the value of a component and assigns it to a name. It is nearly useless without destructuring, because the base value is the value being tested, which is usually assigned to a name already. The values bound are usually the components of a destructured value.  This post will use the keyword `as` to indicate a pattern of binding a value to a name. Other possible keywords are `val` or `let`.

The values bound as usually referenced in the body of the case.

```
switch shape:
  eq origin:
    print("At origin")
  is Point to (eq 0.0, as y):
    print("On x-axis at {y}")
  is Point to (as x, eq 0.0):
    print("On y-axis at {x}")
  is Line to (eq 0.0, as m):
    print("Line through origin with slope {m}")
  _:
    print("Something boring")
```

## Guards

Guards can seem like an afterthought to pattern matching because their behavior is so orthogonal. Guards are the front half of an if statement appended onto the end of a case. The case is only a match if the expression following the `if` keyword evaluates to `true`. Usually, the guards reference variables bound via `as`. Guards are strictly more powerful than `eq` alone because it allows us to compare destructured components to each other or switch on the members of components or apply arbitrary functions to components such as this:

```
switch shape:
  as point is Point if distance_from_origin(point) > 1:
    print("Point outside unit circle")
  is Point to (as x, as y) if x == y:
    print("Point on 45 degree radius")
  _:
    pass
```

## Other Uses of Pattern Matching

Other than in a switch-like construct, pattern matching is used in overloading and variable assignment.

Some programming languages use pattern matching to define overloaded functions. Once famous example is using it to define the factorial function. Because there is no inherent order to the overloaded methods, there needs to be some sort of overload priority resolution that says that `factorial(0)` calls the correct method even through `0` matches both.

```
def factorial(0):
  return 1
def factorial(n):
 return n * factorial(n-1)
```

Note that this is just syntactic sugar for a single, non-overloaded function that pattern matches its parameters.

```
def factorial(n):
  switch n:
    eq 0:
      return 1
    _:
      return n * factorial(n-1)
```

Some programming language also allow values to be pattern matched on assignment. Even Python, which [does not even have a switch statement](https://peps.python.org/pep-3103/), allow [tuple destucturing](https://wrobstory.gitbooks.io/python-to-scala/content/tuples/README.html) like this: `x, y = my_tuple`. When doing an assignment it never makes sense to have tests (what would happen if they failed?). Therefore, assignment pattern matching is restricted to destructuring,  assignment, and otherwise.

```
# Swap
to (x, y) = [y, x]

# Extract just the x
to (x, _) = point
```

## Simplification

If you have ever used pattern matching in programming, you will notice that this syntax contains a lot more keywords than anything you have ever seen. There are six keywords. Can that be reduced a little bit? Is there any redundancy in the syntax? The short answer is that it can be reduced by one keyword without losing features or becoming ambiguous. The long answer is that most programming languages reduce it even more resulting in ambiguity and complex rules to resolve the ambiguity.

Without losing any ambiguity, we can allow one kind of shorthand when an expression starts a pattern without a keyword. How is this bare expression interpretted so that the code can be made terser? Below are some possibilities.

### eq only

In C#, C++, and other non-functional languages, the switch statement only does equality testing, so a keyword is not required.

In Ceylon, the switch statement does not do destructuring, assignment, or guards and [assignment only allows destructuring and assignment](https://web.archive.org/web/20221225101328/https://ceylon-lang.org/blog/2014/12/29/destructuring/). In Ceylon, a bare expression in a switch is interpreted as an equality test and a bare expression in an assignment is interpreted as an assignment.

### eq on literals

In every language I have seen, `eq` is dropped on literals. Because `is 0.0` and `as 0.0` make no sense, we know that a bare `0.0` must refer to `eq 0.0`. In fact, most languages provide no mechanism for equality testing other than literals—the general `eq` operation does not exist. Languages with full pattern matching allow guards to be used in place of equality testing against variables; a value to be tested is captured in a name and then tested for equality in the guard.

This may not be a good idea for one special reason: referential transparency. Referential transparency means that an expression can be replaced by its value. In a referentially transparent language, `f(2.0)` is always the same as `x=2.0; f(x)`. If we allow for the `eq` to be dropped before literals, then replacing those literals with a name of the same value will result in an error. Being able to rely on referential transparency makes it easier to reason about how a programming is working. Nevertheless, dropping `eq` on literals is a popular choice. Scala, F#, Haskell, and Rust all sacrifice referential transparency to make matching literals easier.

The following two definitions are equivalent, showing how `eq` can be eliminated entirely.

```
def is_zero(x: Integer):
  eq 0:
    return true
  _:
    return false

def is_zero(x: Integer):
  zero = 0
  as temp if temp == zero:
    return true
  _:
    return false
```

### as outside pattern matching

Programming languages that allow pattern matching in assignment usually have bare identifiers interpreted as assignments. Instance testing, equality testing, and guards make no sense in this context so there no chance of ambiguity.

### is and to combo

It may be tempting to drop `to` also in addition to `eq` or `is` or `as`, especially if `(something, something)` is not a legal expression in the language. This cannot be done ambiguously, because the one-element destructuring `(something)` is undoubtedly legal syntax as simple unnecessary parentheses. It is clear that `(A, B)` means `to (A, B)`, but does `(A)` mean `to (A)` or `is A`? Technically, it is the parentheses that are unnecessary, but who wants to write `Point to x, y`?

We could take advantage of the fact that destructuring is almost useless without an instance check beforehand. Let's try a hybrid approach and interpret all patterns starting with `Name(...)` as `is Name to (...)`. This is not the same as dropping `is` entirely because `Name(...)` could be a function call returning either a type, unless types are not first class objects in the language and cannot be returned from functions.

This syntax is almost universal among functional programming languages because it is almost entirely how algebraic data types are used in pattern matching. For example, here is how one would use the `Option` data type:

```
print("What's your favorite number?")
integer = Integer.parse(read())
# Type of integer is Option[Integer]

switch integer:
  Some(7):
    print("That's my favorite number too!")
  Some(as value):
    print("I am ok with {value}")
  None:
    print("That's not a number!")
```

### lowercase as and uppercase is

By using the `is`/`to` combo and restricting `eq` to literals and guards, we can get rid of the `is`, `to`, and `eq` keywords, leaving only `as`. Even with `eq` gone, we cannot get rid of `as` without introducing ambiguity. When we write `A`, does that mean `is A` or `as A`. Nevertheless, most of the full-featured pattern matching languages eliminate it, introducing some of the weirdest corner cases in programming history. They parse a lowercase identifier (like `name`) as a binding and parse an uppercase identifier (like `Name`) as an instance test. In languages where types must begin with a capital letter, like Haskell and Ceylon, it is not a big deal. However, in languages where this is the only place where capitalization matters, like Scala and F#, [it](https://lampwww.epfl.ch/~michelou/scala/scala-pitfalls.html) [is](https://stackoverflow.com/questions/7078022/why-does-pattern-matching-in-scala-not-work-with-variables) [particularly](https://stackoverflow.com/questions/4479474/scala-pattern-matching-with-lowercase-variable-name) [jarring](https://web.archive.org/web/20181221065731/https://mergeconflict.com/scala-rage-pattern-matching/).

### type is and object eq

One possibility that I have never implemented is to switch between the behaviors of `is` and `eq` based on whether or not the pattern is a type. If the name refers to a type, then it is likely the user intends to test if the value is an instance. Testing if a value is equal to a type is pretty rare. Similarly, if the name refers to an object that is not a type, then it is likely the user intends to test for equality, as an instance test would be an error.

```
a = 0
b = Integer

switch obj:
  a: # eq a
    print("Found the zero")
  b: # is b
    print("Nonzero")
```

This can be interpreted as requiring a type and always doing an `is` comparison, but having an implicit conversion from any object to a singleton type. It is a good principle to avoid general implicit conversions because it easily masks errors. Scala [learned this the hard way](https://github.com/scala/bug/issues/194) by including a conversion from Any to String.

This could also be done based on the runtime type of the pattern, but that would be even weirder. A pattern that is usually compared via `eq` would suddenly compare via `is` if the object happened to also be a type.

### type is and object eq and undefined as

An extension to the previous technique would be to use `as` whenever the name was undefined, because that is the only right thing to do when the name has no value.

```
a = Some
b = none # an object

switch obj:
  a(c): # is a to (as c)
    print("Some contains {c}")
  b: # eq none
    print("None")
```

This solution has the additional problem that defining a top-level class or other object would silently and subtly alter the behavior of pattern matches using that name as assignment. For saving characters, this solution is a dream. For saving one's sanity, not so much.

## Conclusions

It is kind of difficult to render a judgment on this one. The full syntax is too verbose for simple cases, but the choices made by other programming languages lead to weird corner cases. I think the `is`/`to` combo is a great choice for the behavior of a bare expression. And despite the loss of referential transparency, I think equality should be applied to literals. This is also because literals interact differently with an exhaustiveness checker than variables with the same value. An [exhaustiveness checker](https://fsharpforfunandprofit.com/posts/correctness-exhaustive-pattern-matching/) is something that I did not talk about, but it is an important aspect of pattern matching. I think all six keywords should be retained to allow for unambiguous definitions when necessary.

Testing for equality of non-literals is actually pretty rare (because it interacts poorly with exhaustiveness checking), so shortening the syntax for it seems wasteful. In fact, by having the `eq` keyword, I already make it shorter than most other languages which require that equality testing be done with guards.

Under no circumstances would I choose to have the case of the first letter be significant. Users of any language I design will have to suck it up and use a keyword to assign a value. I am not sure why this is considered so burdensome. In all languages with both block scoping and mutability, the programmer has to use a keyword (like `val` or `var`) to declare every other variable in the program. Why not those declared in pattern matchers? In my world, pattern matching would look something like this:

```
switch shape:
  eq origin:
    print("At origin")
  Point(0.0, val y):
    print("On x-axis at {y}")
  Point(val x, 0.0):
    print("On y-axis at {x}")
  Point(val x, val y) if x == y:
    print("Point on diagonal")
  Line(0.0, val m):
    print("Line through origin with slope {m}")
  _:
    print("Something boring")
```
