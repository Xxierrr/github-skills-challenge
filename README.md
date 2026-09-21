# Payment Service AIOps Demonstration

This project models a lightweight AIOps workflow for a payment-processing service. The scenario simulates a service that is generally healthy but begins to degrade and eventually fails under a short spike in load. The objective is to detect that change early by combining service metrics with error-level log evidence and then verifying that the anomaly is published and consumed through a simple event pipeline.

## 1. AIOps scenario

The service under observation is `payment-service`. It is expected to process payment requests successfully under normal operating conditions, with stable latency and healthy resource usage. In this dataset, the service appears healthy for the first several minutes and then shows a sudden deterioration:

- request latency rises sharply,
- CPU and memory usage increase substantially,
- timeout messages appear in the logs,
- the logs switch from informational messages to error-level events.

The AIOps workflow detects this degradation by looking at operational telemetry and log severity together. The goal is not just to observe a metric spike, but to determine whether the pattern represents a real service issue that should be turned into an event for downstream handling.

## 2. Operational data description

The project uses a synthetic dataset stored in `data/service_data.json`. Each record contains:

- `timestamp`: the event time in ISO-8601 format,
- `service`: the target service name,
- `response_time_ms`: request latency,
- `cpu_percent`: CPU utilization,
- `memory_percent`: memory utilization,
- `log_level`: severity such as `INFO` or `ERROR`,
- `message`: the log message associated with the event.

The dataset is a short time series covering 10 one-minute intervals from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`.

Normal observations are stable:

- response time: roughly 120-150 ms,
- CPU: about 42-57%,
- memory: about 51-57%,
- log level: `INFO`,
- messages indicate successful processing.

Anomalous observations are marked by a sudden jump in resource usage and latency:

- `2026-09-20T10:05:00`: response time 610 ms, CPU 75%, memory 70%, `ERROR` log,
- `2026-09-20T10:06:00`: response time 640 ms, CPU 94%, memory 91%, `ERROR` log.

## 3. Observations from the logs and metrics

The overall pattern is consistent with a service degradation and timeout condition:

- The first five records and final four records show a steady state with healthy processing times and normal resource levels.
- At `10:05:00`, the service records a steep increase in response time and a switch to an error-level log.
- The next record, at `10:06:00`, escalates further: response time becomes 640 ms and CPU/memory usage spike to dangerous levels.
- The error messages include `Payment service timeout` and `Database connection timeout`, which clearly indicate the service is no longer behaving normally.

These observations support the conclusion that the operational issue is not random noise. It is a short-lived but clear failure period that combines performance degradation and infrastructure/resource stress.

## 4. Anomaly-detection findings

The anomaly detector flags records when the following conditions are true:

- response time exceeds the configured threshold,
- CPU usage exceeds the configured threshold,
- memory usage exceeds the configured threshold,
- the log level is `ERROR`.

The final detected anomalies are:

1. `2026-09-20T10:05:00`
   - response time: 610 ms
   - CPU: 75%
   - memory: 70%
   - log level: `ERROR`
   - reasons: `High response time`, `Error log detected`

2. `2026-09-20T10:06:00`
   - response time: 640 ms
   - CPU: 94%
   - memory: 91%
   - log level: `ERROR`
   - reasons: `High response time`, `High CPU utilization`, `High memory utilization`, `Error log detected`

This matches the abnormal behavior visible in the underlying telemetry. No normal record was incorrectly flagged, and the failure window was captured end-to-end.

## 5. Event-processing flow

The repository implements a minimal event-driven AIOps flow:

1. Each service record is evaluated by the anomaly detector.
2. If the record is anomalous, the detector creates an `ANOMALY` event.
3. The producer publishes the event to an in-memory topic.
4. The consumer reads the messages from that same topic.
5. The consumed events are printed as part of the end-to-end pipeline output.

The main components involved are:

- `src/anomaly_detector.py`: identifies anomalous metrics and log conditions,
- `src/event_topic.py`: in-memory event store,
- `src/event_producer.py`: writes anomaly events to the topic,
- `src/event_consumer.py`: reads the published messages,
- `src/aiops_pipeline.py`: orchestrates the data-loading, detection, publishing, and consumption cycle.

This ensures the workflow demonstrates the full operational pipeline: telemetry in, anomaly detection, event publication, and downstream consumption.

## 6. Final workflow execution result

I verified the end-to-end pipeline by running:

```bash
python3 src/aiops_pipeline.py
```

The final output was:

```text
==================================================
AIOps Pipeline Result
==================================================
Records processed: 10
Anomalies detected: 2
Events consumed: 2

Detected Events:

Service: payment-service
Timestamp: 2026-09-20T10:05:00
Type: ANOMALY
Reasons: High response time, Error log detected

Service: payment-service
Timestamp: 2026-09-20T10:06:00
Type: ANOMALY
Reasons: High response time, High CPU utilization, High memory utilization, Error log detected
```

This confirms the pipeline successfully processed the data, detected the failure pattern, and delivered the anomalies through the event flow.

## 7. Issues identified and corrected

Several issues were present in the repository and corrected during the workflow repair process:

- Import path issue: the project needed a package-level initialization so the `src` modules could be imported consistently during execution and tests.
- Event flow mismatch: the producer and consumer were not operating against a shared event topic. This prevented published anomaly events from being correctly consumed.
- Detection logic error: the checker was accidentally looking for `WARNING`-level logs instead of `ERROR`-level logs, which did not match the actual failure conditions in the dataset.

These issues were fixed without replacing the intended architecture. The final design remains a simple, small-scale event-streaming simulation that behaves as expected.

## 8. Limitation and possible improvement

One limitation of this implementation is that it relies on fixed thresholds for latency, CPU, and memory. That approach is easy to understand and debug, but it does not adapt to normal service variability or seasonal traffic changes. A more robust solution would use baseline-aware or percentile-based thresholds, such as a rolling window or expected-service profile, so the detector can distinguish true failure conditions from routine changes in workload.

Another improvement would be to add richer event metadata, such as a severity score, region or instance ID, and a recommended remediation action. That would make the event pipeline more operationally useful in a real monitoring environment.

## 9. Reproduction steps

To reproduce the demonstration on another machine:

1. Clone the repository and change into the project directory:

```bash
git clone <repository-url>
cd github-skills-challenge
```

2. Create and activate a Python environment if needed:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

3. Install the project dependencies:

```bash
pip install -r requirements.txt
```

4. Run the AIOps pipeline:

```bash
python3 src/aiops_pipeline.py
```

5. Confirm the output shows 10 records processed, 2 anomalies detected, and 2 events consumed.

6. Run the test suite to verify the project state:

```bash
pytest -q
```

Expected result: all tests pass and the anomaly-detection workflow completes successfully.

---

This README documents the scenario, the operational evidence, the corrected implementation issues, and the verified end-to-end workflow result for reproducing the demonstration.

