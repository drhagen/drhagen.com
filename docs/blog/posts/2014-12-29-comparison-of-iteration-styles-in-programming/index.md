---
title: "Comparison of Iteration Styles in Programming"
date: 2014-12-29
categories:
  - "programming"
---

It is difficult to overstate the importance of iteration in programming—the process performing an operation on each element of a list. I would rank only variable assignment, functions calling, and branching as more important. Unlike the `if` statement, which is essentially the same in every language, the semantics of the `for` loop varies across languages. Mainly for my own future reference, this post compares the various styles of iteration that I have come across. The examples are in pseudo code which is the only way to write so many different iteration styles under similar syntax. Each of the examples is trying to do the same thing: print out each element of a list called `list`.

<!-- more -->

There are several features that I am looking for. Most techniques are missing one or more of these.

1. Type safety. There is no way to get an exception in code that passes a static type checker.
2. Immutability. Both using and creating iterators should be possible without resorting to mutable state.
3. Generality. Arbitrary elements can be considered
4. Utility. Arbitrary collections can be iterated
5. Controlability. Arbitrary things can be done with the elements

## Indexing

```
for (i = 1; i <= list.count; i++):
  print(list(i))

# which is just sugar for
i = 1
loop:
  if not i <= list.count:
    break
  else:
    print(list(i))
    i = i + 1
```

This is an ancient form of iteration. It is also closest to the metal—that is, how computers actually do this sort of thing. Each iteration requires one comparison and one increment. Getting the next element and then checking to see if it is valid will be a recurring theme of iteration.

Its simplicity is really the only thing going for it. It has more boilerplate than a steam locomotive. And it is easy to get wrong in a way that will pass the type checker. For example, don't mix up `i < list.count` and `i <= list.count`! It depends on whether or not your language is 0-indexed or 1-indexed. You also cannot iterate over anything whose size is not (cheaply) known beforehand. For example, you cannot iterator over the lines of a file this way.

This method succeeds on #3 Generality, but fails on all other counts. It is the only method mentioned here that fails on #4 Utility.

## Throwing Iterators

```
for (element in list):
  print(element)

# which is just sugar for
iterator = list.iterator()
loop:
  try:
    element = iterator.next()
    print(element)
  catch (StopIteration e):
    break
```

This is the first of many styles that use the syntax `for (element in list)`. Here, `list` can any object that implements the `Iterable` interface. The `Iterable` interface has a single method `iterator`, which returns an instance of `Iterator`, a mutable object with a method `next`. What `next` does exactly is the difference between throwing, sentinel, option, and peeking iterators. In every case, the `for` loop calls `next` repeatedly, getting a different element of the list in turn, until the iterator is exhausted and something special happens.

Once there are no elements left in a throwing iterator, the next call to `next` throws a `StopIteration` exception. The `for` loop catches this exception and exits the iteration.

