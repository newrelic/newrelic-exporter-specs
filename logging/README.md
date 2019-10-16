# Logging

Date: August, 2019


This document describes how to provide an extension to logging frameworks so that New
Relic's logging platform can associate log statements with traces and entities.

The infrastructure between the log output from the process running an agent and New
Relic's log endpoint is out of scope for this spec.

Using this extension, it **MUST** be possible to configure an application's logging to
emit log statements formatted as JSON objects, encoded in UTF-8, with one JSON object per
line.

### Required fields

| key | type | description |
| --- | ---- |----------- |
| `message` | String | The rendered and internationalized string logged by the user. |
| `timestamp` | 64 bit Integer |The time that the log statement was emitted, in milliseconds since the epoch. |

In addition, all fields returned by the agent's public linking metadata accessor
**MUST** be included.

### Common Optional Fields

Any fields present in the decorated logging framework's message abstraction should be
considered for inclusion in the JSON output.  When extending a new logging framework, the
PM for the language should decide which fields should be included.

The naming of these fields should follow New Relic's [attribute naming guidelines](../Guidelines.md#naming-conventions).

Some common examples:
* `log.level`
* `logger.name`
* `thread.id`
* `thread.name`
* `file.name`
* `line.number`
* `class.name`
* `method.name`

#### Errors

Many logging APIs accept an exception or error that is logged alongside a separate message. This exception is also 
often encapsulated within the log event. These exceptions should be included in log message data.

| **Field** | **Type** | **Description** | 
| --------- | -------- | --------------- |
| `error.message` | String | The user-provided message associated with the error | 
| `error.class` | String | The fully-qualified name of the error class, if appropriate to the language |
| `error.stack` | String | The stack reported by the error, formatted natively for the language |

## Example

```json
{
  "message": "This is a secondary error in the same span as the error message",
  "timestamp": 1566933234440,
  "thread.name": "Thread-18",
  "log.level": "ERROR",
  "logger.name": "com.newrelic.example.Main",
  "class.name": "com.newrelic.example.Main",
  "method.name": "transactionWithError",
  "line.number": "56",
  "trace.id": "b3b20378e5fda012",
  "hostname": "examplehost.newrelic.com",
  "entity.type": "SERVICE",
  "entity.guid": "MzgwNTYyfEFQTXxBUFBMSUNBVElPTnwxMDQxOTkyNQ",
  "entity.name": "log-plugins",
  "span.id": "ce69844fccfd4fcc"
}
{
  "message": "The actual async method: token linkAndExpire success? true",
  "timestamp": 1566933234449,
  "thread.name": "Thread-22",
  "log.level": "ERROR",
  "logger.name": "com.newrelic.example.Main",
  "class.name": "com.newrelic.example.Main",
  "method.name": "theAsyncMethod",
  "line.number": "75",
  "trace.id": "596988f0dc8c5e7a",
  "hostname": "examplehost.newrelic.com",
  "entity.type": "SERVICE",
  "entity.guid": "MzgwNTYyfEFQTXxBUFBMSUNBVElPTnwxMDQxOTkyNQ",
  "entity.name": "log-plugins",
  "span.id": "6333786127ffa512"
}
```

The above example has been pretty-printed for clarity.  The actual log output should
contain one JSON object per line.
