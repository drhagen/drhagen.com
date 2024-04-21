---
title: "The sort-within-groups problem"
date: 2022-03-06
categories:
  - "programming"
---

There is an interesting edge case in data grammars when a grouped data table is sorted by non-group columns. For example, what should the following dplyr code produce?

```r
library(dplyr)

df <- data.frame(
  group = c(2, 2, 1, 1, 2, 2),
  value = c(3, 4, 3, 1, 1, 3)
)

df %>% group_by(group) %>% arrange(value)
```

<!-- more -->

In this intro example, there is one group column and one other column by which the table is being explicitly sorted. There are two groups, `group=1` and `group=2`, with `group=2` being non-contiguous—meaning the rows of that group are spread throughout the table rather than being all next to each other. There are actually four defensible answers to how the rows should be sorted in the face of grouping:

1. Ignore the grouping
2. Sort by group columns first
3. Cluster by group columns first
4. Sort within the rows owned by each group

## Ignore groups

In the beginning, there was only dplyr and, through version 0.3, dplyr simply ignored the grouping when sorting. At that time, the example problem from the intro would have produced:

```
 group value
     1     1
     2     1  # Note that value is
     2     3  # completely sorted
     1     3
     2     3
     2     4
```

Now, this is admittedly a little unexpected. No other verb in the data grammar completely ignored groups (except `group_by` itself, I suppose).

## Sort by groups first

