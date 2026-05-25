---
title: OpenTelemetry for Industrial Environments
linkTitle: Industrial Environments
description: >-
  Community-maintained guidance for applying OpenTelemetry in industrial and
  operational technology (OT) environments, covering legacy protocols, edge
  deployments, and brownfield integrations.
status: experimental
weight: 40
---

> **This is not official OpenTelemetry documentation.**
> This project is an independent, community-driven effort. It is not affiliated
> with, endorsed by, or maintained by the OpenTelemetry project or the Cloud
> Native Computing Foundation (CNCF). All semantic conventions defined here
> are experimental and subject to change.

## What this is

OpenTelemetry emerged from cloud-native observability needs, particularly microservices and distributed systems. Industrial
environments like manufacturing plants, utilities, oil and gas facilities, building
automation systems, present a fundamentally different set of constraints: legacy
protocols from the 1980s and 1990s, air-gapped networks, real-time control
requirements, safety-critical systems, and hardware that was never designed with
observability in mind.

This documentation exists to bridge that gap. It provides practical guidance for
engineers and architects bringing OpenTelemetry into operational technology (OT)
environments, across the full range of industrial protocols and deployment patterns.

## Scope and intent

This project deliberately brings together several parallel efforts that exist
across the community:

- Experimental semantic conventions for industrial protocols (OPC-UA, Modbus,
  MQTT Sparkplug, PROFINET, and others)
- Collector configuration patterns for industrial receivers
- Deployment guidance for constrained and air-gapped environments
- Architectural blueprints for OT/IT convergence using OTel as the telemetry layer

Rather than these living in separate repositories, blog posts, or internal wikis,
the goal is a single, coherent reference that the community can build on together.

## Future direction

It is the intent of this project to propose this documentation for inclusion in
the official OpenTelemetry documentation, under
[Platforms](https://opentelemetry.io/docs/platforms/) as
**Industrial Environments** — alongside the existing Kubernetes, FaaS, and
Client-side Apps sections.

This will require formal review by the OpenTelemetry documentation SIG and
relevant working groups, as well as separate proposals for any semantic
conventions. **Until that process is complete, nothing here should be treated 
as an official OpenTelemetry specification**.

If you are interested in participating in that process, see
[Contributing](#contributing) below.

## What is covered

| Section | Description |
|---|---|
| [Getting Started](./getting-started/) | Entry points for brownfield and greenfield deployments |
| [Concepts](./concepts/) | OT/IT convergence, signal mapping, data quality, glossary |
| [Collector](./collector/) | Edge deployment, air-gapped networks, receivers, configuration |
| [Protocols](./protocols/) | Per-protocol integration guides |
| [Semantic Conventions](./semantic-conventions/) | Experimental attribute and metric definitions |
| [Security](./security/) | Network segmentation, Purdue model, sensitive data |

## Status

All content in this documentation is **experimental**. The semantic conventions
defined here have not been submitted to or accepted by the OpenTelemetry
specification. Protocol coverage is uneven — some protocols have detailed
guidance, others are placeholders awaiting contribution.

| Protocol | Documentation status | Semantic conventions |
|---|---|---|
| OPC-UA | In progress | Experimental draft |
| Modbus | In progress | Experimental draft |
| MQTT Sparkplug | In progress | Experimental draft |
| PROFINET | Placeholder | Not started |

## Contributing

Contributions are welcome and encouraged. This documentation covers a broad
domain and no single person has deep expertise across every protocol,
deployment pattern, and use case it touches.

**Ways to contribute:**

- **Fill in a placeholder** — several protocol pages and concept pages are
  intentionally left as stubs. If you have hands-on experience with a protocol,
  that knowledge is exactly what is needed.
- **Review experimental conventions** — the semantic conventions defined here
  need review from people who work with these protocols day to day. Naming,
  units, cardinality, and attribute design all benefit from real-world
  experience.
- **Add a new protocol** — if you work with a protocol not covered here
  (DNP3, EtherNet/IP, BACnet, IEC 61850, and others), the
  `protocols/` template is designed to be repeatable.
- **Improve existing content** — corrections, clarifications, additional
  examples, and better explanations are all valuable.

**To contribute:**

1. Open an issue describing what you want to add or change.
2. Fork the repository and create a branch from `docs/otel-industrial-environments`.
3. Follow the structure established in existing pages. For new protocols,
   use the pages in `protocols/opc-ua/` as a template.
4. Open a pull request with a clear description of what changed and why.

For significant additions — new sections, new protocols, or changes to the
semantic conventions — please open an issue for discussion before writing,
so effort is not duplicated and the change fits the overall structure.

If you are interested in the longer-term effort to propose this content to the
OpenTelemetry project, mention that in your issue. Coordination on that process
will happen in the open.