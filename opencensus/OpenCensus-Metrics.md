<!-- # OpenCensus Metric Exporters -->

|OpenCensus Aggregation Type|New Relic Metric Type|How to translate|
|-----|-----|-----|
|Count|Count|Current value minus previous value|
|Last Value|Gauge|Use current value|
|Sum|Count|Current value minus previous value|
|Distribution|Count|<ol><li>Sum: current sum minus previous sum</li><li>Each Bucket: current bucket value minus previous bucket value added to this value for all lower ranged buckets</li></ol>|

## Metric Identities

Metrics are identified by two things in OpenCensus. When aggregating and converting metrics from OpenCensus types to New Relic types, these two attributes must be used to uniquely identify the metric.

1. The name of the View. This name is enforced as unique across all Views. The name also identifies the metric type, therefore there cannot be two views of the same name but with different metric types.
1. The key/values of all Tags, including any custom attributes added by the exporter as explained below.

## Name and Attributes

Each metric must be named after its OpenCensus View. For Distribution type metrics, the suffix `.sum` or `.buckets` must be added (see Distribution section below).

Include these attributes for each metric in addition to creating one for each OpenCensus Tag.

|Attribute|Type|Value|
|-----|-----|-----|
|`measure.name`|string|The name of the OpenCensus Measure used by the View|
|`measure.unit`|string|The unit of the OpenCensus Measure used by the View|
|`aggregation.type`|string|The type of the OpenCensus aggregation used by the View; one of `Count`, `LastValue`, `Sum`, or `Distribution`|

At this time, we will not be adding any common attributes, however we must allow for our users to be able to do so. These attributes must be defined by the user when creating their Exporter and will be added to the `common` block of each metrics post to New Relic.

## Timestamp and Interval

OpenCensus aggregates Count, Sum, and Distribution metrics over the life of a process. Under normal circumstances, the values of these metrics will always increase, and exporters will need to diff the current value with the previous value to find out how much it has increased. All deltas greater than or equal to zero should be sent; sending values of zero means the customer will know the metric was recorded and there has been no change.

OpenCensus instrumentations in some languages allow users to reset or clear the metrics associated with an exporter. If, when comparing a metric to a previous metric, the value has decremented, it can be assumed that the value has been reset, and exporters should send the new value.

This is the same for their Start and End times; Start time will never change and always represents the beginning of the process. End time does change and represents the end time of the current harvest period.

When recording New Relic Count metrics (OpenCensus Count, Sum, and Distribution types), set the `timestamp` to the previous value of the End time. Set the `interval.ms` to the difference between the previous End time and the current End time. If this is the first time this metric is seen, set `timestamp` to the Start time and the `interval.ms` to the difference between the Start and End times.

When recording New Relic Gauge metrics (OpenCensus Last Value type), set the `timestamp` to the current End time. Do not set `interval.ms`.

If for some reason metrics are exported out of order, drop and do not record the current out of order metric. For example, if metrics are exported for 1:00pm, 1:10pm, and 1:05pm in that order, the metrics for 1:05pm must be dropped. This is because OpenCensus metrics are cumulative and we need to send the delta between each newly exported metric with the last one seen. If metrics arrive out of order, calculating the delta will not be feasible.

## Count

These metrics represent the count of the total occurrences of the metric since recording began. Comparison must be done on these metrics before sending.

These metrics must be translated to New Relic's Count metric type. To record them, take the current value of the CountData metric and subtract from it the previous value of the CountData metric. If this is the first time this metric is seen, use its total value. All delta values greater than or equal to zero must be sent.

#### Example

Previous Count

```json
{
  "View": {
    "Name": "Throughput",
    "TagKeys": ["username"],
    "Measure": {
      "Name": "latency",
      "Description": "the amount of time it took for a request to complete",
      "Unit": "ms"
    },
    "Aggregation": {
      "Type": "Count"
    }
  },
  "Start": 1531410000,
  "End": 1531410010,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "username",
          "Value": "sally"
        }
      ],
      "Data": {
        "Value": 10
      }
    }
  ]
}
```

Current Count

