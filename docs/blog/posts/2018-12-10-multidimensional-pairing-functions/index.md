---
title: "Multidimensional Pairing Functions"
date: 2018-12-10
categories:
  - "programming"
---

In the [previous post](../2018-03-27-superior-pairing-function/index.md), I compared ways to take two infinite streams and generate a new stream that is all possible combinations of the elements in those streams. This post takes it up another level and generalizes this procedure to an arbitrarily long list of infinite streams. This is a trickier task than the 2-dimensional case, utilizing recursion into each dimension to cleanly generate all combinations.

Throughout this post, the caret `^` will indicate exponentiation and parentheses `list(index)` will be used to indicate indexing a list. As before, 0-indexing will be used because it makes the math simpler. 

<!-- more -->

## Multidimensional box pairing function

Many programming languages have a function that will generate the Cartesian product of `n` lists of objects. In Python, it is called `itertools.product`. The standard implementation starts with the first element from each list. Then it takes the second element from the last list and the first element from the remaining lists. It continues walking through the last list until it is exhausted. Then it increments the second to last list by one element and walks through the last list again.  This is repeated until the first list (which is walked slowest) is finally walked through once; at that point, all possible combinations have been generated. An algorithm to do this is straightforward to implement. It can with be done imperatively by updating a mutable list of indexes (like the [Python version](https://github.com/python/cpython/blob/9718b59ee5f2416cdb8116ea5837b062faf0d9f8/Modules/itertoolsmodule.c#L2257-L2277)) or it can be done functionally using a recursive function (like the version below).

<figure markdown="span">
    ![Multidimensional box pairing](images/multidimensional_box_pairing_light.svg#only-light)
    ![Multidimensional box pairing](images/multidimensional_box_pairing_dark.svg#only-dark)
</figure>

```
def multidimensional_box_product(*streams: List[Iterable]) -> Iterable[List]:
  def recursive_product(i_stream) -> Iterable[List]:
    for element in streams(i_stream):
      if i_stream == streams.indexes.last:
        # On final dimension, yield each element
        yield [element]
      else:
        # On other dimensions, yield each element followed by each
        # possible combination of elements in the remaining dimensions
        for remaining_elements in recursive_product(i_stream + 1):
          yield [element] ++ remaining_elements

  for elements in recursive_product(0):
    yield elements


def multidimensional_box_pairing(lengths: List[Integer],
                                 indexes: List[Integer]) -> Integer:
  # Should assert that all indexes are less than corresponding lengths
  index = 0
  dim_product = 1
  # Compute indexes from last to first because that is the order the
  # product is grown
  for dim in lengths.indexes.reverse:
    index += indexes(dimension) * dim_product
    dim_product *= lengths(dimension)

  return index


def multidimensional_box_unpairing(lengths: List[Integer],
                                   index: Integer) -> List[Integer]:
  # Should assert that index is less than the product of lengths
  indexes = List.fill(length=lengths.length, element=0)  # Preallocate list
  dim_product = 1
  # Compute indexes from last to first because that is the order the
  # product is grown
  for dim in lengths.indexes.reverse:
    indexes(dim) = mod(floor(index / dim_product), lengths(dim))
    dim_product *= lengths(dim)

  return indexes
```

## Multidimensional Szudzik pairing function

The two-dimensional Szudzik pairing function first divides the x-y plane into shells, each shell defined by `max(x,y)`. This is a straightforward property to generalize. The n-dimensional space is divided into n-dimensional shells defined by `max(x1, x2, ..., xn)`. Next, the two-dimensional Szudzik pairing function walks down each leg of the shell until the `x==y` element is reached. There are several ways to generalize this property.

A three-dimensional shell will have three faces. One sensible way to fill the shell would be to fill each face one at a time, treating each face as a plane to be filled with the two-dimensional Szudzik pairing function. Such a method could be recursively scaled up to arbitrary dimensions.

<figure markdown="span">
    ![Multidimensional recursive Szudzik pairing](images/multidimensional_recursive_szudzik_pairing_light.svg#only-light)
    ![Multidimensional recursive Szudzik pairing](images/multidimensional_recursive_szudzik_pairing_dark.svg#only-dark)
</figure>

However, the [original presentation](https://szudzik.com/ElegantPairing.pdf) briefly suggests in the penultimate slide a different generalization: pure lexicographical ordering within a shell. No implementation of the pairing function, unpairing function, or Cartesian product function is shown. But because this generalization is perfectly reasonable and the implementations straightforward to construct, it is the one I'll show here.

<figure markdown="span">
    ![Multidimensional sorted Szudzik pairing](images/multidimensional_sorted_szudzik_pairing_light.svg#only-light)
    ![Multidimensional sorted Szudzik pairing](images/multidimensional_sorted_szudzik_pairing_dark.svg#only-dark)
</figure>

The top level of the implementation is an infinite loop iterating over the shells, starting from the innermost shell. In each shell, all possible combinations of the streams are generated by taking each element in the first stream (until the shell is reached) and appending all possible combinations of the remaining streams, which is a recursive process. By itself, this recursive process would generate the Cartesian product of all elements within the cube bounded by the shell, like a multidimensional finite Cartesian product function. To restrict this process to just the elements on the shell, the process keeps track of whether or not an element on the shell has been emitted as it recurses. If it has not been emitted yet when the recursion reaches the final stream, only the final (shell) element of that stream is emitted, not the entire stream up to the shell, so that it guarantees that only points with at least one index on the shell are emitted within that shell's iteration. 

```
def multidimensional_szudzik_product(*streams: List[Iterable])->Iterable[List]:
  n = streams.length

  # In the recursive function, it must possible to get the shell
  # element of the final stream. This iterator is stepped through at
  # each shell to produce final_shell_element.
  final_shell_iterator = streams.last.iterator()

  def recursive_product(i_stream: Integer,
                        shell_used: Boolean) -> Iterable[List]:
    if i_stream == n - 1:
      # At the final dimension
      if shell_used:
        # A shell element was emitted earlier in the item so emit all elements
        for [i_element, element] in enumerate(streams(i_stream).take(shell+1)):
          yield [element]
      else:
        # No shell element has been emitted for this item yet and the
        # recursive function is at the last stream, so a shell element
        # only must be emitted or else this item will not be on the shell.
        yield [final_shell_element]
    else:
      for [i_element, element] in enumerate(streams(i_stream).take(shell+1)):
        next_dim = i_stream + 1
        next_shell_used = shell_used or i_element == shell

        for remaining_elements in recursive_product(next_dim, next_shell_used):
          yield [element] ++ remaining_elements

  shell = 0
  loop:
    final_shell_element = final_shell_iterator.next()
    for elements in recursive_product(0, False):
      yield elements
    shell = shell + 1


def multidimensional_szudzik_pairing(*indexes: List[Integer]) -> Integer:
  n = indexes.length
  shell = max(indexes)

  def recursive_index(dim: Integer) -> Integer:
    # Number of dimensions of a slice perpendicular to the current axis
    slice_dims = n - dim - 1

    # Number of elements in a slice (only those on the current shell)
    subshell_count = (shell + 1) ^ slice_dims - shell ^ slice_dims

    index_i = indexes(dim)
    if index_i == shell:
      # Once an index on the shell is encountered, the remaining
      # indexes follow multidimensional box pairing
      lengths = List.fill(element=shell + 1, length=slice_dims)
      return subshell_count * shell + multidimensional_box_pairing(
          lengths, indexes.slice(from=dim+1))
    else:
      # Compute the contribution from the next index the same way
      # by recursing to the next dimension
      return subshell_count * index_i + recursive_index(dim + 1)

  # Start with the number of elements from before this shell and recursively
  # find the contribution from each index to the linear index
  return shell ^ n + recursive_index(0)


def multidimensional_szudzik_unpairing(n: Integer,
                                       index: Integer) -> List[Integer]:
  shell = floor(index ^ (1 / n))

  def recursive_indexes(dim: Integer, remaining: Integer) -> List[Integer]:
    if dim == n - 1:
      # If this is reached, that means that no index so far has been on
      # the shell. By construction, this index must be on the shell or
      # else the point itself will not be on the shell.
      return [shell]
    else:
      # Number of dimensions of a slice perpendicular to the current axis
      slice_dims = n - dim - 1

      # Number of elements in a slice (only those on the current shell)
      subshell_count = (shell + 1) ^ slice_dims - shell ^ slice_dims

      index_i = min(floor(remaining / subshell_count), shell)
      if index_i == shell:
        # Once an index on the shell is encountered, the remaining
        # indexes follow multidimensional box unpairing
        lengths = List.fill(element=shell + 1, length=slice_dims)
        still_remaining = remaining - subshell_count * shell
        return [shell] + multidimensional_box_unpairing(lengths,
                                                        still_remaining)
      else:
        # Compute the next index the same way by recursing to the next
        # dimension
        still_remaining = remaining - subshell_count * index_i
        return [index_i] + recursive_indexes(dim + 1, still_remaining)

  # Subtract out the elements from before this shell and recursively
  # find the index at each dimension from what remains
  return recursive_indexes(0, index - shell ^ n)
```

## Conclusion

I had spent considerable time making a multidimensional version of the Peter pairing function. I had succeeded in writing the product function, got most of the way done with the unpairing function, and was started on the pairing function. However, before I finished, I became convinced that the Szudzik pairing function was better suited for my real-life problem. A multidimensional pairing function that meets only requirement 1 from the previous post (i.e. do it by shells) is much easier to implement than one that also meets requirement 2 (i.e. alternate between legs).

###### The [code used to generate the figures](https://github.com/drhagen/pairing) in this post is available on Github in the same repository as the previous post. Python versions of the pairing, unpairing, and Cartesian product functions are included.
