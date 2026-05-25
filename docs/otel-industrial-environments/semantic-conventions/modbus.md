---
title: Modbus Semantic Conventions
linkTitle: Modbus
description: >-
  Experimental semantic conventions for telemetry collected from Modbus
  devices, covering register types, addressing, and quality inference.
status: experimental
weight: 30
---

> **Status: Experimental.**
> These conventions are not part of the official OpenTelemetry semantic
> conventions specification.

## Overview

Modbus is one of the oldest and most widely deployed industrial protocols,
commonly found in PLCs, drives, sensors, and meters. It defines a simple
register-based data model with four register types, a polling-based
communication model, and no native support for data quality codes,
timestamps, or engineering units.

These conventions define how Modbus-specific concepts map to OpenTelemetry
attributes, and how quality context can be inferred and represented when
collecting Modbus telemetry through an OTel pipeline.

For general data quality conventions that apply across all protocols, see
[Data Quality](../data-quality/).
For site, asset, and controller attributes, see [Common](../common/).

---

## Modbus data model

Modbus organises data into four register types, each with its own address
space and access characteristics:

| Register type | Access | Data type | Typical use |
|---|---|---|---|
| Coil | Read / write | Boolean | Digital outputs — motor running, valve open |
| Discrete input | Read only | Boolean | Digital inputs — limit switch, sensor triggered |
| Holding register | Read / write | 16-bit word | Setpoints, configuration, counters |
| Input register | Read only | 16-bit word | Measured values — temperature, pressure, flow |

A Modbus device is identified on the network by its **unit ID** (also called
slave ID or device ID), a value between 1 and 247. On Modbus TCP, the unit ID
identifies the logical device within a gateway or multi-drop network.

---

## Proposed attributes

### Protocol attributes

These attributes describe the Modbus collection context and should be applied
to all telemetry collected from Modbus devices.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.modbus.transport` | string | `tcp` | Transport variant used for collection |
| `industrial.modbus.endpoint` | string | `192.168.1.10:502` | Host and port of the Modbus device or gateway |
| `industrial.modbus.unit_id` | int | `1` | Modbus unit ID (slave ID) of the target device |
| `industrial.modbus.register_type` | string | `holding_register` | Register type being read |
| `industrial.modbus.address` | int | `40021` | Register address within the device address space |
| `industrial.modbus.quantity` | int | `2` | Number of registers read in a single request |
| `industrial.modbus.function_code` | int | `3` | Modbus function code used for the read request |

### Recommended values for `industrial.modbus.transport`

| Value | Description |
|---|---|
| `tcp` | Modbus TCP over Ethernet |
| `rtu` | Modbus RTU over serial (RS-485, RS-232) |
| `ascii` | Modbus ASCII over serial |
| `rtu_over_tcp` | Modbus RTU framing tunnelled over TCP |

### Recommended values for `industrial.modbus.register_type`

| Value | Modbus register | Function codes |
|---|---|---|
| `coil` | Coil (0x) | FC01 read, FC05/15 write |
| `discrete_input` | Discrete input (1x) | FC02 read |
| `holding_register` | Holding register (4x) | FC03 read, FC06/16 write |
| `input_register` | Input register (3x) | FC04 read |

---

## Register type to OTel metric type mapping

Modbus register types map naturally to OTel metric types based on their
data characteristics:

| Register type | Recommended OTel metric type | Rationale |
|---|---|---|
| `coil` | Gauge (0 or 1) | Boolean state — not additive, not monotonic |
| `discrete_input` | Gauge (0 or 1) | Boolean state — not additive, not monotonic |
| `holding_register` | Gauge or Sum | Gauge for measured values; Sum for monotonic counters |
| `input_register` | Gauge | Measured process values are non-monotonic by nature |

### Multi-register values

Holding and input registers are 16-bit words. Many devices use two
consecutive registers to represent a 32-bit value (float or integer).
When a value spans multiple registers, the following attributes apply:

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.modbus.byte_order` | string | `big_endian` | Byte order used for multi-register values |
| `industrial.modbus.word_order` | string | `high_word_first` | Word order for 32-bit values spanning two registers |

