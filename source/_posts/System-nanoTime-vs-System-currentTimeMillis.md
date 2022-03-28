---
title: System.nanoTime vs System.currentTimeMillis
date: 2022-03-28 18:13:34
categories: BackEnd 
tags:
    - Java
top:
---
# 1. Monotonic Clock

- suitable for measuring a duration, such as a timeout or a service’s response time
    - `clock_gettime(CLOCK_MONOTONIC)`
    - `System.nanoTime()`
- They are guaranteed to always move forward
- However, the *absolute* value of the clock is meaningless: it might be the number of nanoseconds since the computer was started, or something similarly arbitrary. In particular, it makes no sense to compare monotonic clock values from two different computers, because they don’t mean the same thing.
    - On a server with multiple CPU sockets, there may be a separate timer per CPU, which is not necessarily synchronized with other CPUs

# 2. Time of day clock

- It returns the current date and time according to some calendar
    - `clock_gettime(CLOCK_REALTIME)`
    - `System.currentTimeMillis()`
        - return the number of seconds since the epoch
- this time is synchronized with NTP, which means a timestamp from one machine ideally means the same as a timestamp on another machine
- Oddies
    - In particular, if the local clock is too far ahead of the NTP server, it may be forcibly reset and appear to jump back to a previous point in time. These jumps, as well as similar jumps caused by leap seconds, make time-of-day clocks unsuitable for measuring elapsed time