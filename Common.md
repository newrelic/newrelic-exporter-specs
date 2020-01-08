# Common Exporter Behavior

This document describes behavior that is common to all New Relic exporters.

# User-Agent values

Each exporter **MUST** report an identifying string and version as part of the `User-Agent` header when sending data to New Relic.

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
