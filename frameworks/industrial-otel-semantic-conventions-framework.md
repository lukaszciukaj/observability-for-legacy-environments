# Industrial OpenTelemetry Semantic Conventions Framework (Experimental)

## Overview

Industrial environments generate large volumes of operational telemetry, but that telemetry is often difficult to interpret outside of its original system context.

This document proposes an experimental industrial semantic conventions framework inspired by the OpenTelemetry semantic conventions model. It is not an official OpenTelemetry semantic convention and should be treated as a community-driven reference and architectural exploration. The framework may help inform future discussions or proposals around industrial and operational technology (OT) telemetry modeling within the broader OpenTelemetry ecosystem.

OpenTelemetry semantic conventions provide standardized naming and attribute patterns for telemetry data across applications, infrastructure, cloud services, databases, messaging systems, and other domains:

- https://opentelemetry.io/docs/specs/semconv/

A Modbus register, OPC UA node, PLC tag, historian value, or machine alarm may contain valuable operational information, but without consistent metadata it is difficult to correlate, analyze, or reuse across observability, analytics, security, and reliability platforms.

This framework proposes a practical semantic model for describing industrial and legacy telemetry in OpenTelemetry-oriented environments. The goal is not to replace existing OpenTelemetry semantic conventions, but to provide a structured starting point for modeling industrial and legacy system telemetry in a consistent, vendor-neutral way.

---

## Goals

- Provide consistent attributes for industrial telemetry
- Improve correlation across machines, production lines, sites, and protocols
- Support metrics, logs, events, and traces from OT environments
- Enable clearer dashboards, alerts, and analytics
- Reduce vendor-specific naming and modeling differences
- Help bridge industrial protocols with OpenTelemetry pipelines

---

## Why Semantic Conventions Matter in Industrial Environments

Industrial telemetry often comes from heterogeneous systems such as:

- PLCs
- SCADA systems
- HMIs
- historians
- edge gateways
- sensors
- drives
- robots
- packaging machines
- CNC machines
- building automation systems
- legacy applications

Without shared metadata, two values such as `temperature`, `pressure`, or `status` may be impossible to interpret correctly.

For example:

```text
temperature = 72
```

is not very useful without context.

A better telemetry model would include:

```yaml
temperature = 72

attributes:
  industrial.site: "plant-01"
  industrial.area: "packaging"
  industrial.line: "line-03"
  industrial.asset.id: "mixer-07"
  industrial.asset.type: "mixer"
  industrial.signal.name: "bearing_temperature"
  industrial.signal.unit: "fahrenheit"
  industrial.protocol.name: "modbus"
  industrial.protocol.address: "40021"
```

This makes the signal easier to understand, search, alert on, and correlate with other operational data.

---

## Proposed Attribute Groups

### Site and Location Attributes

These attributes describe where telemetry originates.

| Attribute | Example | Description |
|---|---|---|
| `industrial.site` | `plant-01` | Manufacturing site, plant, or facility |
| `industrial.area` | `packaging` | Functional area within the site |
| `industrial.line` | `line-03` | Production line or process line |
| `industrial.cell` | `cell-a` | Work cell, automation cell, or machine group |
| `industrial.zone` | `zone-2` | Physical or logical zone |
| `industrial.location` | `north-wall-panel` | Optional human-readable location |

---

### Asset Attributes

These attributes describe the machine, device, or system producing telemetry.

| Attribute | Example | Description |
|---|---|---|
| `industrial.asset.id` | `mixer-07` | Unique asset identifier |
| `industrial.asset.name` | `Main Mixer 7` | Human-readable asset name |
| `industrial.asset.type` | `mixer` | Asset category or machine type |
| `industrial.asset.vendor` | `example-vendor` | Equipment vendor |
| `industrial.asset.model` | `mx-2000` | Equipment model |
| `industrial.asset.serial_number` | `SN123456` | Serial number, if appropriate |
| `industrial.asset.criticality` | `high` | Business or operational criticality |
| `industrial.asset.owner` | `operations` | Owning team or department |

Recommended values for `industrial.asset.criticality`:

```text
low
medium
high
critical
```

---

### Controller and Device Attributes

These attributes describe PLCs, controllers, gateways, or edge devices.

| Attribute | Example | Description |
|---|---|---|
| `industrial.controller.id` | `plc-packaging-01` | Controller identifier |
| `industrial.controller.type` | `plc` | Controller type |
| `industrial.controller.vendor` | `example-vendor` | Controller vendor |
| `industrial.controller.model` | `controller-5000` | Controller model |
| `industrial.controller.firmware.version` | `1.2.3` | Firmware version |
| `industrial.gateway.id` | `edge-gw-01` | Gateway or edge collector identifier |
| `industrial.gateway.type` | `protocol-bridge` | Gateway type |

Recommended values for `industrial.controller.type`:

```text
plc
rtu
dcs
cnc
robot_controller
edge_gateway
scada_server
historian
```

---

### Protocol Attributes

These attributes describe how telemetry was collected.

| Attribute | Example | Description |
|---|---|---|
| `industrial.protocol.name` | `modbus` | Industrial protocol name |
| `industrial.protocol.transport` | `tcp` | Transport mechanism |
| `industrial.protocol.endpoint` | `192.168.1.10:502` | Endpoint used for collection |
| `industrial.protocol.unit_id` | `1` | Modbus unit/slave ID, when applicable |
| `industrial.protocol.address` | `40021` | Protocol-specific address |
| `industrial.protocol.register_type` | `holding_register` | Register or data source type |
| `industrial.protocol.node_id` | `ns=2;s=Temperature` | OPC UA node ID, when applicable |
| `industrial.protocol.browse_name` | `Temperature` | OPC UA browse name, when applicable |

Recommended values for `industrial.protocol.name`:

```text
modbus
opcua
mqtt
ethernet_ip
profinet
mtconnect
bacnet
serial
custom
```

---

## Disclaimer

This framework is an experimental reference model intended for educational, architectural, and community discussion purposes. It is not an official OpenTelemetry semantic convention.

Industrial environments are safety-sensitive and operationally critical. Any telemetry integration, protocol polling, instrumentation, or data modeling approach should be carefully reviewed, tested, and validated before production use.