<a href="https://opensource.newrelic.com/oss-category/#community-project"><picture><source media="(prefers-color-scheme: dark)" srcset="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/dark/Community_Project.png"><source media="(prefers-color-scheme: light)" srcset="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/Community_Project.png"><img alt="New Relic Open Source community project banner." src="https://github.com/newrelic/opensource-website/raw/main/src/images/categories/Community_Project.png"></picture></a>

# New Relic exporter specifications

### Purpose
With this documentation, we intend to document our general principles, and specific implementations of what we generally
call "exporters". By this, we mean libraries or tools that extract data from existing telemetry systems, convert the
data into a New Relic-friendly format, and send them to New Relic. In general, the implementations of these exporters
will rely on our open-source [SDKs](https://docs.newrelic.com/docs/data-ingest-apis/get-data-new-relic/new-relic-sdks/telemetry-sdks-send-custom-telemetry-data-new-relic) to do the work of sending the data to New Relic.

### Intent
This documentation is divided into two sections, with two different intents. Documentation at the top level of this project
is generally intended to be _prescriptive_. If you are designing an exporter, we strongly recommend following
the [Guidelines.md](Guidelines.md), as it will support interoperability between the metrics exported from various metric libraries. 
The documentation in the subdirectories, of the specific exporters, however, are intended to be purely _descriptive_. They should reflect exactly what the current version of the exporter actually does, even if it doesn't exactly follow the guidelines. 

### Existing available exporter repositories

We have repositories for several New Relic exporters already. The full table can be found in our Telemetry SDK section 
on [docs.newrelic.com](https://docs.newrelic.com/docs/data-ingest-apis/get-data-new-relic/new-relic-sdks/telemetry-sdks-send-custom-telemetry-data-new-relic#external-data).  It includes at least one language versions 
for all of the exporters listed in the Organization section below.

### Organization
See [Guidelines.md](Guidelines.md) for general principles on how to build exporters and
provide adequate information so that they will be able to be queried by [NRQL](https://docs.newrelic.com/docs/query-data/nrql-new-relic-query-language/getting-started/introduction-nrql) and visualizations can be created.

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
The detailed description of how the [New Relic Micrometer registry](https://github.com/newrelic/micrometer-registry-newrelic)
converts [Micrometer metrics](https://micrometer.io/) into New Relic dimensional metrics
can be found in the [micrometer](micrometer) directory.

### Support

For support on this project, visit New Relic's [Explorers Hub](https://forum.newrelic.com/s/). 

### Contributing
Full details are available in [CONTRIBUTING.md](CONTRIBUTING.md).

**A note about vulnerabilities**

As noted in our [security policy](../../security/policy), New Relic is committed to the privacy and security of our customers and their data. We believe that providing coordinated disclosure by security researchers and engaging with the security community are important means to achieve our security goals.

If you believe you have found a security vulnerability in this project or any of New Relic's products or websites, we welcome and greatly appreciate you reporting it to New Relic through [HackerOne](https://hackerone.com/newrelic).

### Licensing
License details are in [LICENSE.md](LICENSE.md).
