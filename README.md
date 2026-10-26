# PulseShift

An engineering project exploring reliable market-data streaming and seasonality-aware drift detection.

## The problem

Market features such as volatility, spread, and order-book imbalance change over time. Some changes follow recurring patterns, while others may reflect an unexpected shift in market behavior.

PulseShift explores how to distinguish those changes using observations available as events arrive, without waiting for later outcomes. Reliable ingestion and replay are part of that problem: missing or repeated data can distort the evidence used by a detector.

## Planned stages

1. Ingest, normalize, and replay market events with a Go producer and a Kafka-compatible broker.
2. Compute rolling features with Flink, serve current values through Redis, and retain history in Iceberg tables.
3. Establish a seasonality-aware statistical detector, then evaluate whether a KAN-based model improves the results.
4. Build a Python API and React dashboard to inspect features, alerts, latency, and recovery behavior.

## Current status

Early development. This repository begins with the project scope and roadmap. A local prototype for synthetic events and Go-to-Python Protobuf decoding has passed offline checks; its implementation will be introduced incrementally.

Live ingestion, the feature pipeline, drift detection, and the dashboard are planned work. Broker integration and a Flink/Iceberg compatibility check still need successful runs.

## Learning focus

The project brings together data contracts, event-time processing, stateful calculations, failure recovery, and model evaluation. Technical decisions, experiments, and results will be documented as the work progresses.
