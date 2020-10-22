# OpenTelemetry Span Exporters

For OpenTelemetry exporters.

In general, generating New Relic spans from OpenTelemetry spans is quite straightforward.
Here is the process:

1. For the `"collector.name"` attribute, use `"newrelic-opentelemetry-exporter"`
1. For `"instrumentation.provider"`, use `"opentelemetry"`
1. Assign the span name, span id, parent span id (if present) and trace id to the New Relic span.
1. Take all OpenTelemetry span attributes, and map them directly over to New Relic span attributes.
1. If the OpenTelemetry span has a `status` that is not `OK`, and there is a status description available,
add an `"error.message"` attribute that contains the status description.
1. If there is a Resource associated with the Span, take all of the Resource's attributes and add them as
span attributes. One exception is the `service.name` attribute, if the exporter has been configured with a
service name, then this name should take precedence over the `service.name` attribute on the resource.
1. If there is instrumentation library information associated with the span,
add the corresponding `"instrumentation.name"` and `"instrumentation.version"` attributes to the span, if non-empty.