From a purity standpoint, it is rather unsanitary to be using exceptions to handle something that is not only unexceptional, but a mandatory consequence of iteration. Python uses this technique and recently had to [change some things](https://www.python.org/dev/peps/pep-0479/) because of the fact that exceptions can be throw anywhere, which leads to weird behavior like loops quietly terminating when an errant exception is thrown. This also seems like it would be the most difficult to optimize because exceptions interrupt the normal flow of the program.

This style succeeds on #3 Generality, #4 Utility, and #5 Controlability. It fails on #1 Type-safety and #2 Immutability.

## Peeking Iterators

```
for (element in list):
  print(element)

# which is just sugar for
iterator = list.iterator()
loop:
  if not iterator.has_next():
    break
  else:
    element = iterator.next()
    print(element)
```

A peeking iterator also throws an exception when `next` is called, but that is not how one is supposed to use a peeking iterator. It has a second method `has_next`, which returns `true` if `next` will return an element and `false` if the iterator is exhausted.

One disadvantage is that the implementation of all iterators requires an extra function whose only purpose is to avoid running into undefined behavior when the iterator runs out. If the next value is generated on the fly, it may be necessary to compute the value of `next`, without calling `next` of course, because that would increment the iterator and throw the value away. If the computation of next is expensive, it will be necessary to store that value after computing `has_next` in anticipation of the subsequent call to `next`.

This style succeeds in #3 Generality, #4 Utility, and #5 Controlability. After the indexing iterator, this is the worst violator of #1 type-safety. Naturally, it does not follow #2 Immutability.

## Sentinel Iterators

```
for (element in list):
  print(element)

# which is just sugar for
iterator = list.iterator()
loop:
  element = iterator.next()
  if element is Null:
    break
  else:
    print(element)
```

Rather than throw an exception when no more elements are found, a sentinel iterator returns a special value (often called `null`). Every `for` loop tests if the returned value is the sentinel value and aborts before running the next iteration if that is true. If the collection has elements of type `Element`, then the return type of `iterator.next()` is the union type `Element|Null`.

The disadvantage of this approach is that now collections cannot contain arbitrary objects, making it the only style that fails on #3 Generality. In particular, a collection cannot contain the sentinel value. Otherwise, when an iterator returns that element of the collection, it will think that the collection has been exhausted. Ceylon uses this method, which means that `{true, finished, null}.size` returns `1` because `finished` is the sentinel value.

This style succeeds on all points except #3 Generality and #2 Immutability.

## Option Iterators

```
for (element in list):
  print(element)

# which is just sugar for
iterator = list.iterator()
loop:
  option = iterator.next()
  if option is None:
    break
  else:
    element = option.get
    print(element)
```

A slight variation of the sentinel iterator is to box the response with a type that makes it possible to distinguish between the iterator being finished and the iterator happening to have a sentinel value in the list. This wrapper class is an abstract class called `Option` with two concrete classes, `Some` and `None`. An iterator returns `Some(element)` until it is exhausted and then it returns `None`. In this case, `None` is the sentinel value that causes termination of the iterations, but if the list has a `None` in it, it returns `Some(None)` which can be distinguished from plain `None`.

This style succeeds on all points except #2 Immutability.

## List Iterators

```
for (element in list):
  print(element)

# which is just sugar for
iterator = list.iterator
loop:
  if iterator is Empty:
    break
  else:
    element = iterator.head
    print(element)
    iterator = iterator.tail
```

The throwing, peeking, sentinel, and option iterators all require mutable iterators—each call to `next` causes the iterator object to be different upon the next call. Mutable state is mildly harmful to sane programming. Fortunately, most users only encounter iterators via the `for` loop, which safely hides all the mutability. The writers of generators still need to deal with mutability, however. Mutability can be completely avoided by having the iterator not only return the next value but the next iterator also. Here, there are two types of iterators `Full` and `Empty`. The `Full` instance has a `head` method that returns the next element of the iteration, but the iterator is also immutable so it returns the same value every time. The `Full` iterator has a second method `tail`, which returns a fresh iterator. That tail is also immutable, and it also is either `Full` or `Empty`. If it is `Empty`, the iteration terminates, but if it is `Full`, the iteration continues with a call to its head.

This is my favorite iterator. There is no required mutability either for creators of iterators or users of iterators. There are no weird sentinel values. There are no required exceptions and no exceptions for anything that type-checks. There is a performance issue here. Rather than just returning one element per iteration, the iteration process needs to create one new iterator object per iteration. This should be easy to optimize away as the old iterator of exactly the same type is immediately discarded. You can also safely reuse the iterator as many times as necessary, even starting in the middle using one of the intermediate immutable tails.

The one situation where this style could be a burden is when one needs an iterator that can only be iterated once and which permanently discards the elements it has produced. I only say this because Scala must have the TraversableOnce trait for a reason, though I have never uncovered it.

This style is the only one that succeeds on all points.

## Internal Iterators

```
list.foreach(element => print(element))
```

An internal iterator is just a method, often called `foreach`, that accepts a function as its only argument which is then applied to each element of the collection. It moves the control of the iteration from the caller to the callee. As a consequence, the ability to break, continue, or otherwise alter the normal iteration is lost. In Scala, even the `for` loops are sugar for internal iterators, and thus lack break or continue.

Internal iterators can coexist with an external ones. Robert Nystrom has a blog series about why [certain problems are harder or easier to solve external vs internal iterators](https://journal.stuffwithstuff.com/2013/01/13/iteration-inside-and-out/). The subsequent posts talk about [ways to get the benefits of one in the other](https://journal.stuffwithstuff.com/2013/02/24/iteration-inside-and-out-part-2/) and [more about sentinel iterators](https://journal.stuffwithstuff.com/2013/04/17/well-done/), his favorite. It is a good series for those interested in learning even more about iterators.

This is the only the style that fails #5 Controlability, but it succeeds on all other counts.
