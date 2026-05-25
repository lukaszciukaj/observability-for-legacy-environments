---
title: Semantic Conventions
linkTitle: Semantic Conventions
description: >-
  Experimental semantic conventions for OpenTelemetry in industrial and
  operational technology (OT) environments.
status: experimental
weight: 50
---

> **Status: Experimental.**
> The conventions defined here are not part of the official OpenTelemetry
> semantic conventions specification. Attribute names, definitions, and
> structure are subject to change. Do not treat these as stable in production
> systems without accepting that risk.

## What are semantic conventions?

OpenTelemetry semantic conventions define standardized names and attributes for
telemetry data — ensuring that a `k8s.pod.name` means the same thing regardless
of who collected it or where it is stored. They are the shared vocabulary that
makes telemetry portable, correlatable, and vendor-neutral.

The official OpenTelemetry semantic conventions cover applications,
infrastructure, cloud services, databases, and messaging systems. Industrial
and operational technology (OT) environments are not yet covered.

These conventions are a community attempt to fill that gap.

## Why industrial environments need their own conventions

A temperature reading of `72` tells you very little on its own. Without
consistent metadata, it is impossible to know which asset produced it, which
protocol delivered it, what unit it is in, or whether the value is trustworthy.

Industrial telemetry comes from heterogeneous systems — PLCs, SCADA, historians,
edge gateways, sensors, drives, and more — each with its own naming conventions,
addressing schemes, and data models. Consistent semantic conventions make it
possible to correlate signals across these systems, build meaningful dashboards
and alerts, and move telemetry through an OTel pipeline without losing context.

## Relationship to official OpenTelemetry conventions

These conventions are designed to complement, not replace, the existing
OpenTelemetry semantic conventions. Where official conventions already cover a
concept (for example, `host.*` or `network.*` attributes), those should be
preferred. The industrial conventions defined here cover concepts that have no
equivalent in the current official specification.

If and when these conventions are proposed to the OpenTelemetry project through
the OTEP process, they will be subject to review and may change significantly.

## Convention groups

| Group | File | Description |
|---|---|---|
| Common | [common.md](./common/) | Site, asset, and controller attributes shared across all protocols |
| Data quality | [data-quality.md](./data-quality/) | Quality codes, deadbands, uncertainty |
| OPC-UA | [opc-ua.md](./opc-ua/) | OPC-UA node and subscription attributes |
| Modbus | [modbus.md](./modbus/) | Register types, addresses, unit IDs |
| MQTT Sparkplug | [mqtt-sparkplug.md](./mqtt-sparkplug/) | Sparkplug B topic and payload attributes |
| PROFINET | [profinet.md](./profinet/) | PROFINET device and slot attributes |

## Attribute naming conventions

All industrial attributes use the `industrial.` namespace prefix, following the
OTel convention of dot-separated namespaces. Protocol-specific attributes use
a further nested prefix, for example `industrial.protocol.modbus.*`.

Attribute names follow these rules:

- Lowercase, dot-separated
- Descriptive and unambiguous without context
- Singular nouns where possible (`industrial.site`, not `industrial.sites`)
- Consistent with OTel naming patterns where applicable

## Stability and versioning

All conventions in this documentation are **experimental**. Breaking changes
may occur between versions. Each page notes the stability status of individual
attributes where it differs from the page default.

Changes to conventions are recorded in the
[CHANGELOG](../../CHANGELOG.md).