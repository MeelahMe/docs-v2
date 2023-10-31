---
title: NoOpNode
description: >
  The `noOp` node does not perform any operation.
note: Auto generated by tickdoc
menu:
  kapacitor_v1:
    name: NoOpNode
    identifier: no_op_node
    weight: 100
    parent: nodes
aliases:
  - /kapacitor/v1/nodes/no_op_node/
---

The `noOp` node does not perform any operation.

> Do not use this node in a TICKscript. There should be no need for it.

If a node does not have any children, then its emitted count remains zero.
Using a [NoOpNode](/kapacitor/v1/reference/nodes/no_op_node/) is a work around so that statistics are accurately reported
for nodes with no real children.
A [NoOpNode](/kapacitor/v1/reference/nodes/no_op_node/) is automatically appended to any node that is a source for a [StatsNode](/kapacitor/v1/reference/nodes/stats_node/)
and does not have any children.


### Constructor

| Chaining Method | Description |
|:---------|:---------|
| **noOp** | Has no constructor signature. |

### Property Methods

| Setters | Description |
|:---|:---|
| **[quiet](#quiet)&nbsp;(&nbsp;)** | Suppress all error logging events from this node.  |


### Chaining Methods
[Alert](#alert),
[Barrier](#barrier),
[Bottom](#bottom),
[ChangeDetect](#changedetect),
[Combine](#combine),
[Count](#count),
[CumulativeSum](#cumulativesum),
[Deadman](#deadman),
[Default](#default),
[Delete](#delete),
[Derivative](#derivative),
[Difference](#difference),
[Distinct](#distinct),
[Ec2Autoscale](#ec2autoscale),
[Elapsed](#elapsed),
[Eval](#eval),
[First](#first),
[Flatten](#flatten),
[GroupBy](#groupby),
[HoltWinters](#holtwinters),
[HoltWintersWithFit](#holtwinterswithfit),
[HttpOut](#httpout),
[HttpPost](#httppost),
[InfluxDBOut](#influxdbout),
[Join](#join),
[K8sAutoscale](#k8sautoscale),
[KapacitorLoopback](#kapacitorloopback),
[Last](#last),
[Log](#log),
[Max](#max),
[Mean](#mean),
[Median](#median),
[Min](#min),
[Mode](#mode),
[MovingAverage](#movingaverage),
[Percentile](#percentile),
[Sample](#sample),
[Shift](#shift),
[Sideload](#sideload),
[Spread](#spread),
[StateCount](#statecount),
[StateDuration](#stateduration),
[Stats](#stats),
[Stddev](#stddev),
[Sum](#sum),
[SwarmAutoscale](#swarmautoscale),
[Top](#top),
[Trickle](#trickle),
[Union](#union),
[Where](#where),
[Window](#window)

---


## Properties

Property methods modify state on the calling node.
They do not add another node to the pipeline, and always return a reference to the calling node.
Property methods are marked using the `.` operator.


### Quiet

Suppress all error logging events from this node.

```js
noOp.quiet()
```


## Chaining Methods

Chaining methods create a new node in the pipeline as a child of the calling node.
They do not modify the calling node.
Chaining methods are marked using the `|` operator.


### Alert

Create an alert node, which can trigger alerts.


```js
noOp|alert()
```

Returns: [AlertNode](/kapacitor/v1/reference/nodes/alert_node/)

### Barrier

Create a new Barrier node that emits a BarrierMessage periodically.

One BarrierMessage will be emitted every period duration.


```js
noOp|barrier()
```

Returns: [BarrierNode](/kapacitor/v1/reference/nodes/barrier_node/)

### Bottom

Select the bottom `num` points for `field` and sort by any extra tags or fields.


```js
noOp|bottom(num int64, field string, fieldsAndTags ...string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### ChangeDetect

Create a new node that only emits new points if different from the previous point.

```js
noOp|changeDetect(field string)
```

Returns: [ChangeDetectNode](/kapacitor/v1/reference/nodes/change_detect_node/)

### Combine

Combine this node with itself. The data is combined on timestamp.


```js
noOp|combine(expressions ...ast.LambdaNode)
```

Returns: [CombineNode](/kapacitor/v1/reference/nodes/combine_node/)

### Count

Count the number of points.


```js
noOp|count(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### CumulativeSum

Compute a cumulative sum of each point that is received.
A point is emitted for every point collected.


```js
noOp|cumulativeSum(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Deadman

Helper function for creating an alert on low throughput, a.k.a. deadman's switch.

- Threshold: trigger alert if throughput drops below threshold in points/interval.
- Interval: how often to check the throughput.
- Expressions: optional list of expressions to also evaluate. Useful for time of day alerting.

Example:


```js
    var data = stream
        |from()...
    // Trigger critical alert if the throughput drops below 100 points per 10s and checked every 10s.
    data
        |deadman(100.0, 10s)
    //Do normal processing of data
    data...
```

The above is equivalent to this example:


```js
    var data = stream
        |from()...
    // Trigger critical alert if the throughput drops below 100 points per 10s and checked every 10s.
    data
        |stats(10s)
            .align()
        |derivative('emitted')
            .unit(10s)
            .nonNegative()
        |alert()
            .id('node \'stream0\' in task \'{{ .TaskName }}\'')
            .message('{{ .ID }} is {{ if eq .Level "OK" }}alive{{ else }}dead{{ end }}: {{ index .Fields "emitted" | printf "%0.3f" }} points/10s.')
            .crit(lambda: "emitted" <= 100.0)
    //Do normal processing of data
    data...
```

The `id` and `message` alert properties can be configured globally via the 'deadman' configuration section.

Since the [AlertNode](/kapacitor/v1/reference/nodes/alert_node/) is the last piece it can be further modified as usual.
Example:


```js
    var data = stream
        |from()...
    // Trigger critical alert if the throughput drops below 100 points per 10s and checked every 10s.
    data
        |deadman(100.0, 10s)
            .slack()
            .channel('#dead_tasks')
    //Do normal processing of data
    data...
```

You can specify additional lambda expressions to further constrain when the deadman's switch is triggered.
Example:


```js
    var data = stream
        |from()...
    // Trigger critical alert if the throughput drops below 100 points per 10s and checked every 10s.
    // Only trigger the alert if the time of day is between 8am-5pm.
    data
        |deadman(100.0, 10s, lambda: hour("time") >= 8 AND hour("time") <= 17)
    //Do normal processing of data
    data...
```



```js
noOp|deadman(threshold float64, interval time.Duration, expr ...ast.LambdaNode)
```

Returns: [AlertNode](/kapacitor/v1/reference/nodes/alert_node/)

### Default

Create a node that can set defaults for missing tags or fields.


```js
noOp|default()
```

Returns: [DefaultNode](/kapacitor/v1/reference/nodes/default_node/)

### Delete

Create a node that can delete tags or fields.


```js
noOp|delete()
```

Returns: [DeleteNode](/kapacitor/v1/reference/nodes/delete_node/)

### Derivative

Create a new node that computes the derivative of adjacent points.


```js
noOp|derivative(field string)
```

Returns: [DerivativeNode](/kapacitor/v1/reference/nodes/derivative_node/)

### Difference

Compute the difference between points independent of elapsed time.


```js
noOp|difference(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Distinct

Produce batch of only the distinct points.


```js
noOp|distinct(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Ec2Autoscale

Create a node that can trigger autoscale events for a ec2 autoscalegroup.


```js
noOp|ec2Autoscale()
```

Returns: [Ec2AutoscaleNode](/kapacitor/v1/reference/nodes/ec2_autoscale_node/)

### Elapsed

Compute the elapsed time between points.


```js
noOp|elapsed(field string, unit time.Duration)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Eval

Create an eval node that will evaluate the given transformation function to each data point.
A list of expressions may be provided and will be evaluated in the order they are given.
The results are available to later expressions.


```js
noOp|eval(expressions ...ast.LambdaNode)
```

Returns: [EvalNode](/kapacitor/v1/reference/nodes/eval_node/)

### First

Select the first point.


```js
noOp|first(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Flatten

Flatten points with similar times into a single point.


```js
noOp|flatten()
```

Returns: [FlattenNode](/kapacitor/v1/reference/nodes/flatten_node/)

### GroupBy

Group the data by a set of tags.

Can pass literal * to group by all dimensions.
Example:


```js
    |groupBy(*)
```



```js
noOp|groupBy(tag ...interface{})
```

Returns: [GroupByNode](/kapacitor/v1/reference/nodes/group_by_node/)

### HoltWinters

Compute the Holt-Winters (/influxdb/v1/query_language/functions/#holt-winters) forecast of a data set.


```js
noOp|holtWinters(field string, h int64, m int64, interval time.Duration)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### HoltWintersWithFit

Compute the Holt-Winters (/influxdb/v1/query_language/functions/#holt-winters) forecast of a data set.
This method also outputs all the points used to fit the data in addition to the forecasted data.


```js
noOp|holtWintersWithFit(field string, h int64, m int64, interval time.Duration)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### HttpOut

Create an HTTP output node that caches the most recent data it has received.
The cached data is available at the given endpoint.
The endpoint is the relative path from the API endpoint of the running task.
For example, if the task endpoint is at `/kapacitor/v1/tasks/<task_id>` and endpoint is
`top10`, then the data can be requested from `/kapacitor/v1/tasks/<task_id>/top10`.


```js
noOp|httpOut(endpoint string)
```

Returns: [HTTPOutNode](/kapacitor/v1/reference/nodes/http_out_node/)

### HttpPost

Creates an HTTP Post node that POSTS received data to the provided HTTP endpoint.
HttpPost expects 0 or 1 arguments. If 0 arguments are provided, you must specify an
endpoint property method.


```js
noOp|httpPost(url ...string)
```

Returns: [HTTPPostNode](/kapacitor/v1/reference/nodes/http_post_node/)

### InfluxDBOut

Create an influxdb output node that will store the incoming data into InfluxDB.


```js
noOp|influxDBOut()
```

Returns: [InfluxDBOutNode](/kapacitor/v1/reference/nodes/influx_d_b_out_node/)

### Join

Join this node with other nodes. The data is joined on timestamp.


```js
noOp|join(others ...Node)
```

Returns: [JoinNode](/kapacitor/v1/reference/nodes/join_node/)

### K8sAutoscale

Create a node that can trigger autoscale events for a kubernetes cluster.


```js
noOp|k8sAutoscale()
```

Returns: [K8sAutoscaleNode](/kapacitor/v1/reference/nodes/k8s_autoscale_node/)

### KapacitorLoopback

Create an kapacitor loopback node that will send data back into Kapacitor as a stream.


```js
noOp|kapacitorLoopback()
```

Returns: [KapacitorLoopbackNode](/kapacitor/v1/reference/nodes/kapacitor_loopback_node/)

### Last

Select the last point.


```js
noOp|last(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Log

Create a node that logs all data it receives.


```js
noOp|log()
```

Returns: [LogNode](/kapacitor/v1/reference/nodes/log_node/)

### Max

Select the maximum point.


```js
noOp|max(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Mean

Compute the mean of the data.


```js
noOp|mean(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Median

Compute the median of the data.

> **Note:** This method is not a selector.
If you want the median point, use `.percentile(field, 50.0)`.


```js
noOp|median(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Min

Select the minimum point.


```js
noOp|min(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Mode

Compute the mode of the data.


```js
noOp|mode(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### MovingAverage

Compute a moving average of the last window points.
No points are emitted until the window is full.


```js
noOp|movingAverage(field string, window int64)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Percentile

Select a point at the given percentile. This is a selector function, no interpolation between points is performed.


```js
noOp|percentile(field string, percentile float64)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Sample

Create a new node that samples the incoming points or batches.

One point will be emitted every count or duration specified.


```js
noOp|sample(rate interface{})
```

Returns: [SampleNode](/kapacitor/v1/reference/nodes/sample_node/)

### Shift

Create a new node that shifts the incoming points or batches in time.


```js
noOp|shift(shift time.Duration)
```

Returns: [ShiftNode](/kapacitor/v1/reference/nodes/shift_node/)

### Sideload

Create a node that can load data from external sources.


```js
noOp|sideload()
```

Returns: [SideloadNode](/kapacitor/v1/reference/nodes/sideload_node/)

### Spread

Compute the difference between `min` and `max` points.


```js
noOp|spread(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### StateCount

Create a node that tracks number of consecutive points in a given state.


```js
noOp|stateCount(expression ast.LambdaNode)
```

Returns: [StateCountNode](/kapacitor/v1/reference/nodes/state_count_node/)

### StateDuration

Create a node that tracks duration in a given state.


```js
noOp|stateDuration(expression ast.LambdaNode)
```

Returns: [StateDurationNode](/kapacitor/v1/reference/nodes/state_duration_node/)

### Stats

Create a new stream of data that contains the internal statistics of the node.
The interval represents how often to emit the statistics based on real time.
This means the interval time is independent of the times of the data points the source node is receiving.


```js
noOp|stats(interval time.Duration)
```

Returns: [StatsNode](/kapacitor/v1/reference/nodes/stats_node/)

### Stddev

Compute the standard deviation.


```js
noOp|stddev(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Sum

Compute the sum of all values.


```js
noOp|sum(field string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### SwarmAutoscale

Create a node that can trigger autoscale events for a Docker swarm cluster.


```js
noOp|swarmAutoscale()
```

Returns: [SwarmAutoscaleNode](/kapacitor/v1/reference/nodes/swarm_autoscale_node/)

### Top

Select the top `num` points for `field` and sort by any extra tags or fields.


```js
noOp|top(num int64, field string, fieldsAndTags ...string)
```

Returns: [InfluxQLNode](/kapacitor/v1/reference/nodes/influx_q_l_node/)

### Trickle

Create a new node that converts batch data to stream data.

```js
noOp|trickle()
```

Returns: [TrickleNode](/kapacitor/v1/reference/nodes/trickle_node/)

### Union

Perform the union of this node and all other given nodes.


```js
noOp|union(node ...Node)
```

Returns: [UnionNode](/kapacitor/v1/reference/nodes/union_node/)

### Where

Create a new node that filters the data stream by a given expression.


```js
noOp|where(expression ast.LambdaNode)
```

Returns: [WhereNode](/kapacitor/v1/reference/nodes/where_node/)

### Window

Create a new node that windows the stream by time.

NOTE: Window can only be applied to stream edges.


```js
noOp|window()
```

Returns: [WindowNode](/kapacitor/v1/reference/nodes/window_node/)