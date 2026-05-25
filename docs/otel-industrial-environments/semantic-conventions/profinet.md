---
title: PROFINET Semantic Conventions
linkTitle: PROFINET
description: >-
  Experimental semantic conventions for telemetry collected from PROFINET
  devices, covering device identity, slot and subslot addressing, alarm
  handling, and diagnostics.
status: experimental
weight: 60
---

> **Status: Experimental.**
> These conventions are not part of the official OpenTelemetry semantic
> conventions specification. PROFINET support in OpenTelemetry tooling is
> limited — this page documents proposed conventions ahead of broader
> collector implementation.

## Overview

PROFINET is an industrial Ethernet standard developed by PROFIBUS &
PROFINET International (PI) and widely deployed in manufacturing automation,
particularly in European industrial environments. It is the successor to
PROFIBUS and is dominant in sectors such as automotive, packaging,
food and beverage, and discrete manufacturing.

PROFINET operates in real-time on standard Ethernet hardware and defines
three communication classes:

| Class | Cycle time | Use case |
|---|---|---|
| TCP/IP (NRT) | > 100ms | Configuration, parameterisation, non-time-critical data |
| RT (Real-Time) | 1–10ms | Process data exchange — I/O data between controller and device |
| IRT (Isochronous Real-Time) | < 1ms | Motion control, synchronised drives |

For OTel telemetry collection, the primary focus is on RT process data
and diagnostic information available through the PROFINET device model.
IRT cycle data is typically too time-critical for OTel-based collection
and is better handled by dedicated motion control infrastructure.

For general data quality conventions that apply across all protocols, see
[Data Quality](../data-quality/).
For site, asset, and controller attributes, see [Common](../common/).

---

## PROFINET device model

### Device identity

Every PROFINET device is identified by:

- **Device name** — a DNS-like name assigned during commissioning, for
  example `mixer-07.line-03.plant-01`. Unique within the PROFINET network.
- **IP address** — assigned during commissioning, typically by the
  controller using DCP (Discovery and Configuration Protocol).
- **Vendor ID and Device ID** — a pair of 16-bit integers that identify
  the device type. The combination maps to a specific GSD (General Station
  Description) file that describes the device's capabilities.
- **MAC address** — the hardware address of the Ethernet interface.

### Slot and subslot model

PROFINET organises a device's I/O data into a hierarchical structure of
**slots** and **subslots**:

- A **slot** represents a physical or logical module within the device —
  for example, a I/O module in a rack
- A **subslot** represents a channel or function within a slot — for
  example, an individual digital input or analogue output channel

Process data is addressed by slot and subslot number. Slot 0 is reserved
for the device itself and carries device-level diagnostics.

### AR and CR

A PROFINET **Application Relation (AR)** is the logical connection between
a controller and a device. Within an AR, **Communication Relations (CRs)**
define the specific data exchanges — typically one cyclic data CR for
process I/O and one acyclic CR for alarms and parameterisation.

AR state is significant for telemetry quality: if an AR is not established
or has been interrupted, process data from that device is no longer valid.

### Alarms and diagnostics

PROFINET defines a structured alarm model for reporting device and channel
faults. Alarms are categorised by type and priority:

| Alarm type | Description |
|---|---|
| Diagnosis alarm | Device or channel has entered a fault state |
| Pull alarm | Module has been removed from a slot |
| Plug alarm | Module has been inserted into a slot |
| Process alarm | Application-defined alarm from the device |
| Status alarm | Device status has changed |
| Return of submodule alarm | A previously faulted submodule has recovered |

Alarms are the primary mechanism for reporting quality degradation in
PROFINET and map naturally to OTel log events.

---

## Proposed attributes

### Device identity attributes

