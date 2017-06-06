---
title: "History of Tech: New York to San Francisco Travel"
---

As a measure of progress in general purpose technologies, it can be fun to look at a specific problem or limitation over time to see how new tech has affected it. For example, Planet Money had an [episode](http://www.npr.org/sections/money/2016/11/30/503769949/episode-534-the-history-of-light) tracking the cost of producing a lumen of light, which decreased exponentially over time. Or, you might look at the amount of time it takes to travel from New York to San Francisco, by the fastest means available to normal consumers.

Here are a few data points on duration, in hours, of trips from New York to San Francisco at different points in time:

| Year | Time (Hours) | Log Scale | Note                                                     |
|------|--------------|-----------|----------------------------------------------------------|
| 1810 | 4320         | 3.635     | Astoria Overseas Expedition                              |
| 1849 | 4320         | 3.635     | Duxbury                                    |
| 1853 | 2136         | 3.330     | Flying Cloud                                             |
| 1876 | 83           | 1.919     | Transcontinental Railroad                                |
| 1934 | 26           | 1.415     | Boeing 247 (Prop Plane. 10 Passenger Capacity)           |
| 1937 | 17           | 1.230     | DC-3 (Prop Plane. 32 Passenger Capacity)                 |
| 1959 | 6            | 0.778     | American Airlines introduces first commercial jet route. |
| 2017 | 6            | 0.778     | Today's in-air times are similar to those of 1959.       |

Here's what that looks like on a linear, and on a log (base 10) scale:

![linear](http://i.imgur.com/1zP6kbX.png){: .center-image}

![log](http://i.imgur.com/AITyYU7.png){: .center-image}

To me, there are two really interesting things here. The first is how much of a difference the transcontinental railroad made; a trip that used to take 3 to 6 *months* could now be made in 3.5 *days*, a reduction of more than 96%. To scale today, that would be like reducing the travel time from New York to San Francisco to under 15 minutes.

The second interesting thing is that transit times haven't gotten any faster since 1959. That isn't to say that planes haven't gotten faster; the [SR-71 Blackbird](https://www.wikiwand.com/en/Lockheed_SR-71_Blackbird) could make the flight in under 1.5 hours. But there are no widely-available commercial options that fast. As others have pointed out, airfare is much cheaper today than it was in 1959; so while speed hasn't improved, costs have.

Just for fun, we can fit a "Moore's Law" line to our data points:

![moore](http://i.imgur.com/IMpmj4w.png){: .center-image}

The slope of this line is -0.0169; very, very roughly, this means that for the period under consideration (1810 to 2017), travel times have gotten 10x faster every 59 years. If you believe this trend will continue (and there's no real evidence that it will), then 59 years from now, we "should" be able to travel from New York to San Francisco in 36 minutes instead of 6 hours.


