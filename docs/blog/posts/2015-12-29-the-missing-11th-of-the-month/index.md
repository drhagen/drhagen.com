---
title: "The Missing 11th of the Month"
date: 2015-12-29
categories:
  - "culture"
---

<figure markdown="span">
    ![In months other than September, the 11th is mentioned substantially less often than any other date. It's been that way since long before 9/11 and I have no idea why.](images/calendar_of_meaningful_dates_light.png#only-light)
    ![In months other than September, the 11th is mentioned substantially less often than any other date. It's been that way since long before 9/11 and I have no idea why.](images/calendar_of_meaningful_dates_dark.png#only-dark)
    Source [xkcd](https://xkcd.com/1140/). Image licensed under [CC-BY-NC](https://creativecommons.org/licenses/by-nc/2.5/).
</figure>

On November 28th, 2012, Randall Munroe published [an xkcd comic](https://xkcd.com/1140/) that was a calendar in which the size of each date was proportional to how often each date is referenced by its ordinal name (e.g. "October 14th") in the [Google Ngrams database](https://books.google.com/ngrams) since 2000. Most of the large days are pretty much what you would expect: [July 4th](https://en.wikipedia.org/wiki/Independence_Day_(United_States)), [December 25th](https://en.wikipedia.org/wiki/Christmas), the 1st of every month, the last day of most months, and of course a [September 11th](https://en.wikipedia.org/wiki/September_11_attacks) that shoves its neighbors into the margins. There are not many days that seem to be smaller than the typical size. [February 29th](https://en.wikipedia.org/wiki/Leap_year) is a tiny speck, for instance. But if you stare at the comic long enough, you may get the impression that the 11th of most months is unusually small. The title text of the comic concurs, reading "In months other than September, the 11th is mentioned substantially less often than any other date. It's been that way since long before 9/11 and I have no idea why." After digging into the raw data, I believe I have figured out why.

<!-- more -->

First I confirmed that the `11th` is actually interesting. There are 31 days and one of them _has_ to be smallest. Maybe the `11th` isn't an outlier; it's just on the smaller end and our eyes are picking up on a pattern that doesn't exist. To confirm this is real, I compared actual numbers, not text size. The Ngrams database returns the total number times a phrase is mentioned in a given year normalized by the total number of books published that year. The database only goes up to the year 2008, so it is presumably unchanged from when Randall queried it in 2012.

I retrieved the count for each day for the year (`January 1st`, `January 2nd` etc.) and took the median over the months for each day (median of `January 1st`, `February 1st`, etc.) for each year. This summarizes how often the `11th` and the other 30 days of the month appear in a given year. Using the median prevents outlier days like `July 4th` from dragging up the average for its corresponding ordinal (the `4th`). Only if a ordinal is unusual for at least 6 of the 12 months will its median appear unusual.

I took the median for each ordinal over the years 2000-2008. The graph below is a histogram of the 31 medians. The `1st` of the month stands out far above them all and the `15th` just barely distinguishes itself from the remainder. Being the first day and the middle day of the month, these two make sense. However, the `11th` stands out as the lowest by a significant margin (p-value < 0.05), with no immediate explanation.

<figure markdown="span">
    ![histogram_2000-2008](images/histogram_2000-2008_light.png#only-light)
    ![histogram_2000-2008](images/histogram_2000-2008_dark.png#only-dark)
</figure>

This deficit has been around for a long time. Below is all the ordinals for every year in the data set, 1800-2008. The data is smoothed over eleven years to flatten out the noise. Even at the beginning, the `11th` is significantly lower than the main group. This mild deficit continues for a few decades and then something weird happens in 1860s; the 11th suddenly diverges from its place just below the pack. The gap between the `11th` and the ordinary ordinals expands rapidly until the `11th` is about half of what one would expect it to be throughout the first half of the twentieth century. The gap shrinks in the second half of the twentieth century, but still persists at a smaller level until the end.

<figure markdown="span">
    ![ordinals_1800-2008](images/ordinals_1800-2008_light.png#only-light)
    ![ordinals_1800-2008](images/ordinals_1800-2008_dark.png#only-dark)
</figure>

Astute graph readers will notice that something else weird is going on. There are four other lines that are much lower than they should be. From highest to lowest, they are the `2nd`, the `3rd`, the `22nd`, and the `23rd`. They were even lower than the `11th` from 1800 until the 1890s. However, starting around 1900, their gaps started shrinking even as the `11th` diverged until the gap disappeared completely in the 1930s. There is an interesting story there, but because their effect doesn't persist to the present, I'll continue to focus on the `11th` and leave the others for a [future post](../2016-01-02-the-missing-23rd-of-the-month/index.md).

## Typographical hijinks

When I began this study, I was hoping to find a hidden taboo of holding events on the 11th or typographical bias against the shorthand ordinal. Alas, the reason is far is far more mundane: a numeral `1` looks a lot like a capital `I` or a lowercase `l` or a lowercase `i` in most of the fonts used for printing books. An `11` also looks like an `n`, apparently. Google's algorithms made mistakes when reading the `11th` from a page, interpreting the ordinal as some other word.

We can find some of these mistakes by directly searching for nonsense phrases like `March llth` or `July IIth` or `May iith`. There are nine possible combinations of `I`, `l`, and `i` that a `11` could be mistaken for.  Five of them can actually be found in the database for at least one month: `IIth`, `Ilth`, `iith`, `lith`, and `llth`. Also found was `1lth`, `1ith`, and `l1th`, in which only one letter was misread. I collectively refer to these errors as `xxth`. [Google books](https://books.google.com/) queries a newer database than the one on which Ngrams was built, but bona fide examples of the misreads can still be found. [Here is something](https://books.google.com/books?id=OJo3AAAAMAAJ&dq=%22January%20IIth%22&pg=RA3-PA34#v=onepage&q=%22January%20IIth%22&f=false) that Google books thinks says `January IIth`: ![January IIth](images/January_IIth_light.png#only-light)![January IIth](images/January_IIth_dark.png#only-dark). [And here is one](https://books.google.com/books?id=EcJQAQAAIAAJ&dq=%22February%20llth%22&pg=RA1-PA79#v=onepage&q=%22February%20llth%22&f=false) for `February llth`: ![February llth](images/February_llth_light.png#only-light)![February llth](images/February_llth_dark.png#only-dark). [And finally one](https://books.google.com/books?id=zYHk_df06QsC&dq=%22March%20lith%22&pg=PA402#v=onepage&q=%22March%20lith%22&f=false) for `March lith`: ![March lith](images/March_lith_light.png#only-light)![March lith](images/March_lith_dark.png#only-dark). There are hordes of these in the database. You can find other ordinals that were misread as well, but the `11th` with its slender and ambiguous `1`s was misread far more often than the others.

I added back in every instance of `January IIth`, `January llth`, etc. to `January 11th` and did the same to the other months. The graph below shows that the `11th` gets a big boost by adding back the nonsense phrases. Before the 1860s, the difference between the `11th` and the main group is erased. After the 1860s, about a quarter to a third of the difference is erased.

<figure markdown="span">
    ![xxth_1800-2008](images/xxth_1800-2008_light.png#only-light)
    ![xxth_1800-2008](images/xxth_1800-2008_dark.png#only-dark)
</figure>

## To the nth degree

So where did the rest of the missing `11th` go? Well, starting in the 1860s, the Google algorithm starts to make a rather peculiar error—it misreads `11th` as `nth`. [Here is one example from a page full](https://books.google.com/books?id=r7QaAAAAYAAJ&dq=%22january%20nth%22&pg=RA1-PA82#v=onepage&q=%22january%20nth%22&f=false) of `January nth`s: ![January nth](images/January_nth_light.png#only-light)![January nth](images/January_nth_dark.png#only-dark). In some years, the number of incorrect reads actually exceeds the number of correct reads. I added `January nth` to `January 11th` and did the same for all the months. The graph below shows both the `nth` and its sum with the `11th`. There was little impact before the 1860s, but then this error alone accounts for nearly all of the missing `11th`.

<figure markdown="span">
    ![nth_1800-2008](images/nth_1800-2008_light.png#only-light)
    ![nth_1800-2008](images/nth_1800-2008_dark.png#only-dark)
</figure>

## Combined graph

When the `xxth` misreads and `nth` misreads are both added back into the `11th`, the gap disappears across the entire timeline and the `11th` looks like an ordinary day of the year. This suggests that the misreading of the `11th` as `nth`, `IIth`, `llth`, etc. is sufficient to explain the unusually low incidence of the `11th` as a day of the month.

<figure markdown="span">
    ![total_1800-2008](images/total_1800-2008_light.png#only-light)
    ![total_1800-2008](images/total_1800-2008_dark.png#only-dark)
</figure>

## Typographical machines

While it makes sense that the `11th` was misread more than others, why is the misread rate not uniform? What happened in the 1860s that caused the dramatic rise in the error rate? I suspect that it has something to do with a special device invented in the 1860s_—_the typewriter. [The earliest typewriters did not have a separate key for the numeral `1`.](https://web.archive.org/web/20190706134959/https://chronicle.com/blogs/linguafranca/2012/03/14/old-style-versus-lining-figures/) Typists were expected to use the lowercase `l` to represent a `1`. When the algorithm read `October llth`, it was far more correct than we have been giving it credit. There are not that many documents in Google books that are typewritten, but this popular new contraption had a powerful effect on the evolution of fonts. The `1` and `l` were identical on the increasingly familiar typewriters, and the fonts even of printed materials began to reflect this expectation. Compare the `l`s and `1`s in this font [from 1850](https://books.google.com/books?id=i-lOAAAAYAAJ&dq=%22january%2011th%22%201850&pg=PA115#v=onepage&q&f=false): ![council_meeting](images/council_meeting_light.png#only-light)![council_meeting](images/council_meeting_dark.png#only-dark). There is a clear difference between an `l` with no serifs on the top and the `1` with a pronounced serif. Compare that to a font [from 1920](https://books.google.com/books?id=STtaT_fheaMC&lpg=PA334&dq=%22january%2011th%22%201920&pg=PA334#v=onepage&q&f=false): ![hotel_on_sunday](images/hotel_on_sunday_light.png#only-light)![hotel_on_sunday](images/hotel_on_sunday_dark.png#only-dark). The characters are identical except for the kerning. Even to this day, most fonts represent both the `1` and the `l` as tall characters with two serifs on the bottom and one left-facing serif at the top. The only difference is that the serif on the `1` is slightly more angled than on the `l`. (In this post, I used a special monospace font to make it easier to tell the difference.) The print quality of more recent books (post 1970s) has reduced the rate of failure, but it still has not gone away entirely, so that the remaining failures were noticeable in the xkcd comic.

The largest open question is why `nth` was chosen so often. It seems like such a strange error to make. The word `nth` is a legal word in mathematical and scientific publications, so that should help its chances of getting picked. In most fonts the top of the `n` is really thin, and is likely invisible in many texts on which they trained the algorithm. But there is a big different in height between `1` and `n`, especially in the typewriter era, which is where the errors occur. And the phrase `January nth` is nonsense so that should have hurt its chances of being selected. Is it possible there was an error in one of the modern training texts that had an `11th` labeled as `nth`, thereby confusing the algorithm? The only way to know for sure would be to crack open the source code of Google's text-reading algorithm. This is left as an exercise for the reader.

###### The [code used for the analysis in this post](https://github.com/drhagen/xkcd11th) is available on Github.
