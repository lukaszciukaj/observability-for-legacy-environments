---
title: Common Attributes
linkTitle: Common
description: >-
  Shared semantic convention attributes applicable across all industrial
  protocols and environments, covering site location, assets, and controllers.
status: experimental
weight: 10
---

> **Status: Experimental.**
> These attributes are not part of the official OpenTelemetry semantic
> conventions specification.

Common attributes provide the shared context that makes industrial telemetry
interpretable and correlatable across systems. They are not specific to any
single protocol and should be applied consistently wherever industrial telemetry
is collected.

## Why context attributes matter

Without shared context, a telemetry value is difficult to interpret:

```text
temperature = 72
```

With common attributes applied, the same value becomes actionable:

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
```

The sections below define the recommended attribute groups. All attributes are
optional unless marked required, but the more context provided, the more useful
the telemetry becomes for correlation, alerting, and analysis.

---

## Site and location attributes

These attributes describe where telemetry originates within the physical or
logical hierarchy of an industrial operation.

The hierarchy follows the ISA-95 / IEC 62264 standard model:
`site → area → line → cell → zone`.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.site` | string | `plant-01` | Manufacturing site, plant, or facility |
| `industrial.area` | string | `packaging` | Functional area within the site |
| `industrial.line` | string | `line-03` | Production line or process line |
| `industrial.cell` | string | `cell-a` | Work cell, automation cell, or machine group |
| `industrial.zone` | string | `zone-2` | Physical or logical zone within a cell or area |
| `industrial.location` | string | `north-wall-panel` | Optional human-readable location description |

### Usage notes

- `industrial.site` is the most important attribute for multi-site deployments.
  It should be set as a resource attribute on the OTel Collector rather than
  on each individual metric, to avoid high cardinality.
- `industrial.area` and `industrial.line` are the most commonly used hierarchy
  levels in practice. `industrial.cell` and `industrial.zone` are optional and
  may not apply in all environments.
- Use consistent, stable values. Renaming a site or area after telemetry has
  been collected breaks dashboards, alerts, and historical queries.

---

## Asset attributes

These attributes describe the machine, device, or system producing the
telemetry — the physical or logical asset being monitored.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.asset.id` | string | `mixer-07` | Unique asset identifier within the site |
| `industrial.asset.name` | string | `Main Mixer 7` | Human-readable asset name |
| `industrial.asset.type` | string | `mixer` | Asset category or machine type |
| `industrial.asset.vendor` | string | `example-vendor` | Equipment vendor or manufacturer |
| `industrial.asset.model` | string | `mx-2000` | Equipment model designation |
| `industrial.asset.serial_number` | string | `SN123456` | Equipment serial number |
| `industrial.asset.criticality` | string | `high` | Operational or business criticality |
| `industrial.asset.owner` | string | `operations` | Owning team or department |

### Recommended values for `industrial.asset.criticality`

| Value | Meaning |
|---|---|
| `low` | Loss of this asset has minimal operational impact |
| `medium` | Loss causes partial degradation or increased manual effort |
| `high` | Loss significantly impacts production or safety |
| `critical` | Loss causes immediate production stop or safety risk |

### Usage notes

- `industrial.asset.id` should be stable and unique within a site. It is
  the primary key for correlating telemetry from a single asset across
  protocols and collectors.
- `industrial.asset.serial_number` is useful for asset lifecycle management
  but may be considered sensitive in some environments. Review data handling
  policies before including it in telemetry that leaves the OT network.
- `industrial.asset.type` should use consistent values across the
  organisation. Consider maintaining a controlled vocabulary for asset types
  to avoid variations such as `cnc`, `cnc_machine`, and `CNC` all appearing
  in dashboards.

---

## Controller and device attributes

These attributes describe the PLC, controller, gateway, or edge device that
is the direct source of the telemetry — distinct from the asset it monitors.

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.controller.id` | string | `plc-packaging-01` | Controller identifier |
| `industrial.controller.type` | string | `plc` | Controller category |
| `industrial.controller.vendor` | string | `example-vendor` | Controller vendor |
| `industrial.controller.model` | string | `controller-5000` | Controller model |
| `industrial.controller.firmware.version` | string | `1.2.3` | Firmware version |
| `industrial.gateway.id` | string | `edge-gw-01` | Gateway or edge collector identifier |
| `industrial.gateway.type` | string | `protocol-bridge` | Gateway category |

### Recommended values for `industrial.controller.type`

| Value | Description |
|---|---|
| `plc` | Programmable Logic Controller |
| `rtu` | Remote Terminal Unit |
| `dcs` | Distributed Control System |
| `cnc` | Computer Numerical Control machine |
| `robot_controller` | Industrial robot controller |
| `edge_gateway` | Edge gateway or protocol bridge |
| `scada_server` | SCADA server acting as a data source |
| `historian` | Process historian |

### Usage notes

- Controller attributes are most useful as resource attributes, set once on
  the Collector or receiver rather than repeated on every metric.
- `industrial.controller.firmware.version` is valuable for troubleshooting
  and vulnerability tracking, but may change during maintenance windows.
  Consider whether firmware version should be a resource attribute (stable
  per session) or a metric attribute (changes over time).
- Where a gateway sits between the OTel Collector and the actual controller,
  both `industrial.gateway.*` and `industrial.controller.*` attributes may
  be relevant — the gateway is the network endpoint, the controller is the
  data source.

---

## Applying common attributes

Common attributes can be applied at several levels in an OTel pipeline:

**As resource attributes** — set on the Collector or receiver, applied to all
telemetry from that source. Most appropriate for attributes that are stable and
the same for all signals from a given device, such as `industrial.site`,
`industrial.controller.id`, and `industrial.asset.id`.

**As metric or log attributes** — set per signal, where the value varies across
signals from the same source. For example, a single PLC may report values from
multiple production lines.

**Using the Collector `resourcedetection` or `transform` processor** — to
enrich telemetry with site and asset context at the edge, before forwarding
to a central backend.

See [Collector configuration](../collector/configuration/) for examples of
applying these attributes in practice.