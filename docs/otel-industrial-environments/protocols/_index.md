---
title: Protocols
linkTitle: Protocols
description: >-
  Per-protocol integration guides for collecting OpenTelemetry telemetry
  from industrial protocols, including OPC-UA, Modbus, MQTT Sparkplug B,
  and PROFINET.
status: experimental
weight: 40
---

Industrial environments communicate over a diverse set of protocols, many
of which were designed decades before observability was a concern. This
section provides per-protocol guidance for integrating those protocols
with OpenTelemetry — covering the protocol data model, signal mapping,
Collector receiver configuration, and known limitations.

Each protocol section follows a consistent structure:

- **Overview** — what the protocol is, where it is used, and what makes
  it distinct from an observability perspective
- **Data model** — how the protocol organises data and how that maps to
  OTel signals
- **Getting started** — minimal working configuration to start collecting
  telemetry
- **Receiver configuration** — full Collector receiver reference
- **Troubleshooting** — common failure modes and how to address them

Semantic conventions for each protocol are defined separately in
[Semantic Conventions](../semantic-conventions/). Protocol pages reference
those conventions but do not redefine them.

---

## Protocol overview

### OPC-UA

OPC Unified Architecture is the primary modern standard for interoperable
industrial communication. Developed by the OPC Foundation, it provides a
platform-independent, service-oriented architecture with a rich information
model, built-in security, and support for both client-server and
publish-subscribe communication patterns.

OPC-UA was designed with observability in mind. Every data point is a typed
node in a structured address space, carries a quality code and timestamp,
and can be accessed via event-driven subscriptions rather than polling.
It is the protocol most naturally aligned with OTel concepts and the
recommended starting point for new industrial OTel integrations.

OPC-UA is dominant in process industries, packaging, building automation,
and as a northbound interface on SCADA and historian systems.

### Modbus

Modbus is one of the oldest and most widely deployed industrial protocols,
originally developed by Modicon in 1979. Its simplicity — a straightforward
register-based data model, a small set of function codes, and no
authentication or encryption — made it ubiquitous across PLCs, drives,
sensors, meters, and virtually every other category of industrial device.

Modbus has no native support for data quality codes, timestamps, or
engineering units. These must be inferred or configured at the collection
layer. Despite its age and limitations, Modbus remains the most common
protocol in brownfield industrial environments and will remain relevant
for decades.

### MQTT Sparkplug B

MQTT is a lightweight publish-subscribe messaging protocol designed for
constrained devices and unreliable networks. On its own, MQTT imposes no
structure on topics or payloads. Sparkplug B is a specification built on
top of MQTT that adds a standardised topic namespace, a Protocol Buffers
payload format, and a session management model with birth and death
certificates.

Sparkplug B was designed specifically for industrial IIoT applications
and is increasingly deployed as the northbound communication layer from
edge nodes to SCADA and cloud platforms. Its event-driven model and
structured payload make it well-suited to OTel integration, though the
mapping from Sparkplug B's topic hierarchy to OTel resource attributes
requires care.

### PROFINET

PROFINET is an industrial Ethernet standard developed by PROFIBUS &
PROFINET International (PI) and the successor to PROFIBUS. It is dominant
in European manufacturing automation, particularly in automotive,
packaging, and discrete manufacturing sectors.

PROFINET operates in real-time on standard Ethernet hardware and defines
three communication classes — from standard TCP/IP for configuration
through to isochronous real-time for motion control. Collecting PROFINET
telemetry with OTel is more complex than other protocols: RT process data
is exchanged directly at the Ethernet frame level with no standard
read interface for third-party collectors. In practice, collection happens
via the controller's data memory, an OPC-UA gateway, or dedicated
monitoring hardware.

---

## Capability matrix

> **Note:** All receivers listed below are community projects, not official
> OpenTelemetry Collector contrib components. Links point to the source
> repositories. Verify current status before production use.

