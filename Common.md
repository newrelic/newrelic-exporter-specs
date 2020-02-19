# Common Exporter Behavior

This document describes behavior that is common to all New Relic exporters.

<!-- TOC -->

- [User-Agent values](#user-agent-values)
- [Attributes](#attributes)
    - [Instrumentation](#instrumentation)
        - [`instrumentation.provider`](#instrumentationprovider)
        - [`instrumentation.name`](#instrumentationname)
        - [`instrumentation.version`](#instrumentationversion)
    - [Collector](#collector)
        - [`collector.name`](#collectorname)
        - [`collector.version`](#collectorversion)
    - [Source Kind](#source-kind)
        - [`host.hostname`](#hosthostname)
        - [`service.name`](#servicename)
        - [`app.name`](#appname)
            - [`mobileApp.name`](#mobileappname)
            - [`device.model`](#devicemodel)
            - [`browserApp.name`](#browserappname)
        - [`db.name`](#dbname)

<!-- /TOC -->

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

The origins of measurements exported to New Relic needs to be identifiable in order for them to be useful.
Identify what generated, measured, collected, and transmitted measurements to New Relic is essential in providing the curated experience customers expect.
This section describes the attributes that are used to serve this purpose.

The following table is an overview for quick reference of all the attributes defined in the following sub-sections.

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
| `mobileApp.name` | REQUIRED† |
| `device.model` | RECOMMENDED† |
| `browserApp.name` | REQUIRED† |
| `db.name` | REQUIRED† |

†: applies only if annotating an applicable source.
See the related sub-section for more information.

### Instrumentation

Instrumentation is the toolset used to make measurements of a system's state.
The instruments making up the instrumentation take the form of library of code, a platform, or a coding language builtin.
From the exporter's perspective, instrumentation is the thing that sends it measurements to export.

Including attributes to identify not only what instrumentation is used but also what code is being instrumented is crucial for interoperability between telemetry systems and the New Relic platform.

#### `instrumentation.provider`

All exported measurements MUST contain an `instrumentation.provider` attribute that identifies the provider of instrumentation.

Example values of this attribute are:

* `newRelic`
* `prometheus`
* `dropwizard`
* `opencensus`
* `opentelemetry`

#### `instrumentation.name`

All exported measurements SHOULD contain an `instrumentation.name` attribute that identifies the code that is being instrumented if the instrumentation provider exposes this information, otherwise it SHOULD be omitted.
This would be the framework, library, or application that uses the instrumentation to report measurements.

Example values of this attribute are:

* `spring-framework`
* `python-twisted`
* `gorilla/mux`

#### `instrumentation.version`

All exported measurements SHOULD contain an `instrumentation.version` attribute set to the version of the instrumentation if the instrumentation provider expose this information, otherwise it SHOULD be omitted.

Example values of this attribute are:

* `v0.1.0`
* `3`
* `alpha-v1`

### Collector

Identifying what collected and transmitted the measurements of a system is important to know.
It allows the New Relic platform to inform customers what telemetry systems were used to transmit their data, something that can help debug issues and provide insight into data flows.

A collector can be as simple as the exporter itself, or as complex as a hierarchical system of proxies.
Regardless of the complexity or simplicity of the collectors, the measurements they transmit need to identify what collector was the ultimate transmitter.
The following attributes are used to make this identification possible.

#### `collector.name`

If the exporter is transmitting to New Relic directly than all exported measurements SHOULD contain a `collector.name` attribute set to the name of the exporter, otherwise it SHOULD be omitted.

Example values of this attribute are:

* `newrelic-dropwizard-reporter`
* `newrelic-opentelemetry-go-exporter`

#### `collector.version`

If the exporter is transmitting to New Relic directly than all exported measurements SHOULD contain a `collector.version` attribute set to the version of the exporter, otherwise it SHOULD be omitted.

Example values of this attribute are:

* `v0.1.0`
* `3`
* `1.0`

### Source Kind

A core feature of the New Relic platform is providing a curated observability experience.
This is only possible to provide if telemetry includes information about what kind of system it is measuring.

The following attributes are used to identify the kind of the measured system.
All exported measurements MUST include at least one of the following attributes and MAY include more than one.

#### `host.hostname`

All measurements exported for a host system MUST contain a `host.hostname` attribute that uniquely identifies that host.
Measurements for other kinds of systems MAY also contain this attribute as it can be helpful in describing their origins.

#### `service.name`

All measurements exported for a generic computer program (not a mobile or browser application) MUST contain a `service.name` attribute set to the name of the program.

#### `app.name`

All measurements exported for a mobile or browser application MUST contain an `app.name` attribute set to the name of the application.

##### `mobileApp.name`

All measurements exported for a mobile application MUST contain an `mobileApp.name` attribute set to the name of the mobile application.

##### `device.model`

All measurements exported for a mobile application SHOULD contain a `device.model` attribute set to the mobile device model.

##### `browserApp.name`

All measurements exported for a browser application MUST contain an `browserApp.name` attribute set to the name of the browser application.

#### `db.name`

All measurements exported for a database MUST contain an `db.name` attribute set to the database name.
