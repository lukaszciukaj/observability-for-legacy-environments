---
title: Data Quality
linkTitle: Data Quality
description: >-
  Experimental semantic conventions for representing industrial data quality,
  including quality codes, deadbands, scan rates, and measurement uncertainty
  in OpenTelemetry telemetry.
status: experimental
weight: 20
---

> **Status: Experimental.**
> The conventions defined here are not part of the official OpenTelemetry
> semantic conventions specification. The design questions raised in this
> document are open for community discussion.

## The problem

Industrial telemetry is not always good telemetry. A value arriving from a
PLC, sensor, or historian may be stale, degraded, out of range, or explicitly
flagged as unreliable by the source device — and it will still arrive looking
like a number.

In IT observability, a bad metric is typically absent. A failed health check
returns no data. A crashed process emits nothing. The absence of data is itself
a signal.

In OT environments, the opposite is often true. A bad value still arrives.
The historian still records it. The Collector still receives it. Without quality
context attached to that value, there is no way to distinguish a real reading
of `0.0` from a sensor that has failed and is returning its default output.

Carrying bad values downstream without quality context causes real problems:

- **False alerts** — an alarm fires because a stuck sensor is reporting an
  out-of-range value, not because the process is actually out of control
- **Corrupted aggregations** — a bad value pulled into a daily average
  distorts the metric permanently
- **Incorrect operational decisions** — an operator or automated system acts
  on a value that was flagged bad at the source but the flag was lost in
  transit
- **Unreliable ML training data** — models trained on historical data that
  includes unfiltered bad values learn the wrong patterns

OpenTelemetry has no native concept of data quality. These conventions propose
a lightweight, protocol-neutral model for attaching quality context to
industrial telemetry so it is not lost as data moves through an OTel pipeline.

---

## Core concepts

### Quality codes

Quality codes are status flags that indicate the reliability of a telemetry
value. They originate at the source — set by the device, the protocol stack,
or the collection layer — and must be preserved as the value moves downstream.

Industrial protocols vary in how rich their native quality models are. Some
attach detailed status codes directly to every value. Others have no native
quality mechanism, requiring the collection layer to infer quality from
collection outcomes such as timeouts, failed polls, or out-of-range readings.

Regardless of the source protocol, the `industrial.data.quality` attribute
provides a common top-level representation with five states:

| Value | Description |
|---|---|
| `good` | Value is reliable and within expected operating conditions |
| `uncertain` | Value may be unreliable; source is degraded or communication is intermittent |
| `bad` | Value is known to be unreliable; the source or collection layer has flagged an error |
| `stale` | Value has not been updated within the expected scan or publish interval |
| `substituted` | Value has been replaced by a default, calculated, or manually entered value |

Where a protocol provides more granular quality information, the
`industrial.data.quality.detail` attribute captures the protocol-specific
substate alongside the top-level quality value.

---

### Deadbands

A deadband is a threshold that must be exceeded before a new value is
published. If a temperature sensor reads `72.3°C` and the deadband is `0.5°C`,
a new value will only be sent when the reading moves above `72.8°C` or below
`71.8°C`. Readings within that band are suppressed.

Deadbands serve two purposes in industrial systems:

1. **Reducing network and storage load** — a sensor sampling at 100Hz on a
   stable process would generate enormous volumes of nearly identical values
   without deadband filtering
2. **Suppressing noise** — small fluctuations within sensor tolerance are
   filtered out, preventing spurious change events

Deadbands are typically configured at the source — in the server, historian,
or edge gateway — before data reaches the OTel Collector. This means the
Collector may receive data that appears sparse or irregular, not because the
process is stable, but because deadband filtering is active.

Understanding whether deadband filtering is in effect, and with what
threshold, is important context for anyone consuming industrial metrics
through OTel. Two deadband types are common:

| Type | Description |
|---|---|
| `absolute` | Deadband is a fixed value in engineering units — for example `±0.5°C` |
| `percent` | Deadband is a percentage of the configured full sensor range |

---

### Scan rate and event-driven collection

IT observability is built around scraping — an exporter exposes current
values and a scraper collects them at a fixed interval. The scrape interval
determines resolution.

Industrial data collection works differently, and the difference matters for
how OTel timestamps and aggregates industrial metrics.