```json
{
  "View": {
    "Name": "Throughput",
    "TagKeys": ["username"],
    "Measure": {
      "Name": "latency",
      "Description": "the amount of time it took for a request to complete",
      "Unit": "ms"
    },
    "Aggregation": {
      "Type": "Count"
    }
  },
  "Start": 1531410000,
  "End": 1531410020,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "username",
          "Value": "sally"
        }
      ],
      "Data": {
        "Value": 15
      }
    }
  ]
}
```

New Relic Metric

```json
[{
  "common" : {
    "attributes": {
      "app.name": "foo",
      "host.name": "dev.server.com"
    }
  },
  "metrics": [
    {
      "name": "Throughput",
      "type": "count",
      "value": 5,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "username": "sally",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Count"
      }
    }
  ]
}]
```

## Last Value

These metrics represent the last recorded value of the metric. OpenCensus will drop all previous values. For this reason, it is impossible to determine from the OpenCensus data alone whether or not a metric value was recorded since the last harvest. Therefore, no comparison with previous values should be done.

These metrics must be translated to New Relic's Gauge metric type. To record them, take the current value of the LastValueData metric and set it as the value of the New Relic Gauge metric.

#### Example

Current Last Value

```json
{
  "View": {
    "Name": "MemoryUsage",
    "TagKeys": ["environment"],
    "Measure": {
      "Name": "memory",
      "Description": "the amount memory the process is using",
      "Unit": "bytes"
    },
    "Aggregation": {
      "Type": "LastValue"
    }
  },
  "Start": 1531410000,
  "End": 1531410010,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "environment",
          "Value": "prod"
        }
      ],
      "Data": {
        "Value": 123456
      }
    }
  ]
}
```

New Relic Metric

```json
[{
  "common" : {
    "attributes": {
      "app.name": "bar",
      "host.name": "prod.server.com"
    }
  },
  "metrics": [
    {
      "name": "MemoryUsage",
      "type": "gauge",
      "value": 123456,
      "timestamp": 1531410010,
      "attributes": {
        "environment": "prod",
        "measure.name": "memory",
        "measure.unit": "bytes",
        "aggregation.type": "LastValue"
      }
    }
  ]
}]
```

## Sum

These metrics represent the total sum of all values recorded since recording began. Comparison must be done on these metrics before sending.

These metrics must be translated to New Relic's Count metric type. To record them, take the current value of the SumData metric and subtract from it the previous value of the SumData metric. If this is the first time this metric is seen, use its total value. All delta values greater than or equal to zero must be sent.

#### Example

Previous Sum

```json
{
  "View": {
    "Name": "UserLogins",
    "TagKeys": ["username"],
    "Measure": {
      "Name": "logins",
      "Description": "the number of times a user has logged in",
      "Unit": "logins"
    },
    "Aggregation": {
      "Type": "Sum"
    }
  },
  "Start": 1531410000,
  "End": 1531410010,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "username",
          "Value": "sally"
        }
      ],
      "Data": {
        "Value": 2
      }
    }
  ]
}
```

Current Sum

```json
{
  "View": {
    "Name": "UserLogins",
    "TagKeys": ["username"],
    "Measure": {
      "Name": "logins",
      "Description": "the number of times a user has logged in",
      "Unit": "logins"
    },
    "Aggregation": {
      "Type": "Sum"
    }
  },
  "Start": 1531410000,
  "End": 1531410020,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "username",
          "Value": "sally"
        }
      ],
      "Data": {
        "Value": 3
      }
    }
  ]
}
```

New Relic Metric

```json
[{
  "common" : {
    "attributes": {
      "app.name": "foo",
      "host.name": "dev.server.com"
    }
  },
  "metrics": [
    {
      "name": "UserLogins",
      "type": "count",
      "value": 1,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "username": "sally",
        "measure.name": "logins",
        "measure.unit": "logins",
        "aggregation.type": "Sum"
      }
    }
  ]
}]
```

## Distribution

These metrics give details about the distribution of the recorded values. Comparison must be done on these metrics before sending.

These metrics will be deconstructed and translated into multiple of New Relic's Count metric type.

Record the Sum of the distribution data as a count metric. The Sum can be calculated by multiplying the count value by the mean value. To record the Sum, take the current sum value of the DistributionData and subtract from it the previous sum value of the DistributionData. If this is the first time this metric is seen, use its current sum value. Name this metric `<view name>.sum` where `<view name>` is the name of the view. All delta values greater than or equal to zero must be sent.

