# 📚 References

A curated collection of technical references, standards, documentation, and learning resources relevant to industrial observability, OpenTelemetry, and legacy modernization. Organized by topic for quick navigation.

---

## 📋 Table of Contents

- [OpenTelemetry](#opentelemetry)
- [Industrial Protocols](#industrial-protocols)
- [Observability Backends & Tools](#observability-backends--tools)
- [Edge & OT Architecture](#edge--ot-architecture)
- [Security in OT Environments](#security-in-ot-environments)

---

## OpenTelemetry

### Core Documentation

| Resource | Description |
|---|---|
| [OpenTelemetry Specification](https://opentelemetry.io/docs/specs/otel/) | The authoritative OTel data model and API specification |
| [OTel Collector Documentation](https://opentelemetry.io/docs/collector/) | Architecture, configuration, and deployment of the Collector |
| [OTel Collector Contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib) | Community receivers, processors, and exporters |
| [OTel Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/) | Standard attribute names and conventions for telemetry data |
| [OTel Registry](https://opentelemetry.io/ecosystem/registry/) | Community plugins, integrations, and instrumentation libraries |

### Collector Development

| Resource | Description |
|---|---|
| [Building a Custom Receiver](https://opentelemetry.io/docs/collector/extend/custom-component/receiver/) | Official guide to developing Collector receivers |
| [opentelemetry-collector-builder (ocb)](https://github.com/open-telemetry/opentelemetry-collector/tree/main/cmd/builder) | Tool for building custom Collector distributions |
| [Collector Configuration Guide](https://opentelemetry.io/docs/collector/configuration/) | Pipelines, receivers, processors, exporters, connectors |
| [OTel Go SDK](https://pkg.go.dev/go.opentelemetry.io/otel) | Go SDK reference for building receivers and instrumentation |

### Instrumentation Guides

| Resource | Description |
|---|---|
| [OTel for Go](https://opentelemetry.io/docs/languages/go/) | Go instrumentation guide |
| [OTel for Python](https://opentelemetry.io/docs/languages/python/) | Python instrumentation guide |
| [Manual Instrumentation Concepts](https://opentelemetry.io/docs/concepts/instrumentation/manual/) | When and how to instrument code manually |
| [Auto Instrumentation](https://opentelemetry.io/docs/concepts/instrumentation/automatic/) | Zero-code and agent-based approaches |

---

## Industrial Protocols

### Modbus

| Resource | Description |
|---|---|
| [Modbus Application Protocol Specification v1.1b3](https://modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf) | Official Modbus protocol specification — function codes, register banks, data model |
| [Modbus over TCP/IP](https://modbus.org/docs/Modbus_Messaging_Implementation_Guide_V1_0b.pdf) | Modbus TCP implementation guide |
| [Modbus Wikipedia](https://en.wikipedia.org/wiki/Modbus) | Protocol overview, register types, function codes |
| [pymodbus](https://pymodbus.readthedocs.io/) | Python Modbus library for simulation and testing |
| [go-modbus](https://github.com/grid-x/modbus) | Go Modbus TCP/RTU client library |
| [modbusreceiver](https://github.com/lukaszciukaj/modbusreceiver) | Custom OTel Collector receiver for Modbus TCP |

### OPC UA

| Resource | Description |
|---|---|
| [OPC Foundation](https://opcfoundation.org/) | Official OPC UA standards body |
| [OPC UA Specification](https://reference.opcfoundation.org/) | Full OPC UA specification reference |
| [open62541](https://www.open62541.org/) | Open-source OPC UA implementation in C |
| [asyncua](https://github.com/FreeOpcUa/opcua-asyncio) | Async Python OPC UA library |
| [OPC UA Receiver](https://github.com/bruegth/opentelemetry-collector-opcua-receiver) | Custom OTel Collector receiver for OPC UA |

---

## Observability Backends & Tools

### Metrics

| Resource | Description |
|---|---|
| [Prometheus Documentation](https://prometheus.io/docs/) | Metrics collection, storage, and querying |
| [PromQL Reference](https://prometheus.io/docs/prometheus/latest/querying/basics/) | Prometheus query language |
| [Grafana Documentation](https://grafana.com/docs/) | Dashboarding and visualization |

### Tracing

| Resource | Description |
|---|---|
| [Jaeger Documentation](https://www.jaegertracing.io/docs/) | Distributed tracing system |

### Logging & Pipelines

| Resource | Description |
|---|---|
| [Fluent Bit Documentation](https://docs.fluentbit.io/) | Lightweight log processor and forwarder |
| [Logstash](https://www.elastic.co/logstash) | Log pipeline for the Elastic Stack |
| [OpenSearch](https://opensearch.org/docs/) | Open-source search and analytics suite |

---

## Edge & OT Architecture

| Resource | Description |
|---|---|
| [CNCF Edge Computing Landscape](https://landscape.cncf.io/guide#runtime--cloud-native-edge) | Overview of edge-native projects and tools |
| [IIoT Reference Architecture](https://www.iiconsortium.org/iira/) | Industrial Internet Reference Architecture (IIC) |
| [Purdue Enterprise Reference Architecture](https://en.wikipedia.org/wiki/Purdue_Enterprise_Reference_Architecture) | Classic OT network segmentation model |
| [NAMUR Open Architecture (NOA)](https://www.namur.net/en/recommendations-and-worksheets/noa.html) | Modern OT architecture framework |

---

## Security in OT Environments

| Resource | Description |
|---|---|
| [IEC 62443](https://www.iec.ch/iec62443) | Industrial cybersecurity standard series |
| [NIST SP 800-82](https://csrc.nist.gov/publications/detail/sp/800-82/rev-3/final) | Guide to OT/ICS security |
| [CISA ICS Security Resources](https://www.cisa.gov/topics/industrial-control-systems) | US government ICS security guidance |
| [MITRE ATT&CK for ICS](https://attack.mitre.org/matrices/ics/) | Threat modeling framework for industrial systems |

---

*References are maintained on a best-effort basis. Links may change over time — please open an issue if you find a broken link.*

*Back to [README.md](README.md)*