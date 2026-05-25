---
title: MQTT Sparkplug B Semantic Conventions
linkTitle: MQTT Sparkplug
description: >-
  Experimental semantic conventions for telemetry collected from MQTT
  Sparkplug B payloads, covering topic structure, device identity,
  metric naming, and the Sparkplug B quality model.
status: experimental
weight: 50
---

> **Status: Experimental.**
> These conventions are not part of the official OpenTelemetry semantic
> conventions specification.

## Overview

MQTT is a lightweight publish/subscribe messaging protocol widely used in
industrial and IoT environments. On its own, MQTT defines only a transport
mechanism — it imposes no structure on topic names or message payloads.

**Sparkplug B** is a specification built on top of MQTT that adds a
standardised topic namespace, a defined payload format using Protocol
Buffers, and a session management model with birth and death certificates.
It was designed specifically for industrial SCADA and IIoT applications.

These conventions cover MQTT with the Sparkplug B payload specification.
Plain MQTT without Sparkplug B is not covered here — the absence of a
standardised payload format makes generic attribute definitions impractical.

For general data quality conventions that apply across all protocols, see
[Data Quality](../data-quality/).
For site, asset, and controller attributes, see [Common](../common/).

---

## Sparkplug B data model

### Topic namespace

Sparkplug B defines a strict topic structure:

```
spBv1.0/<group_id>/<message_type>/<edge_node_id>[/<device_id>]
```

| Component | Description |
|---|---|
| `spBv1.0` | Sparkplug B version namespace — always this literal value |
| `group_id` | Logical grouping of edge nodes, typically a site or plant area |
| `message_type` | Message type — see below |
| `edge_node_id` | Identifier of the edge node (gateway or controller) |
| `device_id` | Identifier of a device connected to the edge node (optional) |

### Message types

| Message type | Direction | Description |
|---|---|---|
| `NBIRTH` | Edge node → broker | Edge node birth certificate — announces the node and its metrics |
| `NDEATH` | Broker (on behalf of node) | Edge node death certificate — published by broker when node disconnects |
| `NDATA` | Edge node → broker | Edge node metric data update |
| `NCMD` | Application → edge node | Command to the edge node |
| `DBIRTH` | Edge node → broker | Device birth certificate — announces a device and its metrics |
| `DDEATH` | Edge node → broker | Device death certificate — device has disconnected from the edge node |
| `DDATA` | Edge node → broker | Device metric data update |
| `DCMD` | Application → edge node | Command to a device |

For OTel telemetry collection, the relevant message types are `NDATA`,
`DDATA`, `NBIRTH`, and `DBIRTH`. Birth messages carry the initial metric
definitions and should be used to populate resource attributes. Death
messages are significant quality events and should be recorded as log
events.

### Metrics

Each Sparkplug B payload contains one or more **metrics**. A metric has:

- A **name** — a string path that identifies the data point, for example
  `BearingTemperature` or `Motors/Pump01/Speed`
- A **value** — typed: int, float, boolean, string, dataset, or template
- A **timestamp** — milliseconds since Unix epoch, set by the edge node
- A **quality** flag — a boolean indicating whether the value is good
- An optional **alias** — an integer that substitutes for the name in
  subsequent DDATA messages after DBIRTH, to reduce payload size

---

## Proposed attributes

### Topic attributes

These attributes are derived from the Sparkplug B topic structure and
identify the origin of the telemetry within the Sparkplug B namespace.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.sparkplug.group_id` | string | `plant-01` | Sparkplug B group ID from the topic |
| `industrial.sparkplug.edge_node_id` | string | `edge-packaging-01` | Sparkplug B edge node ID from the topic |
| `industrial.sparkplug.device_id` | string | `mixer-07` | Sparkplug B device ID from the topic — absent for node-level metrics |
| `industrial.sparkplug.message_type` | string | `DDATA` | Sparkplug B message type |

### Metric attributes

These attributes describe the individual Sparkplug B metric within a payload.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.sparkplug.metric_name` | string | `BearingTemperature` | Sparkplug B metric name as published |
| `industrial.sparkplug.metric_alias` | int | `42` | Numeric alias substituting for the metric name after DBIRTH |
| `industrial.sparkplug.metric_type` | string | `Float` | Sparkplug B metric data type |
| `industrial.sparkplug.is_historical` | boolean | `false` | True if the value is a historical backfill, not a live reading |
| `industrial.sparkplug.is_transient` | boolean | `false` | True if the value is transient and should not be stored |

### Broker and session attributes

These attributes identify the MQTT broker and connection context. They are
most appropriate as resource attributes set once on the receiver.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.mqtt.broker_uri` | string | `mqtt://broker-01:1883` | URI of the MQTT broker |
| `industrial.mqtt.client_id` | string | `otel-collector-01` | MQTT client ID of the OTel Collector |
| `industrial.mqtt.qos` | int | `1` | MQTT QoS level used for the subscription |
| `industrial.sparkplug.version` | string | `spBv1.0` | Sparkplug B version namespace from the topic |

### Recommended values for `industrial.sparkplug.metric_type`

Sparkplug B defines the following metric data types:

```
Int8, Int16, Int32, Int64
UInt8, UInt16, UInt32, UInt64
Float, Double
Boolean
String
DateTime
Text
UUID
DataSet
Bytes
File
Template
```

