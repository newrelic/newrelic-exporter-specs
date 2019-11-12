# Kamon 

New Relic [provides two reporters](https://github.com/newrelic/kamon-newrelic-reporter) 
for [Kamon telemetry](https://kamon.io/):
* `NewRelicMetricsReporter`
* `NewRelicSpanReporter`

These reporters leverage the Kamon reporter/module API and integrate with
data from the Kamon [kanela agent](https://github.com/kamon-io/kanela).

---
# Spans

This section describes Kamon spans that are sent to the 
[New Relic span ingest API](https://docs.newrelic.com/docs/understand-dependencies/distributed-tracing/trace-api/introduction-trace-api).

## Common attributes

All spans will have the following common attributes on them:
* `instrumentation.source`: "kamon-agent"
* `service.name`: value comes from the config at `kamon.environment.service`

## Span field data

Other span field data maps over pretty directly.  The field on the left is the new relic
span field name, and the right is where it is sourced:

* `spanId`: `kamonSpan.id.string`
* `traceId`: `kamonSpan.trace.id.string`
* `parentId` `kamonSpan.parentId`
* `name`: `kamonSpan.operationName`
* `timestamp`: Microseconds->millisecond conversion of `kamonSpan.from`

## Span specific attributes:

Each span has the following attributes:

* `span.kind`: `kamonSpan.kind.toString`
* `remoteEndpoint`: (if client span) `Endpoint{ipv4=<ip>, ipv6=<ip6>, port=<port>}`

In addition, all `tags` and `metricTags` on the kamon span are sent as NR span attributes.

---
# Metrics

Kamon has 5 kinds of Metric data, and the New Relic reporter supports 4 of them:

* Gauge
* Counter
* Histogram
* Timer

---
## Gauge

A Kamon gauge can have multiple "instruments". Each instrument within a given Kamon gauge
is converted into a New Relic Gauge.

* `name`: `gauge.name`
* `value`: `instrument.value`
* `timestamp`: `snapshot.to.toEpochMilli` - Kamon snapshot's "to" instant 

### Attributes:
* all tags on the instrument, plus
* `sourceMetricType`: `gauge`
* `description`: `gauge.description`
* `dimensionName`: `gauge.settings.unit.dimension.name`
* `magnitudeName`: `gauge.settings.unit.magnitude.name`
* `scaleFactor`: `gauge.settings.unit.magnitude.scaleFactor`

---
## Counter

A Kamon counter can have multiple "instruments". Each instrument within a given Kamon counter
is converted into a New Relic Count metric.

* `name`: `counter.name`
* `value`: `counter.value`
* `startTimeMs`: `snapshot.from.toEpochMilli` - Kamon snapshot's "from" instant
* `endTimeMs`: `snapshot.to.toEpochMilli` - Kamon snapshot's "to" instant

### Attributes:

* all tags on the instrument, plus
* `sourceMetricType`: `counter`
* `description`: `counter.description`
* `dimensionName`: `counter.settings.unit.dimension.name`
* `magnitudeName`: `counter.settings.unit.magnitude.name`
* `scaleFactor`: `counter.settings.unit.magnitude.scaleFactor`

---
## Histogram

The structure underlying the Kamon histogram is a "Distribution".
Each histogram has one or more instruments, and each instrument has 
, in addition to count, min, max, and sum, zero or more buckets and 
zero or more percentiles.

We are not currently reporting bucket data, only percentiles.  
Buckets could be added at a later date.  For each instrument, we generate
a New Relic Summary metric and Guage for each percentile:

#### Summary:
* `name`: `distribution.name` + ".summary"
* `count`: `instrument.value.count` (cast down from long to int)
* `sum`: `instrument.value.sum` (cast up from long to double)
* `min`: `instrument.value.min` (cast up from long to double)
* `max`: `instrument.value.max` (cast up from long to double)
* `startTimeMs`: `snapshot.from.toEpochMilli` - Kamon snapshot's "from" instant
* `endTimeMs`: `snapshot.to.toEpochMilli` - Kamon snapshot's "to" instant

*Attributes*:
* all tags on the instrument, plus:
* `sourceMetricType`: "histogram"
* `lowestDiscernibleValue`: `histogram.settings.dynamicRange.lowestDiscernibleValue`
* `highestTrackableValue`: `histogram.settings.dynamicRange.highestTrackableValue`
* `significantValueDigits`: `histogram.settings.dynamicRange.significantValueDigits`
* `magnitude.name`: `histogram.settings.unit.magnitude.name`
* `magnitude.scaleFactor`: `histogram.settings.unit.magnitude.scaleFactor`
* `dimension`: `histogram.settings.unit.dimension.name`

#### Gauges: 

Each percentile in the histogram is turned into a Gauge:

* `name`: `histogram.name` + ".percentiles"
* `value`: `percentile.value` (cast up from long to double)
* `timestamp`: `snapshot.to.toEpochMilli` - Kamon snapshot's "to" instant 

*Attributes*:
* all tags on the instrument, plus:
* `percentile.countAtRank`: `percentile.countAtRank`
* `percentile`: `percentile.rank`
* `sourceMetricType`: "histogram"
* `lowestDiscernibleValue`: `histogram.settings.dynamicRange.lowestDiscernibleValue`
* `highestTrackableValue`: `histogram.settings.dynamicRange.highestTrackableValue`
* `significantValueDigits`: `histogram.settings.dynamicRange.significantValueDigits`
* `magnitude.name`: `histogram.settings.unit.magnitude.name`
* `magnitude.scaleFactor`: `histogram.settings.unit.magnitude.scaleFactor`
* `dimension`: `histogram.settings.unit.dimension.name`

---
## Timer

Just like Histograms, the Kamon Timer is also backed by the Distribution 
type.  Therefore, it is converted just like Histogram (above) with the 
exception of the `sourceMetricType`: `timer` attribute.
