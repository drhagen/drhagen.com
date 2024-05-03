---
title: "The Missing 23rd of the Month"
date:  2016-01-02
categories:
  - "culture"
---

Previously, I explained why the [11th of most months is mentioned far less than the other days](../2015-12-29-the-missing-11th-of-the-month/index.md) in the Google Ngrams database of English literature from 1800-2008. This was to solve a long-standing question posed in an [xkcd comic](https://xkcd.com/1140/). While researching this, I encountered another mystery: the 2nd, 3rd, 22nd, and 23rd are unusually low as well—but only until the 1930s, at which point they become perfectly normal days. Last time, I set this question aside to focus on the 11th. In this installment, I explain the strange behavior of these four days.

<!-- more -->

To remind everyone, the graph below is the mystery we are dealing with. The `2nd`, `3rd`, `22nd`, and `23rd` are practically unused in 1800, the earliest point in the database. Around 1810 is when the first substantial uses appear; they grow at about the same rate as the other days, maintaining a substantial gap at about half of what one would expect until about the 1890s. Then suddenly, the gap shrinks and continues to do so until the 1930s when `2nd`, `3rd`, `22nd`, and `23rd` are absorbed into the main group.

<figure markdown="span">
    ![new_style](images/new_style_light.png#only-light)
    ![new_style](images/new_style_dark.png#only-dark)
</figure>

## Ye old style

So were `2` and `3` unlucky numbers in the 1800s? Did Google's algorithm have a hard time reading the `2`s and `3`s of old-timey fonts? Nope, it turns out that people used to write these ordinals as `2d`, `3d`, `22d`, and `23d`. I took the median over `January 2d`, `February 2d`, etc. for each year and did the same for the other old-style ordinals. The graph below shows the use of old-style ordinals, which start as normal days within the main group, but slowly diverge until they drop off exponentially in the 1890s, reaching a tiny residue by the 1930s.

<figure markdown="span">
    ![old_style](images/old_style_light.png#only-light)
    ![old_style](images/old_style_dark.png#only-dark)
</figure>

Sometimes you can encounter a modern use of the old-style abbreviations when the ordinal is part of a name with a very long history, like the [3d Marine Division](https://www.3rdmardiv.marines.mil/). This is _not_ why the old-style has a small residual in the latter half of the twentieth century. If you [search through Google Books for modern uses](https://www.google.com/search?q=%22january+2d%22&biw=1920&bih=1075&source=lnt&tbs=cdr%3A1%2Ccd_min%3A1%2F1%2F1970%2Ccd_max%3A2008&tbm=bks) of `January 2d`, you will only find reprints of old books and publications of old diaries.

## Combined graph

The old style falls away as the new style emerges. When we add the old-style and new-style ordinals together, we get the graph below, which shows that once the two styles are accounted for, these four days of the months are actually quite ordinary.

<figure markdown="span">
    ![After seven graphs without title text, some people will still check the eighth one.](images/sum_new_old_style_light.png#only-light)
    ![After seven graphs without title text, some people will still check the eighth one.](images/sum_new_old_style_dark.png#only-dark)
</figure>

I don't have a fully satisfying explanation for why the `2nd` and `3rd` now peek their heads above the main group from time to time. I guess if the `1st` on the month is hugely over-represented, it is reasonable to expect that the next smallest ordinals would be slightly over-represented. ("Let's have our meeting on the first of the month." "I have ten other meetings on the first!" "Ok then, the second.") However, if I [search Google Books for instances](https://books.google.com/books?id=axcXAAAAYAAJ&dq=%22January%202d%22&pg=PA248#v=onepage&q&f=false) of `January 2d` or `January 2nd`, there are a sizable number of hits from lists like this: ![january_2d](images/January_2d_light.png#only-light)![january_2d](images/January_2d_dark.png#only-dark) Google Books apparently ignores commas. With the `1st`, `2nd`, `3rd`, and `4th` being the only regular ordinals for weeks of the month, these might get a boost this way.

## Speculation

Why did writers use these one-letter abbreviations? Probably to follow Latin, where [this practice originated](https://en.wikipedia.org/wiki/Ordinal_indicator) and the ordinal indicator is a single letter `o`. The Romance languages, like Spanish, Italian, and Portuguese, still use `o` and `a`. I expect that we would still be using `d` if it wasn't for `1st`, `4th`, etc. whose final consonant sound cannot be represented by a single letter. In the end, consistency within the English language by using two letters for all ordinals was more attractive than similarity to Latin.

###### The [code used for this analysis](https://github.com/drhagen/xkcd11th) is available on Github.
