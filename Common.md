# Common Exporter Behavior

This document describes behavior that is common to all New Relic exporters.

## User-Agent values

Each exporter MUST report an identifying string and version as part of the `User-Agent` header when sending data to New Relic.

These values should be appended using the [add_version_info API](https://github.com/newrelic/newrelic-telemetry-sdk-specs/blob/be844b5ca5044c0f80b037812de7498b3663ae34/communication.md#user-agent) described in the telemetry SDK specification.

The `User-Agent` values that are appended are always in the format `<author>-<exporter_name>-<optional_context>`.  For New Relic exporters, the author is `NewRelic`.

Language names are title cased.

| Exporter | exporter_name | Example Value Appended to the User-Agent |
| -------- | ------------- | ------------------ |
| [opencensus](opencensus) | `OpenCensus` | `NewRelic-OpenCensus-Exporter/0.1.0` |
| [micrometer](micrometer) | `Micrometer` | `NewRelic-Micrometer-Exporter/0.1.0` |
| [kamon](kamon) | `Kamon` | `NewRelic-Kamon-Exporter/0.1.0` |
| [dropwizard](dropwizard) | `DropWizard` | `NewRelic-DropWizard-Exporter/0.1.0` |

A full `User-Agent` example might look like `NewRelic-Python-TelemetrySDK/0.2.3 NewRelic-Python-OpenCensus/0.1.0`.

## Attributes

The origins of telemetry exported to New Relic needs to be identifiable in order for it to be useful.
Being able to identify what generated, measured, collected, and transmitted telemetry to New Relic is essential to providing a curated customer experience.
This section describes the attributes are used to identify these dimensions of the telemetry origins.

The following table is an overview for quick reference all the attributes defined in the following sub-sections.

| Attribute | REQUIRED/RECOMMENDED/OPTIONAL |
| --------- | ----------------------------- |
| `instrumentation.provider` | REQUIRED |
| `instrumentation.name` | RECOMMENDED |
| `instrumentation.version` | RECOMMENDED |
| `collector.name` | RECOMMENDED |
| `collector.version` | RECOMMENDED |
| `host.hostname` | REQUIRED† |
| `service.name` | REQUIRED† |
| `app.name` | REQUIRED† |
| `db.name` | REQUIRED† |

†: REQUIRED only if annotating an applicable telemetry source.
See the related sub-section for more information.

### Instrumentation

Instrumentation is the toolset used to measure telemetry.
This instrumentation usually comes in the form of a library of code, platform, or coding language builtin and provides different instruments to measure the state of a computer program.
This instrumentation is from what the exporter exports.

Including attributes to identify not only what instrumentation is used but also what code is being instrumented is crucial for interoperability between exporters and the New Relic platform.

#### `instrumentation.provider`

All exported telemetry MUST contain an `instrumentation.provider` attribute that identifies the provider of instrumentation.

Example values of this attribute are:

* `newRelic`
* `prometheus`
* `dropwizard`
* `opencensus`
* `opentelemetry`

#### `instrumentation.name`

All exported telemetry SHOULD contain an `instrumentation.name` attribute that identifies the code that is being instrumented.
This would be the framework, library, or application that uses the instrumentation to report telemetry.

Example values of this attribute are:

* `spring-framework`
* `python-twisted`
* `gorilla/mux`

#### `instrumentation.version`

All exported telemetry SHOULD contain an `instrumentation.version` attribute that identifies the version of the instrumentation.
This will be defined by the instrumentation provider.

Example values of this attribute are:

* `v0.1.0`
* `3`
* `alpha-v1`

### Collector

Telemetry sent to New Relic is often produced by many collocated sources.
Given the close proximity these telemetry sources can have, it can be advantageous to unify the transmission of this telemetry to New Relic.
This reduces network overhead and can simplify the design of system architectures.

Being able to backtrack identity what collector was the ultimate transmitter of telemetry to New Relic is often useful.
It can help debug issues and provide context of data flows.
The following attributes are used to make this identification possible.

#### `collector.name`

All exported telemetry SHOULD contain an `collector.name` attribute that identifies the name of the code that collected the telemetry.
This value SHOULD be overwritten if telemetry is proxied.
Otherwise, this will be the exporter name.

Example values of this attribute are:

* `newrelic-dropwizard-reporter`
* `newrelic-opentelemetry-go-exporter`

#### `collector.version`

All exported telemetry SHOULD contain an `collector.version` attribute that identifies the version of the collector.
This value SHOULD be overwritten if telemetry is proxied.
Otherwise, this will be the exporter version.

Example values of this attribute are:

* `v0.1.0`
* `3`
* `1.0`

### Source Kind

Clearly identifying what kind of system produced the telemetry sent to New Relic is crucial to do.
A core feature New Relic offers customers is a curated observability experience and it is not possible to provide this without this identification.

The following attributes are used to identify the kind of system for which an exporter exports telemetry.
All exported telemetry MUST include at least one of the following attributes and MAY include more than one.

#### `host.hostname`

All telemetry exported for a host system running an application MUST contain a `host.hostname` attribute that uniquely identifies that host.
Other kinds of telemetry exported MAY also contain this attribute as this can be helpful to describe the telemetry origin.

#### `service.name`

All telemetry exported for a generic computer program MUST contain a `service.name` attribute that identifies the name of the program.

#### `app.name`

All telemetry exported for a mobile or browser application MUST contain an `app.name` attribute that identifies the application.

##### `device.model`

All telemetry exported for a mobile application MUST contain an `device.model` attribute that identifies the mobile device model.

#### `db.name`

All telemetry exported for a database MUST contain an `db.name` attribute that identifies the database name.