# AIOps Simulation Challenge

This project simulates an AIOps workflow for a `payment-service` that handles payment requests. The goal is to detect abnormal service behavior, convert it into events, and pass those events through a simple producer-topic-consumer flow.

## Scenario

The service normally processes requests successfully with low latency and moderate resource consumption. When response time or resource usage spikes, and error logs appear, the system should flag the event as an anomaly and route it through the processing pipeline.

## Operational data

The dataset in `data/service_data.json` contains service telemetry:

- `timestamp`
- `service`
- `response_time_ms`
- `cpu_percent`
- `memory_percent`
- `log_level`
- `message`

Normal records have low latency, moderate CPU/memory values, and `INFO` messages like `Payment request processed successfully`.

Abnormal records show high latency and resource usage with `ERROR` messages like `Payment service timeout` and `Database connection timeout`.

## Anomaly detection

The detector in `src/anomaly_detector.py` flags records when:

- response time > 500 ms
- CPU > 80%
- memory > 80%

Detected anomalies include:

- `2026-09-20T10:05:00`: high response time, timeout log
- `2026-09-20T10:06:00`: high response time, high CPU, high memory, database timeout log

This is a simple threshold-based approach and can be improved with rolling baselines or trend-based detection.

## Event flow

The workflow is:

Operational data → anomaly detection → event → producer → topic → consumer → final AIOps output

Key components:

- `EventProducer` publishes anomaly events
- `EventTopic` stores the event messages
- `EventConsumer` reads the messages from the topic
- `aiops_pipeline.py` runs the end-to-end flow

## Issues fixed

Two issues were identified:

1. Python import path was missing for `src`
2. Producer and consumer were using different topic names

These were corrected so the workflow runs in the intended architecture.

## Reproduce

```bash
cd /workspaces/github-skills-challenge
export PYTHONPATH=.
pytest -q
PYTHONPATH=. python src/aiops_pipeline.py
```

This validates the normal/anomaly logic, event creation, and end-to-end pipeline behavior.

