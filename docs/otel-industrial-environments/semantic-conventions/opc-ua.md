---
title: OPC-UA Semantic Conventions
linkTitle: OPC-UA
description: >-
  Experimental semantic conventions for telemetry collected from OPC UA
  servers, covering node addressing, namespaces, subscriptions, and the
  OPC UA quality code model.
status: experimental
weight: 40
---

> **Status: Experimental.**
> These conventions are not part of the official OpenTelemetry semantic
> conventions specification.

## Overview

OPC Unified Architecture (OPC-UA) is the primary modern standard for
interoperable industrial communication. Unlike older protocols, OPC-UA was
designed with a rich information model: every data point is a typed node
in an address space, carries a quality code and timestamp, and can be
accessed via subscription rather than polling.

These conventions define how OPC-UA concepts — nodes, namespaces, quality
codes, and subscriptions — map to OpenTelemetry attributes.

For general data quality conventions that apply across all protocols, see
[Data Quality](../data-quality/).
For site, asset, and controller attributes, see [Common](../common/).

---

## OPC-UA data model

### Address space and nodes

Every data point in OPC-UA is a **node** in a hierarchical address space.
Nodes are identified by a **NodeId**, which consists of:

- A **namespace index** — an integer that identifies the namespace, which
  may be vendor-defined or standardised
- An **identifier** — a string, integer, GUID, or byte string that uniquely
  identifies the node within its namespace

A NodeId is typically written as `ns=<index>;<type>=<identifier>`, for example:

```
ns=2;s=Plant01.Line03.Mixer07.BearingTemperature
ns=2;i=1234
```

Nodes that hold process values are **Variable nodes**. They expose a
**Value** attribute containing the current reading, a **StatusCode**
containing the quality, and two timestamps — **SourceTimestamp** (set
by the device) and **ServerTimestamp** (set by the OPC-UA server).

### Namespaces

Namespace index 0 is reserved for the OPC-UA standard itself. Namespace
index 1 is typically used by the server for its own nodes. Vendor and
application-specific nodes use higher namespace indices.

Namespace indices are local to a server and may change when a server is
reconfigured. The **namespace URI** is the stable identifier for a
namespace and should be used in preference to the index where possible.

### Subscriptions and monitored items

OPC-UA supports event-driven data collection through **subscriptions**.
A client creates a subscription and adds **monitored items** — individual
nodes — to it. The server publishes new values when they change, subject
to a configured deadband and sampling interval.

This is fundamentally different from polling: the server decides when to
send a new value, not the collector. This has direct implications for how
OTel timestamps and scan interval attributes should be interpreted for
OPC-UA telemetry.

---

## Proposed attributes

### Node attributes

These attributes identify the OPC-UA node that is the source of a
telemetry value.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.opcua.node_id` | string | `ns=2;s=Temperature` | Full NodeId of the source node |
| `industrial.opcua.node_namespace_index` | int | `2` | Namespace index component of the NodeId |
| `industrial.opcua.node_namespace_uri` | string | `urn:plant01:isa95` | Namespace URI — preferred over index for stability |
| `industrial.opcua.node_identifier` | string | `Plant01.Line03.Mixer07.BearingTemperature` | Identifier component of the NodeId |
| `industrial.opcua.node_identifier_type` | string | `string` | Identifier type: `string`, `numeric`, `guid`, `opaque` |
| `industrial.opcua.browse_name` | string | `BearingTemperature` | Human-readable browse name of the node |
| `industrial.opcua.display_name` | string | `Bearing Temperature` | Display name of the node as configured on the server |

### Server attributes

These attributes identify the OPC-UA server that is the source of the
telemetry. They are most appropriate as resource attributes set once on
the receiver.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.opcua.server_uri` | string | `opc.tcp://plc-01:4840` | Endpoint URL of the OPC-UA server |
| `industrial.opcua.server_application_uri` | string | `urn:plant01:plc01` | Application URI of the server, from its certificate |
| `industrial.opcua.session_id` | string | `session-abc123` | OPC-UA session identifier |

### Subscription attributes