These attributes identify the PROFINET device that is the source of
telemetry. They are most appropriate as resource attributes.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.profinet.device_name` | string | `mixer-07.line-03` | PROFINET device name assigned during commissioning |
| `industrial.profinet.device_ip` | string | `192.168.1.42` | IP address of the PROFINET device |
| `industrial.profinet.device_mac` | string | `00:1B:44:11:3A:B7` | MAC address of the PROFINET device |
| `industrial.profinet.vendor_id` | string | `0x002A` | Vendor ID from the device identity (hexadecimal) |
| `industrial.profinet.device_id` | string | `0x0108` | Device ID from the device identity (hexadecimal) |
| `industrial.profinet.station_type` | string | `ET 200SP` | Station type as reported by the device |
| `industrial.profinet.firmware_version` | string | `V3.1.2` | Firmware version of the device |

### Slot and subslot attributes

These attributes identify the specific data point within the PROFINET
device hierarchy.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.profinet.slot_number` | int | `2` | Slot number within the device |
| `industrial.profinet.subslot_number` | int | `1` | Subslot number within the slot |
| `industrial.profinet.channel_number` | int | `3` | Channel number within the subslot, where applicable |
| `industrial.profinet.submodule_ident` | string | `0x00000001` | Submodule identity number (hexadecimal) |

### Controller and AR attributes

These attributes identify the PROFINET controller and the application
relation under which the data was collected.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.profinet.controller_name` | string | `plc-packaging-01` | PROFINET name of the IO controller |
| `industrial.profinet.controller_ip` | string | `192.168.1.1` | IP address of the IO controller |
| `industrial.profinet.ar_uuid` | string | `dea0-1234-...` | UUID of the Application Relation |
| `industrial.profinet.ar_type` | string | `io_controller` | AR type |

### Alarm attributes

These attributes describe PROFINET alarm events and should be applied to
OTel log events generated from PROFINET alarms.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.profinet.alarm_type` | string | `diagnosis` | PROFINET alarm type — see values below |
| `industrial.profinet.alarm_specifier` | string | `0x0000` | Alarm specifier from the alarm PDU (hexadecimal) |
| `industrial.profinet.maintenance_required` | boolean | `false` | Maintenance required bit set in diagnostic alarm |
| `industrial.profinet.maintenance_demanded` | boolean | `true` | Maintenance demanded bit set in diagnostic alarm |
| `industrial.profinet.fault_code` | string | `0x0820` | Manufacturer-specific fault code (hexadecimal) |

### Recommended values for `industrial.profinet.alarm_type`

| Value | PROFINET alarm type |
|---|---|
| `diagnosis` | Diagnosis alarm |
| `pull` | Pull alarm — module removed |
| `plug` | Plug alarm — module inserted |
| `process` | Process alarm |
| `status` | Status alarm |
| `return_of_submodule` | Return of submodule alarm |
| `released` | Released alarm |

---

## Quality in PROFINET

PROFINET process data does not carry per-value quality codes in the same
way OPC-UA does. Quality in PROFINET is expressed at the AR and device
level rather than the individual data point level.

### AR state mapping

The state of the Application Relation is the primary quality indicator
for PROFINET process data:

| AR state | `industrial.data.quality` | `industrial.data.quality.detail` |
|---|---|---|
| AR established, cyclic data running | `good` | — |
| AR in startup / not yet established | `uncertain` | `ar_startup` |
| AR interrupted — controller retrying | `uncertain` | `ar_interrupted` |
| AR aborted — device unreachable | `bad` | `ar_aborted` |
| Device pulled from slot | `bad` | `module_pulled` |
| Wrong module plugged in slot | `bad` | `wrong_module` |

### Diagnostic alarm mapping

PROFINET diagnostic alarms indicate fault conditions on a specific
channel or submodule and should generate both a log event and a quality
state transition on affected metrics:

| Diagnostic condition | `industrial.data.quality` | `industrial.data.quality.detail` |
|---|---|---|
| Maintenance required | `uncertain` | `maintenance_required` |
| Maintenance demanded | `uncertain` | `maintenance_demanded` |
| Channel fault active | `bad` | `channel_fault` |
| Submodule fault active | `bad` | `submodule_fault` |
| Return of submodule | `good` | — |

### Timestamps

PROFINET does not define a standard mechanism for attaching source
timestamps to individual process data values. Timestamps on PROFINET
telemetry collected by an OTel Collector are typically applied at
collection time:

| `industrial.timestamp.source` | When to use |
|---|---|
| `gateway` | Timestamp applied by a PROFINET gateway or protocol bridge |
| `collector` | Timestamp applied by the OTel Collector at collection time |