---

## Sparkplug B quality model

Sparkplug B includes a **quality** boolean field in each metric. When
`quality` is `false`, the edge node is indicating that the value should
not be trusted — typically because the edge node has lost communication
with the underlying device or sensor.

### Mapping to general quality attributes

| Sparkplug B quality | `industrial.data.quality` | `industrial.data.quality.detail` |
|---|---|---|
| `true` | `good` | — |
| `false` | `bad` | `sparkplug_quality_false` |
| Quality field absent | `uncertain` | `quality_not_reported` |

### Birth and death certificates as quality events

Sparkplug B uses birth and death certificates to manage session state.
These have direct quality implications and should be handled as follows:

**NBIRTH / DBIRTH** — the edge node or device has come online and is
publishing its current metric definitions. Following a birth message,
values should be treated as `good` unless the quality flag indicates
otherwise.

**NDEATH / DDEATH** — the edge node or device has gone offline. All
metrics previously published by that node or device should transition
to `uncertain` with `industrial.data.quality.detail: node_offline` or
`device_offline` respectively, and remain in that state until a new
birth message is received.

Death events should be recorded as OTel log events with the following
attributes:

| Attribute | Value |
|---|---|
| `industrial.sparkplug.message_type` | `NDEATH` or `DDEATH` |
| `industrial.sparkplug.group_id` | Group ID from the topic |
| `industrial.sparkplug.edge_node_id` | Edge node ID from the topic |
| `industrial.sparkplug.device_id` | Device ID if DDEATH |
| `industrial.data.quality` | `bad` |
| `industrial.data.quality.detail` | `node_offline` or `device_offline` |

### Timestamps

Sparkplug B metrics carry a timestamp set by the edge node, in
milliseconds since Unix epoch. This maps to `industrial.timestamp.source:
gateway`, since the timestamp is set at the edge node rather than at the
source device.

Where the underlying device provides its own timestamp and the edge node
propagates it, `industrial.timestamp.source: device` is appropriate —
but this is device and edge node implementation dependent.

---

## Usage notes

- `industrial.sparkplug.group_id` maps naturally to `industrial.site` or
  `industrial.area` in many deployments, but the mapping depends on how
  the Sparkplug B namespace was designed. Do not assume they are equivalent
  — record both and document the relationship for your deployment.

- `industrial.sparkplug.device_id` often corresponds to
  `industrial.asset.id`, but again the mapping is deployment-specific.
  Record both attributes explicitly rather than relying on one to imply
  the other.

- `industrial.sparkplug.metric_alias` is an internal Sparkplug B
  optimisation. It should not be used as a stable identifier for a metric
  across sessions — aliases are reassigned in each DBIRTH. Always resolve
  aliases to metric names before recording telemetry.

- `industrial.sparkplug.is_historical` is important for analytics and
  ML pipelines. Historical backfill values should be clearly distinguished
  from live readings to avoid distorting real-time dashboards and alerts.

---

## Example

A bearing temperature reading from a Sparkplug B device, with full
context attributes:

```yaml
metric: industrial.temperature
value: 72.4
unit: fahrenheit
attributes:
  # Sparkplug B topic and metric
  industrial.sparkplug.version: spBv1.0
  industrial.sparkplug.group_id: plant-01
  industrial.sparkplug.edge_node_id: edge-packaging-01
  industrial.sparkplug.device_id: mixer-07
  industrial.sparkplug.message_type: DDATA
  industrial.sparkplug.metric_name: BearingTemperature
  industrial.sparkplug.metric_type: Float
  industrial.sparkplug.is_historical: false

  # MQTT broker
  industrial.mqtt.broker_uri: mqtt://broker-01:1883
  industrial.mqtt.qos: 1

  # Data quality — mapped from Sparkplug B quality: true
  industrial.data.quality: good
  industrial.timestamp.source: gateway

  # Common context
  industrial.site: plant-01
  industrial.area: packaging
  industrial.asset.id: mixer-07
  industrial.asset.type: mixer
```

---

## Open questions

**Group ID and device ID mapping to common attributes**

The relationship between Sparkplug B topic components (`group_id`,
`edge_node_id`, `device_id`) and the ISA-95 hierarchy attributes defined
in [Common](../common/) (`industrial.site`, `industrial.area`,
`industrial.asset.id`) is deployment-specific. A convention for expressing
this mapping — perhaps as configuration metadata on the receiver — would
help make Sparkplug B telemetry consistently correlatable with telemetry
from other protocols.

**Richer quality model**

Sparkplug B's quality model is a single boolean — significantly simpler
than OPC-UA's StatusCode model. Whether additional quality detail can or
should be inferred from edge node behaviour (repeated identical values,
gaps in sequence numbers, delayed timestamps) is an open question.

**Dataset and Template metric types**

Sparkplug B supports complex metric types — Dataset (a table of values)
and Template (a structured object). How these should be represented in
OTel, which has no equivalent structured metric type, is not yet defined.

---

## Related

- [Data Quality](../data-quality/) — general quality conventions
- [Common attributes](../common/) — site, asset, and controller context
- [MQTT Sparkplug receiver configuration](../../collector/receivers/mqtt-sparkplug/) — how
  to configure the OTel Collector to collect from Sparkplug B brokers