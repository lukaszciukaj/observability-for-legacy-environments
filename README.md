# 🏭 Observability for Legacy Environments

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenTelemetry Member](https://img.shields.io/badge/OpenTelemetry-Member-blue?logo=opentelemetry)](https://opentelemetry.io)
[![CNCF](https://img.shields.io/badge/CNCF-Cloud%20Native%20RTP-informational)](https://community.cncf.io)
[![Medium](https://img.shields.io/badge/Medium-Articles-black?logo=medium)](https://medium.com/@lukasz.ciukaj)

> Reference architectures, integration patterns, and practical frameworks for bringing modern observability to legacy, industrial, and operational technology (OT) environments — using open standards like OpenTelemetry, Prometheus, and MQTT.

---

## 📋 Table of Contents

- [Mission](#-mission)
- [Key Areas](#-key-areas)
- [Publications & Articles](#-publications--articles)
- [Projects](#-projects)
- [Related Technologies](#-related-technologies)
- [Author](#-author)
- [Contributions](#-contributions)
- [License](#-license)

---

## 🎯 Mission

Many manufacturing, industrial, and legacy environments operate with limited telemetry visibility, proprietary protocols, fragmented monitoring systems, and aging infrastructure — and simply replacing those systems isn't an option.

This repository bridges that gap with:

- Practical observability architectures for OT/ICS environments
- Industrial telemetry integration patterns
- Legacy modernization decision frameworks
- OpenTelemetry instrumentation examples
- Protocol translation and normalization concepts
- Vendor-neutral implementation guidance

The focus is on **pragmatic modernization** — improving visibility without disruptive replacement.

---

## 📊 Key Areas

### OpenTelemetry for Industrial Systems

Extending OpenTelemetry beyond cloud-native workloads into industrial and legacy environments:

- Protocol-aware telemetry extraction (Modbus, OPC UA, MQTT)
- Manual and auto instrumentation strategies
- Telemetry normalization and semantic conventions for OT
- Distributed tracing concepts adapted for industrial workflows
- Custom OpenTelemetry Collector receiver development
- Edge telemetry aggregation patterns

### Legacy Modernization Frameworks

Decision frameworks and reference architectures for:

- Evaluating telemetry extraction options without disrupting operations
- Selecting integration approaches (passive tap, active poll, protocol bridge)
- Assessing protocol capabilities and constraints
- Designing phased modernization strategies
- Balancing observability investment against operational risk

### Unified Telemetry Pipelines

Patterns for centralizing telemetry from heterogeneous systems:

- OpenTelemetry Collector pipeline design
- MQTT normalization and bridge patterns
- Custom protocol adapters and receivers
- Log parsing pipelines for legacy formats
- Edge aggregation before cloud egress

---

## 🌎 Why This Matters

Legacy systems power critical infrastructure across manufacturing, energy, water, and transportation — and most will remain operational for decades. Visibility into these systems underpins:

- **Operational awareness** — understanding what your systems are actually doing
- **Predictive maintenance** — catching anomalies before they become failures
- **Cybersecurity** — detecting anomalous behavior in OT environments
- **Analytics & AI** — telemetry is the foundation for any data-driven initiative
- **Compliance** — audit trails and monitoring evidence for regulated industries

> In many environments, **partial visibility is significantly better than no visibility at all.**

---

## 📝 Publications & Articles

### OpenTelemetry & Industrial Observability

- **[Applying OpenTelemetry Security Practices in Legacy Environments (OpenTelemetry Blog)](https://opentelemetry.io/blog/2026/security-legacy-environments/)**
  
  Security challenges unique to legacy and industrial environments — covering Collector placement, network segmentation, sensitive operational data handling, and a pragmatic decision model for securing telemetry pipelines under real-world constraints.
 
- **[Extending OpenTelemetry into Industrial Environments with a Modbus Receiver (Medium)](https://medium.com/@lukasz.ciukaj/extending-opentelemetry-into-industrial-environments-with-a-modbus-receiver-7b5cb42bd430)**
  
  Walks through building a custom OpenTelemetry Collector receiver for Modbus TCP — covering the full pipeline from PLC register polling to OTel gauge metrics. This article is related to my project - [Modbus Receiver for OpenTelemetry Collector](https://github.com/lukaszciukaj/modbusreceiver)
  
- **[Bringing OpenTelemetry to OPC UA: Manual Instrumentation of an Industrial Protocol (Medium)](https://medium.com/@lukasz.ciukaj/bringing-opentelemetry-to-opc-ua-manual-instrumentation-of-an-industrial-protocol-2cfa03475164)**
  
  Practical example of instrumenting OPC UA workflows with OpenTelemetry to improve visibility into industrial communications and legacy manufacturing systems.
- **[Demystifying OpenTelemetry: Why You Shouldn't Fear Observability in Traditional Environments (OpenTelemetry Blog)](https://opentelemetry.io/blog/2026/demystifying-opentelemetry/)**
  
  Simplifying observability adoption in non-cloud-native and traditional enterprise environments.
- **[Your Critical Legacy App is a Black Box? Let's Change That in 5 Minutes! (OpenTelemetry Blog)](https://opentelemetry.io/blog/2025/opentelemetry-for-legacy-app/)**
  
  Improving visibility into legacy applications using lightweight OpenTelemetry instrumentation approaches.
### Industrial Modernization & Operational Resilience

- **[Observability as Critical Infrastructure: Lessons from the Manufacturing USA Program Strategic Plan - Dcember 2025 (Medium)](https://medium.com/@lukasz.ciukaj/4e23501d386b)**

  How operational visibility, telemetry interoperability, and open standards are becoming foundational capabilities for smart manufacturing — drawing on the Manufacturing USA Program Strategic Plan (December 2025).
 
- **[Driving Industrial Modernization: The Role of Observability and Data Analytics (Medium)](https://medium.com/@lukasz.ciukaj/driving-industrial-modernization-the-role-of-observability-and-data-analytics-478843e8c292)**
  
  How observability, telemetry pipelines, and analytics frameworks can modernize manufacturing environments without disruptive replacement.
- **[Resilient by Design: The Role of AI and Security in Observability for Plant Operations (Splunk Blog)](https://www.splunk.com/en_us/blog/observability/resilient-by-design-the-role-of-ai-and-security-in-observability-for-plant-operations.html)**  

  AI-driven observability and security analytics for improving operational resilience across plant operations and industrial systems.

---

## 🔬 Projects & Contributions

### Modbus Receiver for OpenTelemetry Collector

An experimental OpenTelemetry Collector receiver for industrial and OT environments — enabling telemetry extraction from Modbus TCP devices and conversion into standard OpenTelemetry metrics. Written in Go, structured to follow official `otelcol-contrib` conventions.

🔗 [lukaszciukaj/modbusreceiver](https://github.com/lukaszciukaj/modbusreceiver)

### OpenTelemetry for Industrial Environments (Documentation)

Community documentation for applying OpenTelemetry in industrial and OT
environments — covering protocols, semantic conventions, Collector deployment,
and security.

🔗 [docs/otel-industrial-environments](./docs/otel-industrial-environments/)

---

## 👨‍💻 Author

**Lukasz Ciukaj** — Solutions Architect specializing in standards-based observability and operational resilience, with a focus on bridging OT, manufacturing, and legacy enterprise systems with modern cloud-native observability standards.

- 🔵 Contributor and official **OpenTelemetry Member**
- 🤝 Co-lead, **OpenTelemetry Blueprints** initiative
- 🌐 Founder & Co-organizer, **Cloud Native RTP** community group (CNCF)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-lciukaj-blue?logo=linkedin)](https://www.linkedin.com/in/lciukaj)
[![Medium](https://img.shields.io/badge/Medium-%40lukasz.ciukaj-black?logo=medium)](https://medium.com/@lukasz.ciukaj)

---

## 🤝 Contributions

Contributions, ideas, architecture discussions, and feedback are welcome.

**Good areas to contribute:**
- OpenTelemetry instrumentation examples
- Industrial protocol integrations and receiver concepts
- Architecture diagrams and reference implementations
- OT observability case studies
- Documentation improvements

**To get involved:**
- 🐛 Open a [GitHub Issue](../../issues) for bugs, ideas, or architecture discussions
- 🔗 Connect via [LinkedIn](https://www.linkedin.com/in/lciukaj)

---

## 📄 License

This repository is licensed under the [MIT License](LICENSE) unless otherwise
stated. The `docs/otel-industrial-environments/` section is licensed under
[Apache 2.0](docs/otel-industrial-environments/LICENSE) for compatibility
with the OpenTelemetry project.

---

## Disclaimer

The views and opinions expressed in this repository are solely my own and do not reflect the views of my employer or any affiliated organization.

All architectures, examples, integrations, and receiver concepts are provided for educational and reference purposes only. They should be independently evaluated, tested, and validated before any use in production or operational environments. Industrial systems are safety-critical — always follow site procedures, vendor guidance, and change management processes.

---

*See also: [REFERENCES.md](REFERENCES.md) · [CHANGELOG.md](CHANGELOG.md)*