---
layout: post
title: "Dropwizard Metrics - misleading behaviour"
featured: false
author: marek
categories: Dropwizard,Metrics
type: post
image: '/assets/2020-06-01-dropwizard-metrics-misleading-behaviour/oneMinuteRate_100.png'
comments: false
---

In this article, you will find information on:
* The misleading behaviour of the Dropwizard library.

## Expectations about rates
Dropwizard has a nice class `Meter`, which exposes methods: `oneMinuteRate`, `getFiveMinuteRate` and `getFifteenMinuteRate`.
In the beginning, I was expecting that oneMinuteRate would just show value from events for the last 60 seconds.
Additionally, I would expect that metric will react immediately for changes. Let's check the reality.

## OneMinuteRate behaviour

I wrote a very simple application:
* for 30 seconds, every second I call `Meter.mark(100)` and print `oneMinuteRate` value,
* for next 1 hour I just print  `oneMinuteRate` value.

```
import com.codahale.metrics.Meter;

public static void main(string args) throws InterruptedException {
    Meter meter = new Meter();
    // write and print for 30 seconds
    for(int i=0; i<30; i++) {
        meter.mark(100);
        Thread.sleep(1_000);
        System.out.println(meter.getOneMinuteRate());
    }

    // print for 1 hour
    for(int i=0; i<3_600; i++) {
        Thread.sleep(1_000);
        System.out.println(meter.getOneMinuteRate());
    }
}
```
I took the values and drew a diagram.

<figure>
  <img src="/assets/2020-06-01-dropwizard-metrics-misleading-behaviour/oneMinuteRate_100.png" alt="OneMinuteRate for 100 seconds">
</figure>


What I saw surprised me:
* metrics didn't react for 4 seconds after start,
* metrics didn't react for 4 seconds after stop,
* after 1 minute after stop, value of the metric was not zero, but was just decreasing,
* it stopped at value 100 (writes per second were occurring at a rate of 100 writes per second).

Let's see how the diagram looks **after 1 hour**:

<figure>
  <img src="/assets/2020-06-01-dropwizard-metrics-misleading-behaviour/oneMinuteRate_3600.png" alt="OneMinuteRate for 1 hour">
</figure>

From the picture, we can see, that *oneMinuteRate* doesn't go to zero immediately, but exponentially.
After 1 hour, oneMinuteRate achieves value **4,495742528561689E-25**. After 1 hour it is still **not zero**.

## Possible consequences
Imagine that you think that oneMinuteRate behaves in an intuitive way and you set a monitoring alarm to react on value zero or something close to zero.
The alarm would happen with a big delay or will not happen at all. In real time business applications it can cause a big revenue loss.

## Explanation
Of course Dropwizard metrics is a well documented project and we can dive deeper into documentation.

Internally, histograms use reservoirs to store a period of metric data upon which standard statistical information and percentiles are calculated.
By default, a histogram uses an **exponentially decaying reservoir** (EDR).
In [documentation](https://metrics.dropwizard.io/3.2.3/manual/core.html#exponentially-decaying-reservoirs) we can read:

> A histogram with an exponentially decaying reservoir produces quantiles which are representative of (roughly) the last five minutes of data. It does so by using a forward-decaying priority reservoir with an exponential weighting towards newer data. Unlike the uniform reservoir, an exponentially decaying reservoir represents recent data, allowing you to know very quickly if the distribution of the data has changed.

It doesn't sound very bad, but there are several potential implications:
* EDRs are lossy by design; they don’t store every sample (they’re statistically representative).
* EDRs, by default, store a static 1028 samples and samples are weighted towards the past 5 minutes.
* The rate at which samples decay within EDRs is influenced by how frequently the histogram is updated.

## Possible solutions
* In January 2019, Dropwizard 3.2.3 introduced SlidingTimeWindowArrayReservoir / SlidingTimeWindowMovingAverages  - a sliding window-based, lossless reservoir implementation with no loss of precision.
* Calculation of rates on the server side sounds interesting. It simplifies monitoring and looks quite intuitive.
But the second solution could be moving the calculation of rates to the client side. Then you can also move to a different metric library like [Micrometer](https://micrometer.io)

## Dropwizard solution with SlidingTimeWindowMovingAverages
```
import com.codahale.metrics.Meter;

public static void main(string args) throws InterruptedException {
    SlidingTimeWindowMovingAverages averages = new SlidingTimeWindowMovingAverages();
    Meter meter = new Meter(averages);

    // write and print for 30 seconds
    for(int i=0; i<30; i++) {
      meter.mark(100);
      Thread.sleep(1_000);
      System.out.println(meter.getOneMinuteRate());
    }

    // print for 1 hour
    for(int i=0; i<3_600; i++) {
      Thread.sleep(1_000);
      System.out.println(meter.getOneMinuteRate());
    }
  }
```
The result of execution, you can see below:

<figure>
  <img src="/assets/2020-06-01-dropwizard-metrics-misleading-behaviour/oneMinuteRateSlidingWindow.png" alt="OneMinuteRate with SlidingTimeWindow">
</figure>

What we can see is:
* it reacts immediately after start,
* it goes to level `3000` - sum of values of events: 30 * 100,
* 60 seconds after calling it, it goes back to zero.

# Summary
For me, it was pretty surprising behaviour. I know it was well-documented, but for many developers it could also be surprising.
I am pretty sure you want to have a monitoring system that react on spikes / drops in important KPIs in real time.
In general, I think the Dropwizard library was a great library some years ago. Also, the assumption of calculating metrics on the server side can look attractive.
Nowadays, I think it is time to change this assumption.