**Scan-based (polling):** The Collector or gateway polls the device at a
configured interval. The collection interval is fixed and controlled by the
collector. Values are timestamped at collection time unless the device
provides its own timestamp.

**Subscription-based (event-driven):** The device publishes a new value only
when it changes — subject to deadband filtering — or at a configured maximum
publish interval. The timestamp on the value is typically set by the source
device, not by the collector. Values may arrive irregularly.

**Historian export:** Values are read from a historian in batches, with
timestamps reflecting when the historian recorded them — which may be
significantly earlier than when the OTel Collector receives them.

The `industrial.scan.interval_ms` and `industrial.timestamp.source` attributes
provide context for consumers of the telemetry to understand how and when
values were collected.

---

### Timestamp source and clock drift

In OT environments, device clocks are frequently unsynchronised. A PLC may
have a clock that is minutes or hours behind the gateway collecting from it.
When that device timestamp is used as the OTel metric timestamp, the metric
appears to arrive from the past — or in some edge cases, the future.

This causes real problems: time-series backends may reject out-of-order
writes, dashboards display metrics at the wrong point in time, and
correlations between OT metrics and IT logs or traces that use
NTP-synchronised timestamps become unreliable.

The `industrial.timestamp.source` attribute records where the timestamp
originated, so consumers can apply appropriate handling:

| Value | Meaning |
|---|---|
| `device` | Timestamp set by the source device |
| `gateway` | Timestamp applied by an intermediate gateway or protocol bridge |
| `collector` | Timestamp applied by the OTel Collector at collection time |

---

## Proposed attributes

| Attribute | Type | Example | Description |
|---|---|---|---|
| `industrial.data.quality` | string | `good` | Top-level quality state — see values above |
| `industrial.data.quality.detail` | string | `sensor_failure` | Protocol-specific or vendor-specific quality substate |
| `industrial.data.stale` | boolean | `true` | Value has not been updated within the expected interval |
| `industrial.data.substituted` | boolean | `true` | Value is a substitute rather than a live reading |
| `industrial.deadband.value` | double | `0.5` | Deadband threshold applied before the value was published |
| `industrial.deadband.type` | string | `absolute` | Deadband type: `absolute` or `percent` |
| `industrial.scan.interval_ms` | int | `1000` | Configured scan or polling interval in milliseconds |
| `industrial.timestamp.source` | string | `device` | Origin of the timestamp: `device`, `gateway`, or `collector` |

---

## Open design questions

The following questions are unresolved. Community input is welcome.

**1. Attribute or separate metric?**

Should `industrial.data.quality` be an attribute on the existing metric, or
should quality be reported as a separate metric alongside the value?

Option A — quality as an attribute:
```yaml
metric: industrial.temperature
value: 72.4
attributes:
  industrial.data.quality: uncertain
  industrial.asset.id: mixer-07
```

Option B — quality as a separate metric:
```yaml
metric: industrial.temperature
value: 72.4

metric: industrial.temperature.quality
value: 0   # 0 = bad, 1 = uncertain, 2 = good
attributes:
  industrial.asset.id: mixer-07
```

Option A is simpler and keeps quality co-located with the value. Option B
allows quality to be queried and alerted on independently, and maps more
naturally to numeric threshold alerting in common observability backends.
It also avoids the high-cardinality risk of adding a quality label to
every metric.

**2. How to handle bad values**

Should a `bad` quality value still be emitted as a metric data point, or
should it be dropped at the Collector and recorded only as a log event?

Emitting bad values preserves the full history and allows downstream systems
to make their own quality decisions. Dropping them keeps metrics clean but
loses information that may be important for root cause analysis. A Collector
processor filter could implement either policy, configured per deployment.

**3. Deadband and the OTel data model**

Deadband filtering is a core concept in industrial data collection, but it
sits awkwardly in the OTel data model. Deadband parameters describe how the
source decided whether to publish a value — they are metadata about the
collection process, not about the value itself.

It is unclear whether deadband parameters belong as metric attributes, as
resource attributes on the receiver, or as metadata outside the OTel data
model entirely. Feedback from the OTel specification working group would be
valuable here.

---

## Related

- [Common attributes](../common/) — site, asset, and controller context
- [Collector configuration](../../collector/configuration/) — filtering
  and transforming quality attributes in the OTel pipeline