These attributes describe the subscription and monitored item configuration
that produced the telemetry value.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.opcua.subscription_id` | int | `42` | OPC-UA subscription identifier |
| `industrial.opcua.sampling_interval_ms` | double | `500.0` | Configured sampling interval of the monitored item in milliseconds |
| `industrial.opcua.publishing_interval_ms` | double | `1000.0` | Configured publishing interval of the subscription in milliseconds |
| `industrial.opcua.queue_size` | int | `10` | Configured queue size of the monitored item |
| `industrial.opcua.discard_oldest` | boolean | `true` | Whether oldest values are discarded when the queue is full |

---

## OPC-UA quality codes

OPC-UA defines a 32-bit **StatusCode** that accompanies every value. The
StatusCode encodes a severity level, a sub-code, and additional flags.

### Severity mapping

The top two bits of the StatusCode encode the severity:

| OPC-UA severity | Bit pattern | `industrial.data.quality` |
|---|---|---|
| Good | `00` | `good` |
| Uncertain | `01` | `uncertain` |
| Bad | `10` or `11` | `bad` |

### Common StatusCode to quality detail mapping

The following table maps common OPC-UA StatusCodes to recommended values
for `industrial.data.quality.detail`. This list is not exhaustive — full
StatusCode definitions are in the OPC-UA specification Part 8.

**Good substatus codes**

| OPC-UA StatusCode | `industrial.data.quality.detail` | Description |
|---|---|---|
| `Good` | — | Normal good quality, no detail needed |
| `GoodLocalOverride` | `local_override` | Value has been overridden locally |
| `GoodEntryInserted` | `entry_inserted` | Value was inserted into a historical data set |
| `GoodEntryReplaced` | `entry_replaced` | Value replaced an existing historical entry |

**Uncertain substatus codes**

| OPC-UA StatusCode | `industrial.data.quality.detail` | Description |
|---|---|---|
| `UncertainNoCommunicationLastUsableValue` | `last_usable_value` | Communication lost; last good value is being held |
| `UncertainLastUsableValue` | `last_usable_value` | Value may not reflect current state |
| `UncertainSubstituteValue` | `substitute_value` | A substitute value is being provided |
| `UncertainInitialValue` | `initial_value` | Value has not yet been updated since startup |
| `UncertainSensorNotAccurate` | `sensor_not_accurate` | Sensor is operating outside its calibrated range |
| `UncertainEngineeringUnitsExceeded` | `engineering_units_exceeded` | Value is outside configured engineering unit limits |
| `UncertainSubNormal` | `sub_normal` | Value derived from fewer inputs than configured |

**Bad substatus codes**

| OPC-UA StatusCode | `industrial.data.quality.detail` | Description |
|---|---|---|
| `BadNoCommunication` | `no_communication` | No communication with the data source |
| `BadWaitingForInitialData` | `waiting_for_initial_data` | Server is waiting for first value since startup |
| `BadNodeIdUnknown` | `node_id_unknown` | NodeId is not recognised by the server |
| `BadNodeIdInvalid` | `node_id_invalid` | NodeId syntax is invalid |
| `BadAttributeIdInvalid` | `attribute_id_invalid` | Requested attribute does not exist on this node |
| `BadOutOfRange` | `out_of_range` | Value is outside the range defined for this node |
| `BadDataUnavailable` | `data_unavailable` | Data is not available for this node |
| `BadNotReadable` | `not_readable` | Node does not support read |
| `BadDeviceFailure` | `device_failure` | Source device has reported a failure |
| `BadSensorFailure` | `sensor_failure` | Sensor has reported a failure |
| `BadOutOfService` | `out_of_service` | Source is not in service |
| `BadDeadbandFilterInvalid` | `deadband_filter_invalid` | Deadband filter configuration is invalid |

### Timestamps

OPC-UA provides two timestamps with every value:

| OPC-UA timestamp | `industrial.timestamp.source` | Description |
|---|---|---|
| `SourceTimestamp` | `device` | Set by the data source — the PLC or sensor |
| `ServerTimestamp` | `gateway` | Set by the OPC-UA server when it recorded the value |

When both timestamps are available, `SourceTimestamp` is preferred as it
reflects when the physical measurement was taken. `ServerTimestamp` should
be used when the source does not provide a timestamp, or when source clock
reliability is in doubt.

---

## Usage notes

- `industrial.opcua.node_id` is the most important attribute for traceability.
  It uniquely identifies the data point on the server and allows a value to
  be traced back to its origin. It should always be included.

- `industrial.opcua.node_namespace_uri` is preferred over
  `industrial.opcua.node_namespace_index` for long-term stability. Namespace
  indices can change when a server is reconfigured; the URI does not.

- `industrial.opcua.browse_name` and `industrial.opcua.display_name` are
  useful for human readability in dashboards but add cardinality. Consider
  setting them as resource attributes or including them only where the
  consumer cannot resolve them from the NodeId.

- Subscription attributes — `sampling_interval_ms`, `publishing_interval_ms`,
  `queue_size` — are configuration metadata. They are most useful as resource
  attributes on the subscription rather than repeated on every metric.

- OPC-UA servers may return `Good` quality with a value that is nonetheless
  outside the expected engineering range. Consider configuring range checks
  in the Collector processor and mapping out-of-range values to `uncertain`
  with `industrial.data.quality.detail: engineering_units_exceeded`.

---

## Example

A bearing temperature reading from an OPC-UA server, with full context
attributes:

```yaml
metric: industrial.temperature
value: 72.4
unit: fahrenheit
attributes:
  # OPC-UA node
  industrial.opcua.server_uri: opc.tcp://plc-packaging-01:4840
  industrial.opcua.node_id: ns=2;s=Plant01.Line03.Mixer07.BearingTemperature
  industrial.opcua.node_namespace_uri: urn:plant01:isa95
  industrial.opcua.browse_name: BearingTemperature
  industrial.opcua.sampling_interval_ms: 500.0

  # Data quality — mapped from OPC-UA StatusCode Good
  industrial.data.quality: good
  industrial.timestamp.source: device

  # Common context
  industrial.site: plant-01
  industrial.area: packaging
  industrial.line: line-03
  industrial.asset.id: mixer-07
  industrial.asset.type: mixer
```

---

## Open questions

**NodeId cardinality**

OPC-UA address spaces can contain thousands or millions of nodes. Using
`industrial.opcua.node_id` as a metric attribute creates one time series
per node, which may exceed cardinality limits in some observability
backends. Strategies for managing this — aggregation, filtering, or
recording NodeId only in logs — need further discussion.

**Security mode and certificate attributes**

OPC-UA connections can operate in different security modes (None, Sign,
SignAndEncrypt) with different certificate configurations. Whether security
mode should be a telemetry attribute or left to infrastructure-level
metadata is an open question.

---

## Related

- [Data Quality](../data-quality/) — general quality conventions
- [Common attributes](../common/) — site, asset, and controller context
- [OPC-UA receiver configuration](../../collector/receivers/opc-ua/) — how
  to configure the OTel Collector to collect from OPC-UA servers