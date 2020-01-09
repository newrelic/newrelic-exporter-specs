
[Micrometer](https://micrometer.io/) calls their metrics "`Meter` primitives", and they support the following:

* [Gauge](#Gauge)
* [TimeGauge](#TimeGauge)
* [Counter](#Counter)
* [DistributionSummary](#DistributionSummary)
* [FunctionCounter](#FunctionCounter)
* [FunctionTimer](#FunctionTimer)
* [Timer](#Timer)
* [LongTaskTimer](#LongTaskTimer)

None of these track time, so the registry is responsible for synthesizing the times
at the moment of export.

All micrometer `Meter` types may contain "tags", which is a collection of key/value pairs.
All of these Tags are converted to New Relic metric Attributes at export time. 

Here is how the New Relic registry converts each of these types:

# Gauge

We export the micrometer `Gauge` as a New Relic `Gauge`.

Attributes:
* baseUnit: (the gauge's base unit from the id)
* description: (the gauge's description)

# TimeGauge

We export the micrometer `TimeGauge` as a New Relic `Gauge`.

Attributes:
* baseUnit: (the gauge's base unit from the id)
* description: (the gauge's description)

# Counter

We export the micrometer `Counter` as a New Relic `Count`.

* value - delta between current count and previous count (should always be positive)

Attributes:
* baseUnit: (the counter's base unit from the id)
* description: (the counter's description)

# DistributionSummary

Distribution Summary provides a summary of an event's distribution.

We take a snapshot and marshall it as:

* Snapshot `Summary`
  * name: "&lt;name>.summary"
  * count: `snapshot.count()`
  * total: `snapshot.total()`
  * min: `null`
  * max: `snapshot.max()`

We also generate one `Gauge` per snapshot percentile:
* name: "&lt;name>.percentiles"
* percentile: (the actual percentage number)

The framework also generates a `Gauge` for each histogram bucket.

# FunctionCounter

The `FunctionCounter` is converted to a New Relic `Count` object.

Attributes:
* baseUnit: (the function counter's base unit from the id)
* description: (the function counter's description)


# Timers

There are 3 kinds of timers:  `Timer`, `FunctionTimer`, and `LongTaskTimer`.
Aside from the basic `Meter` interface, these three do not share a common interface.

## Timer

The micrometer `Timer` is intended to track a large number of short running events.

We export it as a `Summary`:

**Summary:**
* name: "&lt;name>.summary", 
* count: `timer.count()` (the total number of times stop() has been called on this timer)
* total: the total time spent in all occurrences
* min: `null`
* max: `timer.max()` the maximum time seen for a single event

The summary has these attributes:
  * baseTimeUnit: (the timer's base time unit)
  * description: (the timer's description)

The framework also generates a `Gauge` for each histogram bucket.

## FunctionTimer

The micrometer `FunctionTimer` is intended to track two monotonically increasing values: count
(the number of total invocations) and time (total time spent for all invocations).

We export it a `Summary`:

**Summary:**
* name = "&lt;name>.summary", 
* count: `functionTimer.count()`
* total: `functionTimer.totalTime()`
* min: `null`
* max: `null`

The summary has these attributes:
  * baseTimeUnit: (the timer's base time unit)
  * description: (the timer's description)

## LongTaskTimer

The micrometer `LongTaskTimer` is intended to track the count and total duration of all 
in-flight long-running tasks.  It is used to time tasks that are still ongoing (because 
the plain `Timer` only reports on completion).

We export it as a two gauges:

**Gauge 1:**
* name = "&lt;name>.active_tasks", 
* value = number of currently active tasks

**Gauge 2:**
* name = "&lt;name>.total_duration",
* value - The cumulative duration of all active tasks in nanoseconds

Both metrics have these attributes:
* baseUnit: (the timer's base unit from the id)
* baseTimeUnit: hard coded to "NANOSECONDS"
* description: (the timer's description)
