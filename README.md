# New Relic Exporter Specifications

### Purpose
With this documentation, we intend to document our general principles, and specific implementations of what we generally
call "Exporters". By this, we mean libraries or tools that extract data from existing telemetry systems, convert the
data into a New Relic-friendly format, and send them to New Relic. In general, the implementations of these exporters
will rely on our open-source SDKs to do the work of sending the data to New Relic.

### Intent
This documentation is divided into two sections, with two different intents. Documentation at the top level of this project
is generally intended to be _prescriptive_. If you are designing an exporter, we strongly recommend following
the [Guidelines.md](Guidelines.md), as it will support interoperability between the metrics exported from various metric libraries. 
The documentation in the subdirectories, of the specific exporters, however, 
are intended to be purely _descriptive_. They should reflect exactly what the current version of the exporter 
actually does, even if it doesn't exactly follow the guidelines. 

### Existing available exporter repositories

We have repositories for several New Relic exporters already. The full table can be found at the docs.newrelic.com site.  It 
includes at least one language versions for all of the exporters listed in the Organization section below.

### Organization
See the [Guidelines.md](Guidelines.md) for general principles on how to build exporters and
provide adequate information so that they will be able to be queried by NRQL and visualizations can be created.

Each subdirectory contains a specification for how the relevant exporter functions.

#### OpenTelemetry
Cross-language specifications for how we extract data from [OpenTelemetry](https://opentelemetry.io/) can be found in the
[opentelemetry](opentelemetry) directory.

#### OpenCensus
Cross-language specifications for how we extract data from [OpenCensus](https://opencensus.io/) can be found in the
[opencensus](opencensus) directory.

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