`device` timestamps are not available for standard PROFINET RT process
data. Some devices expose timestamped data through acyclic read services
or manufacturer-specific extensions — in those cases `device` is
appropriate.

---

## Usage notes

- `industrial.profinet.device_name` is the most stable identifier for a
  PROFINET device. Unlike IP addresses, device names persist across
  network reconfigurations and should be the primary key for correlating
  telemetry from a single device.

- `industrial.profinet.vendor_id` and `industrial.profinet.device_id`
  together identify the device type and correspond to a specific GSD file.
  They are useful for asset inventory and vulnerability tracking but add
  cardinality — consider setting them as resource attributes rather than
  metric attributes.

- PROFINET process data is typically collected indirectly — via the
  controller's data memory, a PROFINET gateway, or a monitoring tap —
  rather than directly from the device. The collection method affects
  which attributes are available and the reliability of the data.

- `industrial.profinet.alarm_type` values should be recorded on OTel
  log events rather than metrics. Alarms are discrete events, not
  continuous measurements, and are better represented in the log signal.

- The `industrial.profinet.maintenance_required` and
  `industrial.profinet.maintenance_demanded` attributes reflect the
  PROFINET maintenance concept, which distinguishes between conditions
  that will eventually cause failure (required) and conditions that
  require immediate attention (demanded). Both map to `uncertain` quality
  but carry different operational urgency.

---

## Example

A temperature reading collected from a PROFINET device via a protocol
gateway, with a concurrent diagnostic alarm:

```yaml
# Process data metric
metric: industrial.temperature
value: 84.2
unit: celsius
attributes:
  # PROFINET device
  industrial.profinet.device_name: mixer-07.line-03.plant-01
  industrial.profinet.device_ip: 192.168.1.42
  industrial.profinet.vendor_id: "0x002A"
  industrial.profinet.device_id: "0x0108"
  industrial.profinet.slot_number: 2
  industrial.profinet.subslot_number: 1

  # Data quality — maintenance demanded alarm active on this submodule
  industrial.data.quality: uncertain
  industrial.data.quality.detail: maintenance_demanded
  industrial.timestamp.source: gateway

  # Common context
  industrial.site: plant-01
  industrial.area: packaging
  industrial.line: line-03
  industrial.asset.id: mixer-07
  industrial.asset.type: mixer

---

# Corresponding alarm log event
log:
  body: "PROFINET diagnosis alarm: maintenance demanded on mixer-07 slot 2"
  attributes:
    industrial.profinet.device_name: mixer-07.line-03.plant-01
    industrial.profinet.alarm_type: diagnosis
    industrial.profinet.slot_number: 2
    industrial.profinet.subslot_number: 1
    industrial.profinet.maintenance_demanded: true
    industrial.profinet.fault_code: "0x0820"
    industrial.data.quality: uncertain
    industrial.data.quality.detail: maintenance_demanded
```

---

## Open questions

**Process data collection mechanism**

PROFINET RT process data is exchanged directly between the IO controller
and IO devices at the Ethernet frame level. There is no standard read
interface for a third-party collector to access this data without being
a participant in the PROFINET network or using dedicated monitoring
hardware. The practical collection mechanisms — controller memory read,
OPC-UA gateway, PROFINET tap — need to be documented with their
respective limitations and attribute implications.

**IRT and time-sensitive data**

Isochronous Real-Time (IRT) data operates at sub-millisecond cycle times
and is fundamentally incompatible with OTel's current data model and
collection mechanisms. The boundary between what is and is not appropriate
for OTel collection in PROFINET environments needs explicit definition.

**GSD file integration**

PROFINET GSD (General Station Description) files contain rich metadata
about device capabilities, slot configurations, and channel definitions.
Whether and how GSD file data should be integrated into OTel resource
attributes — for example to automatically populate slot and subslot
descriptions — is an open question.

---

## Related

- [Data Quality](../data-quality/) — general quality conventions
- [Common attributes](../common/) — site, asset, and controller context
- [PROFINET receiver configuration](../../collector/receivers/profinet/) — how
  to configure the OTel Collector to collect from PROFINET networks