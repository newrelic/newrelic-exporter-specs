DropWizard metrics has 5 high-level metric types. 
Here is how we are mapping them to New Relic metric types.

### Reporter
The DropWizard library provides a base class called `ScheduledReporter` for Reporter implementations to extend.
The ScheduledReporter has a single abstract method to implement, `report` which takes a group of `Map<String, Metric>` maps
containing the data which can be reported. It also holds onto 2 separate fields which contain scaling factors
for rates and durations. This allows users of the library to describe how they would like their values to be reported. 
For example, you can specify that you want your rates provided in per-second, or per-minute, etc. 
And, for durations, you can specify that you want them reported in milliseconds, or seconds (or any other standard Java `TimeUnit`).

### General Information
We use the provided scaling factors as follows:

* rates : When reporting data from the `Metered` interface implementations (`Timer` and `Meter`), we scale the rates by the supplied factor. 
* durations: When reporting `Timer` data, we scale the durations measured by the underlying Histogram to the specified units.

Note: We don't currently report the time units with the metrics, but this could be added in the future.

Whenever we generate a Count or Summary metric, we populate the `interval.ms` field with the time interval between the current time and the end of the last time the `report` method completed.

### Metrics:

#### Gauge
A DropWizard Gauge is a parameterized type with a value. The parameterized type has no bounds, so it can include Strings, or any custom java type.

We model a DropWizard gauge as a New Relic Gauge metric, but only generate metrics for numerically-typed DropWizard gauges.

The `"name"` of the Gauge matches the name of the DropWizard Gauge.

#### Counter
A DropWizard Counter is a Long-valued (64-bit signed integer) counter than can be incremented and decremented. 
Because it can be decremented, we cannot model this as a New Relic Count metric, but instead must report it as a Gauge.

We model a DropWizard Counter as a New Relic Gauge that passes through the current value as-is (i.e. we don't create a delta-count of this value).

The `"name"` of the Gauge matches the name of the Counter, and set `"source.type" : "counter"`

#### Histogram
A DropWizard Histogram is not a classic evenly bucketed histogram, but is actually a block of percentiles. 
Since New Relic does not currently have a native percentile type, we instead have to model this as a set of other metrics.

A DropWizard Histogram is converted to the following, all with the same `"name"`, which is set to the name of the Histogram,
and `"source.type" : "histogram"`:
* A Count metric, representing the total number of samples that have been added to the Histogram since its creation. 
The value of the Count is the change in the number of samples since the last reporting period happened.
* A set of Gauge metrics, one per standard percentile: 50, 75, 90, 95, 99, 99.9
  * On each of these, we add an attribute `"percentile"` with the decimal value of what is being measured (eg. 99.9)
  * On the 50% gauge, we also add an attribute `"commonName": "median"`
  * On all these Gauges, we also add an attribute `"groupingAs": "percentiles"`, to inform future visualizations.
* A Summary metric which summarizes the state of the histogram at the report time.

*Note*: The Histogram is configurable with a variety of sampling strategies. 
The chosen sampling strategy isn't easily accessible to the reporter, so we are currently not utilizing this in how we report the data.

#### Meter
A DropWizard Meter is a metric that allows the user to measure the rate that things are added to it.
It provides several pieces of information that we can report to New Relic. 
First, it provides the count of the total number of events that have been seen.
Second, it provides a set of average rates. One is the mean rate over the lifetime of the meter. 
The others are the 1, 5 and 15 minute moving averages of the rates.

We model this as a Count, and 4 Gauge metrics, all with the same `"name"`, which is set to the name of the Meter,
and `"source.type" : "meter"`:
* A Count metric, representing the total number of events that have been seen by the meter, over its lifetime
The value of the Count is the change in the number of samples since the last reporting period happened.
* 4 Gauge metrics, all of which have an attribute `"groupingAs": "rates"`
  * The mean rate, with attribute `"rate": "mean_rate"`
  * The 1-minute moving average rate, with attribute `"rate": "m1_rate"`
  * The 5-minute moving average rate, with attribute `"rate": "m5_rate"`
  * The 15-minute moving average rate, with attribute `"rate": "m15_rate"`
   
#### Timer
A DropWizard Timer is used to time things. Under the hood, it is implemented as a Histogram of timings + a Meter of the rate of timing events.

Because this metric is a composite of other types, we model it as such, using the same kind of New Relic Metric types as above. 

We model a Timer with a Count, 4 Gauges for the Meter, and 6 Gauges for the underlying Histogram.
All of these have a `"name"` set to the name of the Timer, and `"source.type" : "timer"`:
* A Count metric, representing the total number of events that have been timed by the Timer, over its lifetime
The value of the Count is the change in the number of events since the last reporting period happened.
* 4 Gauge metrics, all of which have an attribute `"groupingAs": "rates"`
  * The mean rate, with attribute `"rate": "mean_rate"`
  * The 1-minute moving average rate, with attribute `"rate": "m1_rate"`
  * The 5-minute moving average rate, with attribute `"rate": "m5_rate"`
  * The 15-minute moving average rate, with attribute `"rate": "m15_rate"`
* A Set of Gauge metrics, one per standard percentile: 50, 75, 90, 95, 99, 99.9, all of which have an attribute `"groupingAs": "percentiles"`
  * On each of these, we add an attribute `"percentile"` with the decimal value of what is being measured (eg. 99.9)
  * On the 50% gauge, we also add an attribute `"commonName": "median"`
* A Summary metric which summarizes the state of the underlying histogram at the report time.
