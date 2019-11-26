# Common Exporter Behavior

This document describes behavior that is common to all New Relic exporters.

# User-Agent values

Each exporter **MUST** report an identifying string and version as part of the `User-Agent` header when sending data to New Relic.

The `User-Agent` values are always in the format `<author>-<language>-<exporter_name>`.  For New Relic exporters, the author is `NewRelic`.

Language names are title cased.

| Exporter | exporter_name | Example User Agent |
| -------- | ------------- | ------------------ |
| [opencensus](opencensus) | `OpenCensus` | `NewRelic-Python-OpenCensus/0.1.0` |
| [micrometer](micrometer) | `Micrometer` | `NewRelic-Java-Micrometer/0.1.0` |
| [kamon](kamon) | `Kamon` | `NewRelic-Java-Kamon/0.1.0` |
| [dropwizard](dropwizard) | `DropWizard` | `NewRelic-Java-DropWizard/0.1.0` |
