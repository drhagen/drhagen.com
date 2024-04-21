---
title: "One-Sided Debate over Sequence Syntax"
date: 2015-05-09
categories:
  - "programming"
---

Computers are [famously stupid machines](https://www.benshoemate.com/2008/11/30/einstein-never-said-that/). You have to tell them in perfect detail not just what you want them to do but how to do it. A computer may be able to add 1 and 2 faster than I can, but it will take me longer to tell it to do that than for me to do it myself. The more complex the task, the more time it takes to code. Coding is still laborious and entirely not worth it unless such code will be used many times. I posit that a computer is only useful for doing work when the vast majority of the work to be done is repetitive tasks on simple objects. The most common abstraction for representing a bunch of objects is the sequence (also known as a list or array), in which each object in the collection is associated with an integer called its index. There is a wide diversity of syntax and semantics for accessing and changing sequences.

<!-- more -->

## Indexing with `()` or `[]`

Getting things out of a sequence is essential. In fact, programming languages almost universally have special syntax for this operation. If `seq` is a sequence and `i` is an integer, then usually either `seq(i)` or `seq[i]` is an expression that returns an element of the array. Which element it returns is strangely a matter of debate discussed in the next section. The use of `[]` usually appears in languages descended from C and the use of `()` appears in languages either descended from Fortran or designed by people with a penchant for minimalism and abstraction. Those who are familiar with using `[]` may be wondering what the syntax is for function calls if `()` is already taken. It is the same for both; Matlab and Scala all use `()` for both function calls and array access. This is not ambiguous at all. Like any operation on an object, the behavior depends on the type. Just as some languages uses `+` for both integer addition and string concatenation, some languages use `()` for both function calling and sequence indexing.

By abstracting away the interfaces for function calling and sequence access, the designer gives the programmer the ability to his own objects that behave like functions and sequences. Once they are abstracted away, there is little reason to keep separate syntax for `()` and `[]`. Python is one of the few languages that allows the user to implement his own versions of `()` (`__call__`) and `[]` (`__getitem__`). I have never seen a class use both in a way that could not be better designed to use just one of them.

If I were designing a language, I would reserve `[]` for doing more useful things like applying type parameters (e.g. `sum[Integer](my_numbers)`) and leave `()` to pull double duty on functions and sequences.

## Indexing with 0 or 1

If you have a sequence `seq` equal to `[5,6,7,8]`, does `seq(1)` return 5 or 6?  If you have only used languages where it returns the first element 5, you may be surprised that not only are there languages where `seq(1)` returns the second element 6, but that it is actually more common to return the second element than the first, and that there are people out there who strongly believe in this design. Unlike the first section, which describes diversity that many people don't notice, the first index is a highly contentious debate. The underlying difficulty is that humans prefer 1 to be the first element (called 1-indexing) and [computers prefer 0](https://exple.tive.org/blarg/2013/10/22/citation-needed/) to the be first element (called 0-indexing).

The most famous [argument in favor of 0-indexing](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html) was made by Dijkstra. But I think a better argument is a practical one: when the sequence represents a multidimensional array, finding the `[i,j]` element is quite easier when everything is 0-indexed. Consider this example with a two-dimensional array embedded in a sequence:

```python
# Array with values stored in column-major order
n = 3 # three rows
m = 3 # three columns
A = [1,2,3,4,5,6,7,8,9]
# Interpreted as
# 1,2,3
# 4,5,6
# 7,8,9
```

```python
# 1-indexing
i = 2 # second row
j = 3 # third column
A((i-1)*n + j)
```

```python
# 0-indexing
i = 1 # second row
j = 2 # third column
A(i*n + j)
```

Notice how cleaner the code is inside the indexing when using 0-indexing. It becomes uglier as the number of dimensions increases. If you are thinking that is not worth the silly definitions of `i`, `j`, and `k`, then we are thinking the same thing. It would be much better to actually structure the data so that it does the math for the programmer:

```python
# Sequence of sequences
A = [[1,2,3],[4,5,6],[7,8,9]]

i = 2 # second row
j = 3 # third column
A(i)(j)

# 2D array
B = [1,2,3;4,5,6;7,8,9]
B(i,j)
```

When using a sequence of sequences or a full-fledged multidimensional array, there is no need to store the number of rows and columns in variables because that information is contained in the object itself. Once the linear indexing no longer has to be calculated by hand, the advantages of 0-indexing falls away leaving the far more natural 1-indexing to prevail.

## Mutating or updating

Every language not only has a way to access an element of the sequence but also a way to change an element of the sequence. The syntax for changing a sequence is deceptively similar across languages. Most languages have something like looks like `seq(1) = 2` which conceptually replaces the first element with the object `2`. Depending on the language, it will actually mean one of these three things:

1. **Mutate.** Replace the first element of the sequence object with`2`. All previous references to the object now see a new sequence.
2. **Alter.** Make a copy of the sequence with the first element changed to `2` and then repoint the name `seq` to this copy. The local name `seq` is now a new sequence but other references to the sequence remain unchanged.
3. **Update.** Be an expression whose value is a copy of the sequence with the fist element changed to `2`. No references are changed, but the user can assign this value to a name or perform other operations on it.

I just made up all those names to help with the discussion. The procedural programming languages, like C and Python, use the Mutate interpretation. Replacing an element in these languages has far-reaching consequences. The scientific programming languages, like R and Matlab, use the Alter interpretation. It looks like sequences are mutable in these languages when looking at Matlab or R, but in fact, all sequences are immutable (or assignment always makes a copy). For those not familiar with one of the syntaxes, here is an example of each:

```python
A = [1,2,3]
B = A

A(1) = 5

print([A(1), B(1)])
# Mutate prints [5,5]
# Alter prints [5,1]
# Update prints [1,1] (value was discarded)
```

In how one uses them, Mutate and Alter are very similar. In what they actually do internally, Mutate and Update are almost exactly the same. But Mutate and Update have something in common that Alter does not have—Mutate and Update can be replaced by method calls. Mutate is a void method that simply has the side effect of changing the object (e.g. `void mutate(Element, Integer)`), while Update returns a value of the same type (e.g. `def update(Element,Integer): [Element*]`). Alter requires help from the language to actually complete its task because it repoints a name (like an assignment) after calling the update function. Below is how the language may desugar the `A(1) = 5` line from above.

```python
A.mutate(1,5) # Mutate
A = A.update(1,5) # Alter
A.update(1,5) # Update
```

The only language that I have found that actually uses the Update syntax is Scala when using an immutable type; mutable types interpret that syntax as mutate, which can be a little confusing. The Update syntax is a little awkward to actually use, which is probably why it is so rare. Notice how it does nothing in the example. To actually do something, one has to assign it to a name like `C = A(1) = 5`, whose two equal signs are a little weird.

The conflict here is between mutability and immutability. Unlike the other debates, this is one where both sides have merit. There are times when immutable objects are useful and times when mutable objects are useful. One of the nice things about Scala is that it allows these two paradigms to be used side by side. Scala is the only language I have found that provides syntax for two of the three possibilities, even though the syntax is a little clunky for update.

What syntax should be used? Here are some possible syntaxes:

```python
A(i,j) = a
A(i,j) <- a
A(i,j) := a

A.(i,j) = a # etc.

A{(i,j) = a} # etc
```

Alter is special in that it is the only one that requires reassignment. Because of this, I think whatever syntax is used for normal assignment (an `=` in most programming languages), that same symbol should be used for Alter.  Let's assume that I get fancy so that `A ← [1,2,3]` is the syntax for assigning`[1,2,3]` to the name `A`. This would make `A(1) ← 5` the natural choice for Alter.

Mutate can be translated to a method call, but we should still choose some nice syntax for it. I guess I am partial to `A(1) := 5` for mutation. There is a case to be made for using `=`, the most common syntax, for mutation, the most common interpretation. My hesitance is that `=` for assignment was a bad choice for assignment in the first place. The `=` sign is much more natural at acting as an equality operator. But let's not get into that here.

Update can also be translated to a method call, but it too deserves some nice syntax. I am partial toward something like `A{(i,j) ← a}` because it is clear that there is no reassignment of the name `A` going on here. It also naturally allows for more than one update to be done at once, like `A{(i,j) ← a; (k,l) ← b}`. Considering that the sequence will have to be copied for each update, this may result in some easy computational savings. This may seem silly for sequences, but if we extend this to structures, like `A{field1 ← a; field2 ← b}`, the utility becomes clearer.