dplyr was already quite popular when 0.3 came out—popular enough that there were Stack Overflow questions ([1](https://stackoverflow.com/questions/24649730/r-dplyr-combination-of-group-by-and-arrange-does-not-produce-expected-res), [2](https://stackoverflow.com/questions/26555297/arrange-a-grouped-df-by-group-variable-not-working), [3](https://stackoverflow.com/questions/29567659/dplyr-arrange-sort-groups-by-another-column-and-then-sort-within-each-group)) and GitHub issues ([1](https://github.com/tidyverse/dplyr/issues/369), [2](https://github.com/tidyverse/dplyr/issues/491)) from users confused that `arrange` was ignoring grouping. It was in that last GitHub issue that ignoring groups was [considered a bug by the lead maintainer Hadley Wickham](https://github.com/tidyverse/dplyr/issues/491#issuecomment-50345872). Starting in [dplyr 0.4](https://dplyr.tidyverse.org/news/index.html#data-framestbl_df-0-3), the `arrange` verb was [explicitly defined](https://github.com/tidyverse/dplyr/issues/491#issuecomment-50892729) as sorting by group columns first and then the supplied columns. This sorting by group columns first ensured that all rows belonging to a group were contiguous. The rows would be sorted by the supplied columns, as expected, within the block, but not globally across the whole table. The example problem from the intro would result in:

```
 group value
     1     1  # Note how all the group=1 row
     1     2  # have been moved to the top
     2     1
     2     3
     2     3
     2     4
```

This change caused a handful of Stack Overflow questions ([1](https://stackoverflow.com/questions/32446893/dplyr-arrange-not-behaving-as-expected-after-group-by-and-summarize), [2](https://stackoverflow.com/questions/33642923/arrange-grouped-local-data-frame-in-descending-order)) and a multitude of GitHub issues ([1](https://github.com/tidyverse/dplyr/issues/676), [2](https://github.com/tidyverse/dplyr/issues/986), [3](https://github.com/tidyverse/dplyr/issues/1206), [4](https://github.com/tidyverse/dplyr/issues/1553)) to be be raised by users confused about the change. These complaints must have made an impression on Hadley because [he decided to revert it](https://github.com/tidyverse/dplyr/issues/1206#issuecomment-118471800) back to the ignore-groups behavior and, in version 0.5, that happened. This triggered a new avalanche of Stack Overflow questions ([1](https://stackoverflow.com/questions/38749417/dplyr-arrange-not-arranging-by-groups), [2](https://stackoverflow.com/questions/39409193/dplyr-0-5-arrange-using-groupings), [3](https://stackoverflow.com/questions/43832434/arrange-within-a-group-with-dplyr), [4](https://stackoverflow.com/questions/46046032/trying-to-understand-dplyr-function-group-by), [5](https://stackoverflow.com/questions/54729267/dplyr-group-by-and-arrange-functions-together-doesnt-group-same-values-together)) and GitHub issues ([1](https://github.com/tidyverse/dplyr/issues/1721), [2](https://github.com/tidyverse/dplyr/issues/2116)) from the users who were still confused that `arrange` ignored groups. Hadley [said](https://github.com/tidyverse/dplyr/issues/1206#issuecomment-239848173) that reverting it yet again was out of the question. Eventually, a `.by_group=TRUE` option was [added in 0.7](https://dplyr.tidyverse.org/news/index.html#dplyr-070) to cause `arrange` to also sort by group columns like in 0.4.

I would argue that those users didn't actually want `arrange` to ignore groups; they didn't know they _had_ groups when they used `arrange`. If you look at the six complaints about the sort-by-groups-first behavior, you notice that that are all doing the exact same sequence of operations: group by two columns, summarize a third column, and finally sort by that third column. They seemed to not realize that the table was still grouped after the `summarize`. You see, `summarize` in dplyr drops the rightmost group column, leaving the table still grouped by the remaining columns. If you group by one column, summarizing returns an ungrouped table. But if you group by two columns, summarizing returns a table that is still grouped by the first column. This is the confusing behavior of dplyr, not the sorting by groups first.

Partial ungrouping after `summarize` probably warrants its own blog post. I understand why dplyr does it this way, but it can be surprising even to an experienced user. I think the reason this comes up with `arrange` instead of the other verbs is that, if you are expecting an ungrouped table, most of verbs usually still do what you want. If you do `filter,` `mutate`, `transmute`, even `group_by` then `summarize`, you usually get pretty much what you expect even if there are unexpected groups left over. But `arrange` is not like them. If you `arrange` a table with unexpected groups, you get really weird garbage out if the sorting prioritizes the group columns above the columns you requested.

## Cluster by groups

Back when I used dplyr 0.4 (my first exposure to dplyr), I was little bit annoyed that it sorted groups because my groups were usually already contiguous and in the order that I wanted them. I thought that better behavior would be to "cluster" by group columns first and then sort within groups. Under clustering-before-sorting, the example problem from the intro would look like:

```
 group value
     2     1  # Note how group=2
     2     3  # is now contiguous
     2     3  # but still first
     2     4
     1     1
     1     3
```

### Cluster

Cluster is a verb in its own right; it takes a list of columns and and behaves a lot like sort in that the rows of all groups are contiguous at the end. However, the groups retain the same order relative to each other. Running `cluster` on a table who groups are already contiguous is a no-op. For tables who groups are not contiguous, the groups are ordered according to the order that the first row from each group appears.

The cluster operation can be defined (rather inefficiently) as:

1. Add a temporary column saving the current row indexes
2. Group by the cluster columns
3. Set the temporary equal to the minimum value in each group
4. Ungroup
5. Sort by the temporary column
6. Delete the temporary column

In dplyr, clustering alone can be done like this:

```r
df %>%
  mutate(temp = row_number()) %>%
  group_by(group) %>%
  mutate(temp = min(temp)) %>%
  ungroup() %>%
  arrange(temp) %>%
  select(-temp)
```

```
 group value
     2     3  # Groups have been
     2     4  # clustered, but not
     2     1  # sorted
     2     3
     1     3
     1     1
```

## Rows owned by groups

When grouping, each row belongs to a particular group. For example, rows 1, 2, 5, and 6 belong to group `group=2` in our example table. We could define sorting on a grouped table as sorting each group in isolation, but the same rows belong to that group at the end. For example, using the problem from the intro, we pull out rows 1, 2, 5, and 6, sort them, and then put those four rows back into rows 1, 2, 5, and 6 following their new order:

```
 group value
     2     1
     2     3
     1     1  # Relative order of
     1     3  # rows is unchanged
     2     3
     2     4
```

Arguably, this is the most technically correct solution. It most strongly reinforces the idea that grouping creates subtables isolated from each other and applies subsequent verbs to each subtable individually. This is basically how `mutate` works conceptually; it pulls each group out, mutates the given columns, and then puts the rows back where they came from.

## Conclusion

The seeming correctness of the owned-rows solution is tempting. This is identical to the cluster-first solution if the table is already clustered by groups, which is probably the most typical situation when users care about the row order of a grouped table.

The Polars library actually does it this way. If you use the equivalent (albeit, not as readable) syntax, you get the same result as in the "Rows owned by groups" section:

```python
import polars as pl

df = pl.DataFrame({'group': [2,2,1,1,2,2], 'value': [3,4,3,1,1,3]})

df.with_column(pl.col('value').sort().over('group'))

# shape: (6, 2)
# ┌───────┬───────┐
# │ group ┆ value │
# │ --- ┆ --- │
# │ i64   ┆ i64   │
# ╞═══════╪═══════╡
# │ 2     ┆ 1     │
# ├╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┤
# │ 2     ┆ 3     │
# ├╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┤
# │ 1     ┆ 1     │
# ├╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┤
# │ 1     ┆ 3     │
# ├╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┤
# │ 2     ┆ 3     │
# ├╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┤
# │ 2     ┆ 4     │
# └───────┴───────┘
```

The Pandas `groupby` operation itself [sorts by default and clusters if sorting is turned off](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.groupby.html), rendering moot the question of how sort should be behave on grouped tables. Here is the operation using Pandas's characteristic syntax:

```python
import pandas as pd

df = pd.DataFrame({'group':[2,2,1,1,2,2], 'value':[3,4,3,1,1,3]})

df.groupby('group', sort=False).apply(lambda x: x.sort_values('value'))

#          group  value
# group
# 2     4      2      1
#       0      2      3
#       5      2      3
#       1      2      4
# 1     3      1      1
#       2      1      3
```

Always clustering groups is interesting in its own right. Clustering allows the group columns to be indexed efficiently and guarantees that a number of edge cases that come from non-contiguous groups simply can never happen. But ultimately, the dplyr behavior is more versatile even if it leaves a lot of weird edge cases. As a user, you often want to preserve row order while still using various verbs, including grouping.

Nevertheless, I think that Pandas is actually on to something here. Once the group verb has executed, it is reasonable to expect that all subsequent verbs will operate within each group. However, it is perfectly reasonable for the group verb itself to have three modes of operation:

1. Sort rows by groups
2. Cluster rows by groups
3. Preserve row order

"Groups are completely isolated from each other" is nice in principle, but being pedantic about this can lead to some weird places. If groups are completely isolated from each other then `df %>% group_by(group) %>% arrange(group)` is a no-op because the sorting is done within each group (where the group column is always constant). Even `cluster`, if implemented as its own verb, could not cluster along group columns because the relationship between groups was already frozen. This is definitely unexpected, and simply doing nothing, is putting principle over practicality. That does not mean library writers should sacrifice principles. Principles keep away edge cases. I am currently inclined toward having the three modes of operation and raising a well-crafted error when a group column is sorted that could guide users to sort before grouping, sort after ungrouping, or sort during grouping.
