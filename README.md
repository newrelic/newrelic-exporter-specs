# New Relic Exporter Specifications

### Purpose
With this documentation, we intend to document our general principles, and specific implementations of what we generally
call "Exporters". By this, we mean libraries or tools that extract data from existing telemetry systems, convert the 
data into a New Relic-friendly format, and send them to New Relic. In general, the implementations of these exporters
will rely on our open-source SDKs to do the work of sending the data to New Relic. 

### Organization
See the [Guidelines.md] for general principles on how to build exporters and 
provide adequate information so that they will be able to be queried by NRQL and visualizations can be created.

Each subdirectory contains a specification for how the exporter for a different metrics library functions.

#### OpenCensus
Cross-language specifications for how we extract data from [OpenCensus](https://opencensus.io/) can be found in the [opencensus] directory.

#### DropWizard metrics
The detailed description of how the [New Relic DropWizard Reporter](https://github.com/newrelic/dropwizard-metrics-newrelic) 
converts [DropWizard metrics](https://metrics.dropwizard.io) into New Relic dimensional metrics
can be found in the [dropwizard] directory.

#### Micrometer
The detailed description of how the [New Relic Micrometer Registry](https://github.com/newrelic/micrometer-registry-newrelic) 
converts [Micrometer metrics](https://micrometer.io/) into New Relic dimensional metrics
can be found in the [micrometer] directory. 

### Licensing
The contents of this repository are licensed under the Apache 2.0 License.

### Contributing
Full details are available in our [CONTRIBUTING.md] file. 

We'd love to get your contributions to improve the New Relic Exporter specifications! 
Keep in mind when you submit your pull request, you'll need to sign the CLA via the click-through using [CLA-Assistant](https://cla-asisstant.io).
You only have to sign the CLA one time per project.
To execute our corporate CLA, which is required if your contribution is on behalf of a company, or if you have any questions, 
please drop us an email at open-source@newrelic.com. 
