# OpenCensus Trace Exporters

## Exporting Spans

Each span received by the OpenCensus Exporter must be transformed into a New Relic span before being sent.

|NR Span Field|OpenCensus Span Field|Type|Required|Comments|
|-----|-----|-----|-----|-----|
|`id`|`Span.SpanID`|string|yes|Unique identifier for this span|
|`trace.id`|`Span.SpanContext.TraceID`|string|yes|Unique identifier shared by all spans within a single trace|
|`name`|`Span.Name`|string|yes|The name of this span|
|`parent.id`|`Span.ParentSpanID`|string|yes (except root span)|The span id of the previous caller of this span. Can be empty if this is the first span in which case the key should be omitted.|
|`timestamp`|`Span.StartTime`|long|yes|Epoch ms timestamp|
|`duration.ms`|`Span.EndTime` - `Span.StartTime`|float|yes|Duration of this span in milliseconds|
|`service.name`|(none)|string|yes| The name of the service that created this span. This value is not gathered from OpenCensus. Instead it should be set directly by the user when defining the Exporter.|
|`attributes`|`Span.Attributes`|map|no|Map of user specified "tags" on this span. Keys are strings, values can be any of bool, long, float, or string. Key should be omitted if empty.|
|`attributes.error`|`Span.Status.Code` not in `IgnoreStatusCodes` list and is not 0|bool|no|When `Span.Stats.Code` is not `0` and is not in the `IgnoreStatusCodes` list, include the value `"error": true` in the `tags` map. If an `error` tag was already set by the user, do not override it.|

## Configuration

When creating their Exporter, users must be able to configure them. The following items must be configurable:

|Configuration|Type|Comments|
|-----|-----|-----|
|`APIKey`|string|The API used to send spans to New Relic|
|`EntityName`|string|The name of this entity or application|
|`IgnoreStatusCodes`|list of int|When a span's status code is in this list, the span will not be recorded as an error. The value 0, while not in this list, will also never be considered an error.|
