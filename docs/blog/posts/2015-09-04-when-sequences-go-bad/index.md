---
title: "When Sequences Go Bad"
date: 2015-09-04
categories:
  - "programming"
---

In my [last post](/blog/one-sided-debate-over-sequence-syntax/), I talked about the various kinds of syntax for getting and setting elements in sequences. This post will talk about semantics. What exactly should `get` and `mutate` do when invoked? What should happen when the index is valid is hopefully obvious. But because we have to handle the case of an invalid index—in particular, an index larger than the length of the sequence, the answer is not as clear-cut as it may seem. If "throw an exception" is the only thing that comes to mind, you have been stuck in procedural programming for too long.

<!-- more -->

## Return value of `get`

Whether mutable or immutable, the getting process is the same. Let's ignore the possibility that `seq(i)` is not the right tool for your job and that something like `map` or a `for` comprehension would be better suited for the problem. For many reasons, using an index to get an item is necessary. Unless your programming language is either not Turing complete or its compiler has solved the halting problem, there is always the possibility that getting an item via its index will be outside the bounds of the sequence at runtime. What should happen, ignoring the infamous return-a-random-value-from-memory?

1. Throw an exception
2. Return null
3. Return an optional value

I am not a fan of using exceptions for non-exceptional cases. And an index being out-of-bounds is not exceptional—it is to be expected. An exception makes it too easy to ignore the missing case while writing the code. The type system, or at least the design, should encourage handling the out-of-bounds case. This is how Scala does it, probably for nostalgia purposes given its otherwise attention to safety.

Returning `null` is an attractive idea. It is how Ceylon does it. For a collection of type `Seq[A]` the return type of `get` is `A|Null`, which forces the user to handle the `null` case. But there is minor difficulty with this: what if the sequence itself contains `null`? If I ran `seq(i)` and get back `null`, how could I tell the difference between there being a `null` at point `i` in the sequence and `i` being larger than the length? I couldn't—the value is `null` is both cases. Maybe it doesn't matter because I want to handle both cases the same. Maybe the collection is typed so that it can't have `null`s. But if I do care and it can happen, then I am stuck. This makes this a suboptimal choice for a collections library, which needs to handle sequences of arbitrary objects.

We can escape the confusion of `null` by returning a optional value—returning `Some(value)` if we found a value and returning `None` if not value was found. Notice how a sequence containing a `None` is handled seamlessly. If `seq(i)` is `None`, then `Some(None)` is returned, which is different from `None` itself. Like in [my iterators post](/blog/comparison-of-iteration-styles-in-programming/), I would go with this, the safest option.

## Return value of `mutate`

Some languages (Ceylon, Haskell) don't even have `mutate` syntax. It is not a construct of functional programming, so it is difficult for me to have strong of feelings about it.  It is always called for its side effect—mutating the sequence—rather than for its return value. Nevertheless, having a return value is not unheard of, so what should it be?

1. Required to have no return value
2. Required to have a return value
3. Not required either way

In many languages, mutation is a built-in construct, not syntactic sugar over a method of the sequence. This means that the return value and not just the return type of mutation is defined by the language itself and not just a constraint or convention. In C, C++, Java, and C#, the value of mutation is always the element, the right-hand side. In Matlab, alteration is a statement with no value at all. Most languages seem to not care either way. Python, whose mutation syntax `array[1] = 2` is a statement with no value, allows user implementations of the desugared method `__setitem__` to return any value, which is quietly discarded, though the convention appears to be to return`None`. I have yet to find a language that both let's its users define their own mutation methods, yet constrains those implementations to return no value (`void` or `Unit`) or to return a particular value (`Element` or `Seq[Element]`).

If I were designing a language, I would discard the return value of `mutate` so that it formed a statement, like Python. If someone really needed that return value, then he could call the underlying method the normal way.

Let's assume that a language does want to return a value. If so, what should that value be?

1. The mutable sequence if it succeeded and throws an exception if it did not
2. The element if it succeeded and throws if it did not
3. The mutable sequence and silently does nothing if the mutation is not legal
4. The element if it succeeded and silently does nothing if the mutation is not legal
5. An option containing `Some(element)` if it succeeded or `None` if it failed
6. An option containing `Some(element)` if it failed or `None` if it succeeded ("Couldn't put it in, here's your element back!")
7. A flag (`True` or `False`) indicating whether or not the mutation succeeded

I don't like the throwing options #1 and #2 for the same reasons I don't like it in `get`—this is not exceptional behavior. The silent options (#3 and #4) are type safe but even worse. I can't imagine trying to debug something like that.

There is something both elegant and pointless with returning the `Option`. #5 quite symmetric with the `get` and there is something cute about #6, but echoing the object just passed in undeniably redundant. The first four options (echoing the mutable sequence or the element) are also redundant, but they mirror what the C languages do.

The only choice that is neither redundant nor unsafe is #7, though I have never seen this design in action. It likely because the languages that care about safety that comes from removing exceptions also care about the safety that comes in removing mutability. In some sense, I have to agree with this sentiment. Maybe we really shouldn't be using mutable sequences anyway. If you are using them, this had better be an exceptional case already. Return nothing on success; throw an exception on failure; don't forget to check the index beforehand.
