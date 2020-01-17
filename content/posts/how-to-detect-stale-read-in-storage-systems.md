---
title: How to Detect Stale Reads in Storage Systems
date: 2020-01-17
---

In developing storage systems, whether a distributed storage or local key-value storage, stale reads generally need to be detected and eliminated. From the user's perspective, a stale read is a read that returns the old value of a key, for example, in the following read-write sequence:

```
Thread-0 write key X = 1 at T0
Thread-0 write key X = 2 at T1
Thread-0 read key X = 1 at T2
(T0 < T1 < T2)
```

`Thread-0` updates X to 2 at T1, but still reads X = 1 at T2, hence a "stale read". So how to detect it?

At my first attempt, straightforwardly, I relied on timestamps to report stale reads but found it generates a lot of false alarms. Because there's a gap between the time a value changes and the time a client thread reports the change (usually through a callback), especially in multi-threaded environments. Here is a slightly changed example from above:

```
Thread-0 write key X = 1 at T0
Thread-0 write key X = 2 at T1
Thread-1 read key X = 1 at T2
(T0 < T1 < T2)
```

Now at T2, `Thread-1` reports X = 1 instead of `Thread-0`. We cannot claim a stale read happens at T2, it's possible that the actual read happens at some time between T0 and T1, but reports at T2 for some reason (context switches, races for a lock, etc). We can only confirm that **in the same thread**, events happen one after another.

While timestamps are not reliable, causality comes to the rescue. One can't be born before his father, also a value can't be read if not written. In my final test program, I generate unique values for different write threads, run them in parallel with different read threads, and collect those read/write sequence results in the end. Below is a seemingly correct result:

```
thread 6 write key A content 006-A1
thread 6 write key B content 006-B1
thread 6 write key A content 006-A2
thread 6 write key B content 006-B2
thread 9 read key B content 006-B1
thread 9 read key B content 006-B2
thread 9 read key A content 006-A1
```
But stale reads happen. `read B 006-B2` happens after `write B 006-B2`, `write B 006-B2` happens after `write A 006-A2` since they are in the same thread, `write A 006-A2` happens after `read A 006-A1` otherwise `006-A2` would be read, so `read B 006-B2` should happen after `read A 006-A1`, boom!

This problem could be modeled as finding cycles in a directed graph, and there're [very](https://en.wikipedia.org/wiki/Tarjan%27s_strongly_connected_components_algorithm) [quick](https://www.geeksforgeeks.org/detect-cycle-in-a-graph/) algorithms. Obviously, all those reads and writes are vertices. To build the graph, 3 kinds of edges need to be considered:

- In the same thread, Event[n+1] happens after Event[n]
- The read of X[n] happens after the write of X[n]
- The read of X[n] happens before the write of X[n+1]

Where `Event[n]` denotes the n-th read or write in the thread, `X[n]` denotes the n-th value of a key.