Recommended values for `industrial.modbus.byte_order`: `big_endian`,
`little_endian`.

Recommended values for `industrial.modbus.word_order`: `high_word_first`,
`low_word_first`.

---

## Quality in Modbus

Modbus has no native quality code mechanism. A register read either succeeds
and returns a value, or fails with an exception or timeout. There is no
protocol-level way to indicate that a value is uncertain or stale.

Quality context for Modbus telemetry must be inferred by the collection
layer — the OTel Collector receiver, the edge gateway, or a Collector
processor — and mapped to the general quality attributes defined in
[Data Quality](../data-quality/).

### Mapping collection outcomes to quality states

| Collection outcome | `industrial.data.quality` | `industrial.data.quality.detail` |
|---|---|---|
| Read succeeded, value within configured range | `good` | — |
| Read succeeded, value outside configured range | `uncertain` | `out_of_range` |
| Exception response from device | `bad` | `modbus_exception` |
| No response within timeout | `bad` | `timeout` |
| Connection to device or gateway failed | `bad` | `connection_failure` |
| Last known value held after failed read | `uncertain` | `last_known_value` |
| Polled value unchanged beyond expected scan interval | `stale` | — |

### Modbus exception codes

When a Modbus device responds with an exception, the exception code provides
additional detail. The exception code should be recorded in
`industrial.data.quality.detail` using the following values:

| Modbus exception code | Value for `industrial.data.quality.detail` |
|---|---|
| 01 — Illegal function | `modbus_exception_01` |
| 02 — Illegal data address | `modbus_exception_02` |
| 03 — Illegal data value | `modbus_exception_03` |
| 04 — Server device failure | `modbus_exception_04` |
| 05 — Acknowledge | `modbus_exception_05` |
| 06 — Server device busy | `modbus_exception_06` |

---

## Usage notes

- `industrial.modbus.unit_id` and `industrial.modbus.address` together
  uniquely identify a register within a device. Both should be included
  when the telemetry may be consumed by systems that need to trace a value
  back to its source.

- `industrial.modbus.endpoint` should be treated as a resource attribute
  set once on the receiver, not repeated on every metric. It identifies
  the collection endpoint, not the individual register.

- `industrial.modbus.function_code` is useful for diagnostics and auditing
  but adds cardinality when used as a metric attribute. Consider whether it
  is needed in your deployment before including it on every metric.

- Modbus register addresses vary by manufacturer convention. Some vendors
  use 1-based addressing (register 1 = address 0 on the wire), others use
  0-based. The `industrial.modbus.address` attribute should record the
  address as configured in the Collector or gateway, with documentation
  noting which convention is in use.

---

## Example

A temperature reading from holding register 40021 on a Modbus TCP device,
with full context attributes:

```yaml
metric: industrial.temperature
value: 72.4
unit: fahrenheit
attributes:
  # Modbus-specific
  industrial.modbus.transport: tcp
  industrial.modbus.endpoint: 192.168.1.10:502
  industrial.modbus.unit_id: 1
  industrial.modbus.register_type: holding_register
  industrial.modbus.address: 40021

  # Data quality
  industrial.data.quality: good
  industrial.timestamp.source: collector
  industrial.scan.interval_ms: 1000

  # Common context
  industrial.site: plant-01
  industrial.area: packaging
  industrial.asset.id: mixer-07
  industrial.asset.type: mixer
```

---

## Related

- [Data Quality](../data-quality/) — general quality conventions
- [Common attributes](../common/) — site, asset, and controller context
- [Modbus receiver configuration](../../collector/receivers/modbus/) — how
  to configure the OTel Collector to collect from Modbus devices