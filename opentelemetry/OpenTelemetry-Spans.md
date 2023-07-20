# OpenTelemetry Span Exporters

For OpenTelemetry exporters.

In general, generating New Relic spans from OpenTelemetry spans is quite straightforward.
Here is the process:

1. For the `"collector.name"` attribute, use `"newrelic-opentelemetry-exporter"`
1. For `"instrumentation.provider"`, use `"opentelemetry"`
1. Assign the span name, span id, parent span id (if present) and trace id to the New Relic span.
1. Take all OpenTelemetry span attributes, and map them directly over to New Relic span attributes. OpenTelemetry span attributes that have the same name as the attributes called out in this spec should not be mapped over to the New Relic span.
1. The OpenTelemetry span status is comprised of a status code and optional description. The status
code is `Unset`, `Ok`, or `Error`. If the status code is not `Unset`, then add an `otel.status_code`
attribute and set it to the value of the status code. If the status code is not `Unset` and the status has a description then add an
`otel.status_description` attribute and set it to the value of the description.
1. If there is a Resource associated with the Span, take all of the Resource's labels and add them as
span attributes.
1. If there is instrumentation library information associated with the span,
add the corresponding `"instrumentation.name"` and `"instrumentation.version"` attributes to the span, if non-empty.