| Protocol | Receiver | Status | OTel signals | Client libraries | Manual instrumentation | Auto-instrumentation | Docs |
|---|---|---|---|---|---|---|---|
| [OPC-UA](./opc-ua/) | [opcua-receiver](https://github.com/bruegth/opentelemetry-collector-opcua-receiver) | Alpha | Logs only ¹ | [gopcua](https://github.com/gopcua/opcua), [node-opcua](https://node-opcua.github.io/), [opcua-asyncio](https://github.com/FreeOpcUa/opcua-asyncio) | Possible | Not available | In progress |
| [Modbus](./modbus/) | [modbusreceiver](https://github.com/lukaszciukaj/modbusreceiver) | Development | Metrics | [pymodbus](https://pymodbus.readthedocs.io/), [libmodbus](https://libmodbus.org/), [modbus4j](https://github.com/MangoAutomation/modbus4j) | Possible | Not available | In progress |
| [MQTT Sparkplug B](./mqtt-sparkplug/) | [sparkplugreceiver](https://github.com/jmacd/caspar.water/tree/main/sparkplug/sparkplugreceiver) | Experimental | Metrics | [Eclipse Tahu](https://github.com/eclipse/tahu), [tahu Python](https://github.com/eclipse/tahu/tree/master/python) | Possible | Not available | In progress |
| [PROFINET](./profinet/) | None available | — | — | Limited options ² | Limited | Not available | Placeholder |

### Notes

**¹ OPC-UA receiver — logs only**
The current community receiver implements the OPC-UA Part 26 LogObject
specification and collects log records from OPC-UA servers. It does not
collect general process data (Variable node values) as metrics. Collection
of OPC-UA Variable nodes as OTel metrics requires a custom receiver or
a different integration approach. See [OPC-UA getting started](./opc-ua/getting-started/)
for options.

**² PROFINET client libraries**
Native PROFINET RT access from application code requires either vendor
SDKs (typically commercial and hardware-dependent) or low-level raw socket
access. There is no widely-used open source library equivalent to pymodbus
or gopcua for PROFINET. In practice, PROFINET telemetry is most commonly
accessed via an OPC-UA gateway or the controller's data interface.

### Column definitions

**Receiver** — community Collector receiver that supports this protocol.
These are not official OpenTelemetry Collector contrib components.

**Status** — stability as declared by the receiver project: Alpha,
Development, or Experimental. None of these receivers have reached Beta
or Stable status.

**OTel signals** — which OTel signal types the receiver produces: Metrics,
Logs, or Traces. This reflects the current receiver implementation, not
the theoretical maximum of what the protocol could support.

**Client libraries** — open source libraries implementing the protocol,
usable for building custom receivers or instrumenting applications
directly. Not OTel-specific.

**Manual instrumentation** — whether OTel SDK calls can be added to
applications using these libraries to produce traces, metrics, and logs.
"Possible" means no tooling exists but it can be done by hand.

**Auto-instrumentation** — whether an OTel zero-code instrumentation
agent exists for this protocol. Currently none of the protocols covered
here have auto-instrumentation support.

---

## Choosing a collection approach

Industrial protocol telemetry can be collected in three ways, with
different trade-offs:

**Collector receiver (recommended for most cases)** — use the OTel
Collector with a protocol-specific receiver. Requires no changes to
existing infrastructure, produces OTLP-formatted telemetry, and is the
approach documented in this section. Most appropriate for read-only
monitoring of existing systems.

**Custom receiver** — implement a Collector receiver using the protocol
client libraries listed above. Appropriate when existing receivers do not
meet requirements — for example, when you need OPC-UA Variable node
values as metrics rather than logs, or when collecting from a protocol
with no existing receiver such as PROFINET.

**Application instrumentation** — instrument the application or
middleware that communicates with the industrial devices using the OTel
SDK directly. Appropriate when you control the application code, need
traces that cross OT and IT boundaries, or need context propagation
beyond what a standalone receiver can provide.

---

## Related

- [Semantic Conventions](../semantic-conventions/) — attribute and metric
  definitions for each protocol
- [Collector](../collector/) — Collector deployment and configuration
  for industrial environments
- [Getting Started](../getting-started/) — where to begin if you are
  new to OTel in industrial environments