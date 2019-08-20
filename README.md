# New Relic Exporter Specifications

### Purpose
With this documentation, we intend to document our general principles, and specific implementations of what we generally
call "Exporters". By this, we mean libraries or tools that extract data from existing telemetry systems, convert the
data into a New Relic-friendly format, and send them to New Relic. In general, the implementations of these exporters
will rely on our open-source SDKs to do the work of sending the data to New Relic.

### Organization
See the [Guidelines.md](Guidelines.md) for general principles on how to build exporters and
provide adequate information so that they will be able to be queried by NRQL and visualizations can be created.

Each subdirectory contains a specification for how the exporter for a different metrics library functions.

#### OpenCensus
Cross-language specifications for how we extract data from [OpenCensus](https://opencensus.io/) can be found in the [opencensus](opencensus) directory.

#### DropWizard metrics
The detailed description of how the [New Relic DropWizard Reporter](https://github.com/newrelic/dropwizard-metrics-newrelic)
converts [DropWizard metrics](https://metrics.dropwizard.io) into New Relic dimensional metrics
can be found in the [dropwizard](dropwizard) directory.

#### Micrometer
The detailed description of how the [New Relic Micrometer Registry](https://github.com/newrelic/micrometer-registry-newrelic)
converts [Micrometer metrics](https://micrometer.io/) into New Relic dimensional metrics
can be found in the [micrometer](micrometer) directory.

### Contributing
Full details are available in [CONTRIBUTING.md](CONTRIBUTING.md).

### Licensing
License details are in [LICENSE.md](LICENSE.md)..
