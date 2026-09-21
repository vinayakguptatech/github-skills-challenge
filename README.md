# AIOps Simulation Challenge

This project simulates an AIOps workflow for a `payment-service` that handles payment requests. The goal is to detect abnormal service behavior, convert it into events, and pass those events through a simple producer-topic-consumer flow.

## Scenario

The service normally processes requests successfully with low latency and moderate resource consumption. When response time or resource usage spikes, and error logs appear, the system should flag the event as an anomaly and route it through the processing pipeline.

## Operational data analysis

The dataset in `data/service_data.json` contains synthetic operational records for the `payment-service`.

### 1. Metric fields
The metric fields are:
- `response_time_ms`
- `cpu_percent`
- `memory_percent`

These represent quantitative service performance and resource usage metrics.

### 2. Log information
The log-related fields are:
- `log_level`
- `message`

These capture the severity and textual description of the event.

### 3. Use of timestamps
The `timestamp` field records when each observation was generated. The records progress in a sequence from `2026-09-20T10:00:00` to `2026-09-20T10:09:00`, allowing the workflow to observe a pattern over time. The anomaly window becomes clear around `10:05:00` and `10:06:00`.

### 4. Normal behavior
Normal observations include:
- `response_time_ms` around 120–150 ms
- `cpu_percent` around 42–50%
- `memory_percent` around 51–57%
- `log_level` values of `INFO`
- message text such as `Payment request processed successfully`

These values indicate expected, stable service operation.

### 5. Unusual behavior
Unusual observations include:
- `response_time_ms` of 610 and 640 ms
- `cpu_percent` of 75% and 94%
- `memory_percent` of 70% and 91%
- `log_level` values of `ERROR`
- messages such as `Payment service timeout` and `Database connection timeout`

These records indicate degraded service performance and likely operational issues.

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

