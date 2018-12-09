---
title: A Tip on Go's Time Formatting
date: 2018-12-09
---

It seems not so many people understand Go's design of time formatting since it's so different from other languages and the official document doesn't intentionally explain it. Actually it's simple as count from 1 to 7:

```
01/02 03:04:05PM '06 -0700

01 - month
02 - day
03 - hour
04 - minute
05 - second
06 - year
07 - time zone
```

So if you'd like literal month, "01" becomes "Jan"(short version) or "January"(long version); if you want 24-hour format, "03PM" becomes "15"; if you want 4-digit year format, "06" becomes "2006"; if you want literal time zone, "-0700" becomes "MST".

And there're "Mon" and "Monday" for day-of-week, ".999999999" for nanosecond, and ".000", ".000000", ".000000000" for millisecond, microsecond, nanosecond, respectively.