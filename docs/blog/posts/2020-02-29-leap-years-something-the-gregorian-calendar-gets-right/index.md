---
title: "Leap Years: Something the Gregorian Calendar Gets Right"
date: 2020-02-29
categories:
  - "calendars"
---

Calendars coordinate people with people. It is better to be on vacation the same week your family is also. It is better for kids to be in school the same days as their teachers. It is better to be at work when everyone else is. (Though it is worse to be driving to said work when everyone else is.) In the modern world, it can be easy to think that coordinating people with people is all calendars do. If that were all, we could certainly do with a much simpler calendar—4 weeks to a month, 12 months to a year, no leap days, no irregularities forever. I won't argue that it couldn't be simpler, but I will argue that it cannot be perfectly simple. Because calendars also coordinate people with nature.

<!-- more -->

Seconds, minutes, and hours divide time into somewhat arbitrary periods; that is, there is no physical process with which they are meant to be synchronized. However, days and years in the Gregorian calendar and months in lunar calendars are intended to be synchronized with tangible astronomical processes. It is nice if the sun reaches its zenith at roughly the same time each day. It is nice if the sun reaches its equinox on roughly the same day each year. It is nice if the moon is full on roughly the same day each month.

Synchronizing with astronomical events is an indispensible when those events have a large impact on human behavior. The day is by far the most important. If you tell me you would like to have a meeting at 8:00 AM, I know that that is in the morning, early, but not too early. The circadian rhythm is so critical to human functioning that it is difficult to even imagine a calendar not synchronized with terrestrial days.

The year is the second most important astronomical cycle. The life of a pre-industrial person revolved around the seasons. A calendar synchronized with the tropical year would tell him when to plant and harvest crops, when to hunt and fish, when his domesticated animals were available for breeding, etc. The modern world is not ruled so tightly by the seasons (thanks to global trade we can buy any produce any day of the year), but it is still very strongly affected by it. It will likely be fun to have an barbecue in August (New England latitude). It will likely be dangerous to drive in January. I will need to start mowing the grass again in April.

The month is the third most important astronomical cycle. Before the invention streetlighting, the moon had a meaningful effect on people's lives. When there is a full moon, people can still walk around at night without artificial light, meaning that people can travel, socialize, or whatever. If there is no moon, it is too dark to do anything. Many cultures have holidays close to full moons because of this. The Gregorian calendar is not synchronized with the moon, but one holiday, Easter, moves around because its definition includes being the next Sunday after a full moon. However, widespread artificial lighting has eliminated any need to consider the phase of the moon before heading out. In fact, I would be surprised if 10% of the people reading this could even say what phase the moon was in right now. The absence of moon synchronization is one thing the Gregorian calendar gets right.

### Leap Xs

Synchronizing a calendar with more than one astronomical cycle poses an important difficulty to making a calendar: no astronomical cycle divides cleanly into another astronomical cycle.

- Mean solar days per tropical year: [365.24217](https://en.wikipedia.org/wiki/Tropical_year)
- Mean solar days per synodic month: [29.530587981](https://en.wikipedia.org/wiki/Lunar_month#Synodic_month)
- Synodic months per tropical year: 12.368266

It is not possible to count out any integer number of days or integer number of months to start a new year without ending up desynchronized with the seasons. _The desire to have perfectly regular periods is fundamentally incompatible with the desire to have perfectly synchronized periods._ There must be a different number of days per calendar year if that calendar is to remain synchronized with the seasons. The only question is which years get an extra day/days.

## Observation

One solution is to synchronize by observing the actual astronomical phenomenon of interest. Perhaps day one of the month is when the new moon is observed. Perhaps day one of the year is the first new moon after the winter solstice is observed. This ensures that the length of the longer unit differs by at most one smaller unit.

One problem with this is that sometimes it can be uncertain if the desired astronomical event was observed. When looking for a new moon, it may be cloudy, or just a really close call. When a close call happens, people in two towns may think the new year/month started on two different days, ruining the main purpose of a calendar, which is to coordinate separated people. Assigning this job to a central authority (like a priest in the capital) can help, but then requires distant towns to wait on messengers to deliver the verdict.

A more serious problem with the observation approach is that the number of days in a month or months in a year is unpredictable. It is possible to predict in advance some months and some years, but some will be too close to call until the judgement is rendered.

## Math

An alternative solution is to rely on a mathematical formula for determining leap days and leap months. Ideally, the mathematical formula would be simple, but the ratios above suggest that no simple formula exists. There is a fundamental trade-off between the complexity of the formula and how closely it tracks the true astronomical cycle.

For example, the Julian calendar adds one leap day every four years. This is a very simple formula, but it drifts relative to the tropical year by adding about 3 too many leap days every 4 centuries. After about a millennium, the seasons come about a week too early. (To appreciate the importance of simple formulas, read about how [hilariously long](https://en.wikipedia.org/wiki/Julian_calendar#Julian_reform) it took the Romans to correctly implement even this formula after deciding to use it.)

The Gregorian calendar is a slight modification of the Julian calendar. The leap day is skipped if the year is divisible by 100, unless the year is also divisible by 400. By dropping three leap days every 400 years, the calendar has essentially no long term drift, with maybe 1 too many leap days every few thousand years.

Despite the absence of long-term drift, the Gregorian calendar still drifts more than necessary in the short term. Any calendar with leap days must drift about one day, but the drift in the Gregorian calendar is more than two days. The winter solstice is listed as December 21, but it actually happens sometime between the morning of December 20 and the afternoon of December 22. When a leap day is skipped every 100 years, it means that 8 years go by all at once with no leap day. The seasons would drift less if, instead of letting 8 years pass every 100 years or so, it let 5 years pass every 33 or 34 years. This would reduce the amount of drift that was allowed to accumulate, but make it much harder to compute if a given year (say, 2020) was a leap year or not. The fact that the Gregorian calendar has an extra day of drift within the 400 year cycle does not appear to be something that anyone cares about.

## Conclusions

When designing an optimal calendar, the first choice to be made is which astronomical cycles to synchronized to. The more cycles that are captured, the more useful information about the environment can be extracted from a given date. I think the solar calendar, including the Gregorian calendar, gets this right. The time of day and the season of the year still matter a lot even in the modern world full of artificial light, air conditioning, and desk jobs. The phase of the moon, however, no longer matters at all. Each cycle that much be synchronized adds considerable complexity and trade-offs. Lunisolar calendars differ in length by a month rather than differing in length by a year. If I were designing an optimal calendar, I would drop the moon and keep the sun and earth.

Once deciding to use a solar calendar, one needs to decide the formula for the leap years. Here is probably the most remarkable thing about the Gregorian calendar. Some renaissance dudes managed to get everyone to switch to a new calendar whose only purpose was to fix a one-day per century drift in the Julian calendar. Furthermore, even with modern astronomy, we probably can't come up with a better formula. If I were designing an optimal calendar, I would keep the every-4-years-except-every-100-except-every-400 formula.

If you are looking for something to remark upon this Leap Day, remember that the weather is just as bad (New England latitude) as it has been every Leap Day since a pope with an abacus ensured that Leap Days would stay in the dead of winter forever.