Record the Bucket values of the distribution data each as a separate count metric. To do so, for each bucket, take the current count value and subtract from it the previous count value. If this is the first time this metric is seen, use its current count value. Then, add to this value the sum of all delta values of lower buckets. (This transforms the value from the count of items in this bucket to the count of items in this bucket and below. This change allows for line charts to stack properly in the UI.) Name this metric `<view name>.buckets` where `<view name>` is the name of the view. All delta values greater than or equal to zero must be sent. Include two additional attributes on each metric: `lower_bound` and `upper_bound` both string types. The first bucket will have a lower bound of `"-Inf"` (negative infinity) and the last bucket will have the upper bound of `"Inf"` (positive infinity). Adding these attributes allows users to facet on the buckets.

Example code for calculating bucket values:

```python
bucket_total = 0
for i in range(len(buckets):
    # calculate delta for this bucket
    this_bucket = current_values[i] - previous_values[i]
    # add to it the sum of all lower bucket values
    bucket_total += this_bucket
    # record metric
    harvester.RecordMetric(Count(value=bucket_total))

# save values for next export
previous_values = current_values
```

#### Example

Previous Distribution

```json
{
  "View": {
    "Name": "Latency",
    "TagKeys": ["endpoint"],
    "Measure": {
      "Name": "latency",
      "Description": "the amount of time it took for a request to complete",
      "Unit": "ms"
    },
    "Aggregation": {
      "Type": "Distribution",
      "Buckets": [1, 5, 10]
    }
  },
  "Start": 1531410000,
  "End": 1531410010,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "endpoint",
          "Value": "index"
        }
      ],
      "Data": {
        "Count": 4,
        "Min": 0.01,
        "Max": 12.02,
        "Mean": 0.55,
        "SumOfSquaredDev": 0.25,
        "CountPerBucket": [1, 1, 0, 2],
        "ExemplarsPerBucket": [...]
      }
    }
  ]
}
```

Current Distribution

```json
{
  "View": {
    "Name": "Latency",
    "TagKeys": ["endpoint"],
    "Measure": {
      "Name": "latency",
      "Description": "the amount of time it took for a request to complete",
      "Unit": "ms"
    },
    "Aggregation": {
      "Type": "Distribution",
      "Buckets": [1, 5, 10]
    }
  },
  "Start": 1531410000,
  "End": 1531410020,
  "Rows": [
    {
      "Tags": [
        {
          "Key": "endpoint",
          "Value": "index"
        }
      ],
      "Data": {
        "Count": 7,
        "Min": 0.01,
        "Max": 12.06,
        "Mean": 0.57,
        "SumOfSquaredDev": 0.45,
        "CountPerBucket": [1, 2, 1, 3],
        "ExemplarsPerBucket": [...]
      }
    }
  ]
}
```

New Relic Metrics

```json
[{
  "common" : {
    "attributes": {
      "app.name": "foo",
      "host.name": "dev.server.com"
    }
  },
  "metrics": [
    {
      "name": "Latency.sum",
      "type": "count",
      "value": 1.79,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "endpoint": "index",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Distribution"
      }
    },
    {
      "name": "Latency.buckets",
      "type": "count",
      "value": 0,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "endpoint": "index",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Distribution",
        "lower_bound": "-Inf",
        "upper_bound": "1"
      }
    },
    {
      "name": "Latency.buckets",
      "type": "count",
      "value": 1,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "endpoint": "index",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Distribution",
        "lower_bound": "1",
        "upper_bound": "5"
      }
    },
    {
      "name": "Latency.buckets",
      "type": "count",
      "value": 2,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "endpoint": "index",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Distribution",
        "lower_bound": "5",
        "upper_bound": "10"
      }
    },
    {
      "name": "Latency.buckets",
      "type": "count",
      "value": 3,
      "timestamp": 1531410010,
      "interval.ms": 10000,
      "attributes": {
        "endpoint": "index",
        "measure.name": "latency",
        "measure.unit": "ms",
        "aggregation.type": "Distribution",
        "lower_bound": "10",
        "upper_bound": "Inf"
      }
    }
  ]
}]
```
