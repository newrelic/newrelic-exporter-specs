## Capturing errors in context

### Rationale

Many logging APIs accept an exception or error that is logged alongside a separate message. This exception is also 
often encapsulated within the log event. These exceptions should be included in log message data.

### Attributes

The attributes to include are all optional.

| **Field** | **Type** | **Description** | 
| --------- | -------- | --------------- |
| `error.message` | String | The user-provided message associated with the error | 
| `error.class` | String | The fully-qualified name of the error class, if appropriate to the language |
| `error.stack` | String | The stack reported by the error, formatted natively for the language |
