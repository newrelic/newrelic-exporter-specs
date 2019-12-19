# Logging

Date: August, 2019

This document describes the serialized format for New Relic Logs-in-Context. The intention of this
document is to provide guidance to anyone writing a log extension for a language or framework not
supported by the existing Logs-in-Context libraries.

## Preconditions

This document assumes the following things:

1. [New Relic Logs](https://docs.newrelic.com/docs/logs/enable-logs/enable-logs/enable-new-relic-logs) is already enabled.
2. The application is instrumented with a New Relic agent that is [capable of providing entity metadata](https://docs.newrelic.com/docs/logs/enable-logs/logs-context-agent-apis/annotate-logs-logs-context-using-apm-agent-apis).
3. The application logs to a file, standard out, standard error, or other application stream that is
read by a [New Relic log output plugin](https://docs.newrelic.com/docs/logs/enable-logs/enable-logs/enable-new-relic-logs#enable-logs).

Please refer to the links above for more information.

## Stream format

The output stream (the log file, or if console, standard output or standard error) from the application should 
meet the following requirements:

1. The stream **MUST** log one line per message, separating with the appropriate platform-specific line separator.
2. The stream **MUST** log in JSON format, with one JSON object per message.
3. The resulting object **MUST** be less than 4096 bytes.
4. The stream **SHOULD** log in UTF-8; however, it's more important that the log forwarder is capable of 
reading the output character set.

## Required fields

| key | type | description |
| --- | ---- |----------- |
| `message` | String | The rendered and internationalized string logged by the user. |
| `timestamp` | 64 bit Integer |The time that the log statement was emitted, in milliseconds since the epoch. |

In addition, all fields returned by the agent's public [linking metadata accessor](https://docs.newrelic.com/docs/logs/enable-logs/logs-context-agent-apis/annotate-logs-logs-context-using-apm-agent-apis)
**MUST** be included.

## Common Optional Fields

Any fields present in the decorated logging framework's message abstraction should be
considered for inclusion in the JSON output.

The naming of these fields should follow New Relic's [attribute naming guidelines](../Guidelines.md#naming-conventions).

Here are some common examples. These examples may or may not apply in all situations. 
These fields will all become queryable attributes on the log messages.

* `log.level`
* `logger.name`
* `thread.id`
* `thread.name`
* `file.name`
* `line.number`
* `class.name`
* `method.name`

### Errors

Many logging APIs accept an exception or error that is logged alongside a separate message. This exception is also 
often encapsulated within the log event. These exceptions should be included in log message data.

| **Field** | **Type** | **Description** | 
| --------- | -------- | --------------- |
| `error.message` | String | The user-provided message associated with the error | 
| `error.class` | String | The fully-qualified name of the error class, if appropriate to the language |
| `error.stack` | String | The stack reported by the error, formatted natively for the language |

## Example

*Note*: This example has been pretty-printed for clarity.  The actual log output should
contain one JSON object per line.

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


