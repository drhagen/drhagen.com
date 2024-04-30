---
title: "Superior Pairing Function"
date: 2018-03-27
categories:
  - "programming"
---

Given two sequences of objects, it is often desirable to generate a sequence which is all possible pairwise combinations of those sequences—the Cartesian product. If the sequences are finite in length, then it is a trivial function to write in any programming language. The function even exists in many standard libraries and packages, such as `itertools.product` in Python. But if the sequences are infinite in length (that is, they are streams rather than arrays, depending on your terminology), the typical approach fails. Finding a way to iterate over all pairs of two infinitely long sequences is called a "pairing function" in mathematics and has practical uses. There are some existing pairing functions, but many have limitations. I describe the properties of a superior pairing function and a couple of methods that satisfy them.

<!-- more -->

### Finite Cartesian product

Before considering pairing functions for infinite streams, let's consider them for finite streams first. Technically, it is not a pairing function if it does not work on infinite streams, but for illustrative purposes, I'll use the same terminology to describe any function that takes two indexes as its arguments and returns the index assigned to that pair. This is a fairly simple task for finite streams, and it illustrates the difficulties with handling infinite streams.

Whether dealing with the finite case or the infinite case, there exist different methods for indexing each pair. For example, with a 3 by 2 matrix, column-[major ordering](https://en.wikipedia.org/wiki/Row-_and_column-major_order) will say the entry in the second row and second column is the 5th element, while row-major ordering will say it is the 4th element. As long as each method is a one-to-one mapping of `(x, y)` to `i`, then it is a valid pairing function. In fact, there is a symmetry between the `x` and `y` values. One could swap the `x` and `y` values in any method and still have a valid pairing function (column-major and row-major ordering are mirrored in this way). (In the literature, a method may be described as one version or the other. In this post, I will always choose the version that starts out `[[0,0], [1,0], ...]` rather than the one that starts `[[0,0], [0,1], ...]` because that is [the one Wikipedia picked](https://en.wikipedia.org/wiki/Pairing_function).)

Each method is a single one-to-one mapping, but there are three programmatic functions that we might need depending on how we want to get values out of it:

- **Pairing** function `(Index, Index) -> Index` which takes two indexes into streams and returns the index into the Cartesian product stream
- **Unpairing** function `(Index) -> [Index, Index]` which takes an index into the Cartesian product stream and produces two indexes into the individual streams
- **Cartesian product** function `(Iterable[A], Iterable[B]) -> Iterable[[A, B]]` which takes two streams and produces a stream of all possible combinations

For simplicity of math, I will be using 0-indexing for all of the pairing and unpairing functions even though 1-indexing may be more natural. In languages with 1-indexing, simply add `+1` and `-1` where needed. For simplicity of implementation, I'll use Python generator syntax for creating the Cartesian products even though [other Iterable designs](/blog/comparison-of-iteration-styles-in-programming/) may be better for general programming.

I will be making use of 2D plots to illustrate the pairing functions. The index into one stream will be `x` and the index into the other stream will be `y`. The possible pairs of indexes will be laid out as points on the grid. A pairing function can be thought of as stepping from one point to the next. By requirement of being a one-to-one mapping, it must land on each point exactly once—no more, no less. The order that each method steps through the points is what makes the method special, so they will be labeled with both numbers and arrows.

## Box pairing function

If I have two finite lists of objects, the naive solution to the Cartesian product is to simply create a nested loop, pairing each element of the first list with the first element of the second list, then pairing each element of the first list with the second element of the second list, and so on. Intuitively, this walks down one complete row or column of a matrix (depending on your orientation) and then walks down the next row or column. This finite Cartesian product is illustrated in the figure below, where each of the possible pairs from a list of length 3 and a list of length 2 are labeled 0 to 5.

<figure markdown="span">
    ![Box pairing](images/box_pairing_light.svg#only-light)
    ![Box pairing](images/box_pairing_dark.svg#only-dark)
</figure>

```python
def box_product(stream1: Iterable, stream2: Iterable):
  for element2 in stream2:
    for element1 in stream1:
      yield [element1, element2]

def box_pairing(length1: Integer, length2: Integer,
    index1: Index0, index2: Index0):
  return length2 * index1 + index2

def box_unpairing(length1: Integer, length2: Integer, index: Index0):
  index1 = floor(index / length2)  # integer division
  index2 = mod(index, length2)  # modulo operation
  return [index1, index2]

```

Calling `box_product([1,2,3], [4,5])` produces the stream `[[1,4], [2,4], [3,4], [1,5], [2,5], [3,5]]`. Each possible pair of elements between `[1,2,3]` and `[4,5]` are produced.

### Infinite Cartesian product

The problem with the box method is that it does not work if the first stream is infinite. The first step, pairing each element of the first list with the first element of the second list, is infinitely long, so the second element of the second list is never reached.

<figure markdown="span">
    ![Bad Pairing](images/bad_pairing_light.svg#only-light)
    ![Bad Pairing](images/bad_pairing_dark.svg#only-dark)
</figure>

## Cantor pairing function

If we have two infinite sequences, then all possible pairs form an infinite two-dimensional grid of points, which we are trying to unravel into one dimension. Going row-by-row or column-by-column will never work because there is no end to any row or column. Intuitively, we need some systematic method of starting in the corner and working our way out. George Cantor, the 1800s mathematician, proved this is possible with his [famous pairing function](https://en.wikipedia.org/wiki/Pairing_function).

Cantor's solution to the problem was fairly straightforward; don't walk row by row or column by column, but start in the corner and walk along each diagonal.

<figure markdown="span">
    ![Cantor pairing](images/cantor_pairing_light.svg#only-light)
    ![Cantor pairing](images/cantor_pairing_dark.svg#only-dark)
</figure>

```python
def cantor_product(stream1: Iterable, stream2: Iterable):
  start_index = 0
  sequence1 = stream1.lazy_sequence()  # Turns a stream into indexable
  loop:
    index1 = start_index
    index2 = 0  # Not used, but included for reference
    for element2 in stream2:
      element1 = sequence1(index1)  # Gets the ith element of the sequence
      yield [element1, element2]
      if index1 == 0:
        break
      else:
        index1 = index1 - 1
        index2 = index2 + 1
    start_index = start_index + 1

def cantor_pairing(index1: Index0, index2: Index0):
  return (index1 + index2) * (index1 + index2 + 1) / 2 + index2

def cantor_unpairing(index: Index0):
  w = floor((sqrt(8 * index + 1) - 1) / 2)
  t = (w ^ 2 + w) / 2
  index2 = index - t
  index1 = w - index2
  return [index1, index2]
```

## Szudzik pairing function

One disadvantage of walking along the diagonal is that extreme values of the streams are used before some less extreme values. For example, if we ask for the first four elements of the Cartesian product between `[0,1,2,...]` and `[0,1,2,...]`, then we might expect that we would get the elements `[[0,0], [1,0], [0,1], [1,1]]`, not necessarily in that order. But with `cantor_product`, we get `[[0,0], [1,0], [0,1], [0,2]]`. The `[0,2]` element appears before the `[1,1]` element. It is undesirable in some situations to see the element `x` of a stream before all combinations that include element `x-1` have been exhausted.

> Requirement 1: When element `[x,y]` is produced by the Cartesian product stream, all elements `[a,b]` where `max(a,b) < max(x,y)` have already been produced.

Another disadvantage is that the generator cannot run efficiently because it must walk the first iterable backwards. I use a `lazy_sequence` method to cheat on this; it is meant to represent a sequence that iterates over a stream once and caches all the values it generates so that future index lookups are fast. Unless the iterables are such that they can be walked backwards and forwards, the generator must either store `O(sqrt(index))` elements or use `O(sqrt(index))` time to fast forward to the `index1` element each time.

Matthew Szudzik recently invented [a new pairing function](https://szudzik.com/ElegantPairing.pdf) that intentionally avoids the first disadvantage of Cantor's pairing function and (I think) unintentionally avoids the second as well. It was constructed to ensure that for each stream, no index `x` appears before all possible combinations of `x-1` have been exhausted. It does this by walking down the edges of successively larger squares as illustrated below. I call the current square the "shell" and its two edges the "legs".

<figure markdown="span">
    ![Szudzik pairing](images/szudzik_pairing_light.svg#only-light)
    ![Szudzik pairing](images/szudzik_pairing_dark.svg#only-dark)
</figure>

```python
def szudzik_product(stream1: Iterable, stream2: Iterable):
  shell = 0

  loop:
    index2 = 0
    for element2 in stream2:
      if index2 == shell:
        max_element2 = element2
        break
      yield [max_element1, element2]
      index2 = index2 + 1

    index1 = 0
    for element1 in stream1:
      if index1 > shell:
        max_element1 = element1
        break
      yield [element1, max_element2]
      index1 = index1 + 1

    shell = shell + 1

def szudzik_pairing(index1: Index0, index2: Index0):
  if index1 > index2:
    return index1 ^ 2 + index2
  else:
    return index2 ^ 2 + index2 + index1

def szudzik_unpairing(index: Index0):
  shell = floor(sqrt(index))
  if index - shell ^ 2 < shell:
    return [shell, index - shell ^ 2]
  else:
    return [index - shell ^ 2 - shell, shell]
```

## Peter pairing function

_**Edit:** I had derived this myself, but after this post was first published, it was brought to my attention that Rozsa Peter had derived a pairing function with the same sequence using a pair of recursive functions in Recursive Functions (1951), page 44. I have yet to find an online reference to this pairing function._

While the Szudzik pairing function is useful in that it completes shell `s` before continuing to the next shell, it completes each leg one at a time. This means that Cartesian product stream is biased toward containing more of the larger indexes of the first stream than the second stream until a shell completes. For example, if I ask for the first six elements of the Cartesian product of `[0,1,2,...]` and `[0,1,2,...]` then I might expect to get `[[0,0], [1,0], [0,1], [1,1], [2,0], [0,2]]`. But with `szudzik_product` I get `[[0,0], [1,0], [0,1], [1,1], [2,0], [2,1]]`, because in shell `s` I get all the `[s,y]` elements before any of the `[x,s]` elements. If I always take `s^2` elements, then it does not matter, but if I often take a slice of unrelated length, then it would be nice if the sequence was not biased toward one of the legs.

> Requirement 2: When element `[x,y]` is produced by the Cartesian product stream, the difference between the number of times `x` or `y` has appeared in the first position versus the second position is at most 1.

How would this be solved? Instead walking down one entire leg and then the other, I alternate between legs with each step until the shell is complete as illustrated below.

<figure markdown="span">
    ![Peter pairing](images/peter_pairing_light.svg#only-light)
    ![Peter pairing](images/peter_pairing_dark.svg#only-dark)
</figure>

```python
def peter_product(stream1: Iterable, stream2: Iterable):
  shell = 0
  max_element1 = stream1.iterator().next()
  max_element2 = stream2.iterator().next()

  loop:
    index1 = 0 # Not used, just for reference
    index2 = 0
    leg1 = stream1.iterator()
    leg2 = stream2.iterator()

    loop:
      yield [max_element1, leg2.next()]
      index2 = index2 + 1

      if index2 > shell:
        leg1.next()
        max_element1 = leg1.next()
        max_element2 = leg1.next()
        break

      yield [leg1.next(), max_element2]
      index1 = index1 + 1

    shell = shell + 1

def peter_pairing(index1: Index0, index2: Index0):
  shell = max(index1, index2)
  step = min(index1, index2)
  if step == index2:
    flag = 0
  else:
    flag = 1
  return shell ^ 2 + step * 2 + flag

def peter_unpairing(index: Index0):
  shell = floor(sqrt(index))
  remainder = index - shell ^ 2
  step = floor(remainder / 2)
  if mod(remainder, 2) == 0:  # remainder is even
    return [shell, step]
  else:
    return [step, shell]
```

## Alternative pairing function

While the previous method alternates between legs as it walks down them, it always starts on the same leg, the leg with element `[s,0]`. An alternative method would also alternate between starting with `[0,s]` and `[s,0]`. This won't change the maximum imbalance, which is still one more element in one leg than another, it just changes it so that the `[0,s]` leg sometimes has one more element than the other. Whether this is a better method is probably a matter of taste.

<figure markdown="span">
    ![Alternative pairing](images/alternative_pairing_light.svg#only-light)
    ![Alternative pairing](images/alternative_pairing_dark.svg#only-dark)
</figure>

```python
def alternative_product(stream1: Iterable, stream2: Iterable):
  shell = 0
  flag = true
  max_element1 = stream1.iterator().next()
  max_element2 = stream2.iterator().next()

  loop:
    index1 = 0
    index2 = 0
    leg1 = stream1.iterator()
    leg2 = stream2.iterator()

    loop:
      flag = not flag

      if flag:
          yield [max_element1, leg2.next()]
          index2 = index2 + 1
          if index2 > shell:
            leg1.next()
            max_element1 = leg1.next()
            max_element2 = leg2.next()
      else:
          yield [leg1.next(), max_element2]
          index1 = index1 + 1
          if index2 > shell:
            leg2.next()
            max_element2 = leg2.next()
            max_element1 = leg1.next()

def alternative_pairing(index1: Index0, index2: Index0):
  shell = max(index1, index2)
  step = min(index1, index2)
  if mod(shell, 2) == 0 and step == index1
      or mod(shell, 2) == 1 and step == index2:
    flag = 0
  else:
    flag = 1
  return shell ^ 2 + step * 2 + flag

def alternative_unpairing(index: Index0):
  shell = floor(sqrt(index))
  step = floor((index - shell ^ 2) / 2)
  if mod(index, 2) == 0:  # index is even
    return [step, shell]
  else:
    return [shell, step]
```

## Conclusion

I asserted that this was a practical exercise, and you may be wondering what practical purpose making such fine distinctions between methods could serve.

The first requirement was the 4th element from either component stream could not appear in the Cartesian product stream until all pairs involving the 1st, 2nd, and 3rd elements from both had been exhausted. The Szudzik method had already incorporated this requirement, because it was a more practical generator of SK-combinator expressions. For an example closer to my biological realm, consider that I have two base classes of drugs and an unbounded number of undeveloped modifications schemes. Once I develop a modification scheme, I can apply it to both classes of drugs to make a variant of each, but each development is costly. I wish to find a combination therapy that is effective against some disease by successively trying pairs of variants in some standard experiment. Because a modification scheme is costly to develop, then I will want to try all combinations involving the variants based on modifications I already have before developing a new scheme. Now in the real world, I suppose I could just find all combinations of my existing schemes by hand, but if I am simulating this whole process, then I need a function that correctly and reproducibly generates pairs for the experiment.

What about the requirement that the 4th element from one stream cannot appear 3 times before the 4th element from the other stream appears at least 2 times? Say that the failures in the pairwise experiments are correlated; that is, if the 4th variant of a drug fails in an experiment, then any experiments containing the 4th variant are also more likely to fail. That means I should first try combinations that involve variants that have been tried the least, for instance, the 4th variant of the other drug. Combined with the previous rule, this means I should alternate which drug class I take the new variant from and which drug class I take an older variant from to be its pair.

###### The [code used to generate the figures in this post](https://github.com/drhagen/pairing) is available on Github. Included is Python versions of all pairing, unpairing, and Cartesian product functions described.
