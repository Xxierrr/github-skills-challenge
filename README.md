# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## Service overview

This assessment monitors a payment processing service, `payment-service`, which handles incoming customer requests and records operational telemetry such as latency, CPU usage, memory usage, and log events. The service is expected to operate normally under steady load, processing requests successfully and emitting informational log entries.

## Operational problem being addressed

The goal is to detect when the service begins to behave abnormally under stress or failure states. In this dataset, the operational issue is a degradation and outage pattern: response times spike, resource usage rises, and timeout-related error logs appear. These symptoms indicate that the service is not operating normally and needs investigation.

## Purpose of AIOps in this assessment

AIOps in this assessment is used to combine telemetry metrics and log evidence into a readable detection workflow. Instead of inspecting raw data manually, the anomaly-detection pipeline looks for unusual metric patterns and relevant error signals, then reports which observations were flagged and why. This helps distinguish healthy behaviour from abnormal events and makes the operational problem easier to understand and act on.

## Operational data analysis

The repository contains a small synthetic service log dataset in [data/service_data.json](data/service_data.json). Based on the records in that file:

1. Metrics fields
   - `response_time_ms`: request latency measured in milliseconds.
   - `cpu_percent`: CPU utilization percentage.
   - `memory_percent`: memory utilization percentage.
   - These are numeric operational metrics that can be compared over time to detect changes in performance or health.

2. Log information fields
   - `log_level`: severity indicator such as `INFO` or `ERROR`.
   - `message`: human-readable log message describing the event.
   - `timestamp` provides the time context for the log, while `service` identifies the application component affected.

3. How timestamps are used
   - Each record has a UTC-like ISO timestamp in the format `YYYY-MM-DDTHH:MM:SS`.
   - The timestamps are sequential, spaced one minute apart from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`.
   - This makes it possible to view the service state as a short time series and identify changes that happen over time.

4. Normal behaviour
   - The first five records and the final four records show a stable pattern: response time stays roughly 120-150 ms, CPU stays around 42-57%, and memory stays around 51-57%.
   - `log_level` is `INFO` throughout these records.
   - Messages such as `Payment request processed successfully` indicate successful transactions without errors.
   - This represents the expected operating range for the service.

5. Unusual behaviour
   - The records at `10:05:00` and `10:06:00` are clearly abnormal.
   - `response_time_ms` jumps to 610 and 640 ms, far above the normal range.
   - `cpu_percent` rises to 75 and 94, and `memory_percent` rises to 70 and 91, indicating resource pressure.
   - `log_level` changes to `ERROR`, and the messages report `Payment service timeout` and `Database connection timeout`.
   - These observations appear to represent performance degradation and a service-level failure condition, unlike the regular successful requests seen earlier and later.

Overall, the data supports a simple health model: low, stable metrics and `INFO` logs indicate healthy service behaviour, while elevated latencies, rising resource usage, and `ERROR` messages indicate unusual or failing conditions.

## Task 3: Identify anomalies with the provided detector

I used the repository’s supplied anomaly detector through the pipeline to analyze the operational data in [data/service_data.json](data/service_data.json). The detector was run with the project’s pipeline and produced the following output:

- `Records processed: 10`
- `Anomalies detected: 2`
- `Events consumed: 2`

### Detected anomalies

The detector flagged two observations as anomalous:

1. `2026-09-20T10:05:00`
   - `service`: `payment-service`
   - `response_time_ms`: `610`
   - `log_level`: `ERROR`
   - `cpu_percent`: `75`
   - `memory_percent`: `70`
   - Reasons: `High response time`, `Error log detected`

2. `2026-09-20T10:06:00`
   - `service`: `payment-service`
   - `response_time_ms`: `640`
   - `log_level`: `ERROR`
   - `cpu_percent`: `94`
   - `memory_percent`: `91`
   - Reasons: `High response time`, `High CPU utilization`, `High memory utilization`, `Error log detected`

These two observations are the only ones flagged by the detection mechanism. They align with the obvious abnormal periods in the data: high latency and elevated resource usage paired with timeout-related error events.

### Relevant metric and log information

The detector uses the following fields to decide whether an observation is anomalous:

- Metrics:
  - `response_time_ms`
  - `cpu_percent`
  - `memory_percent`
- Log information:
  - `log_level`
  - `message`
  - `timestamp`
  - `service`

For both flagged records, the metric values jump well beyond the normal behavior seen in the surrounding observations. The log events also become `ERROR`, and the messages explicitly mention timeouts.

### Normal vs anomalous observations

Normal observations:
- The records from `10:00:00` through `10:04:00`, plus `10:07:00` through `10:09:00`, are consistent with stable service behavior.
- Their latency stays around `120-150 ms`, CPU is roughly `42-57%`, and memory is around `51-57%`.
- Their `log_level` is `INFO`, and the message indicates successful processing.

Anomalous observations:
- The records at `10:05:00` and `10:06:00` are abnormal.
- They exceed the configured thresholds for response time and resource usage and include error-level log events.

Expected anomaly check:
- No expected anomaly was missed in this dataset: the abnormal periods at `10:05:00` and `10:06:00` were both detected.

False positive check:
- No normal event appears to have been incorrectly flagged by the detector in this dataset.

### Limitation / possible improvement

One limitation is that the detection logic is threshold-based and uses only a few metrics and a single log-level check. It may miss nuanced problems that are not extreme enough to exceed the thresholds, or it may overreact if normal traffic patterns vary by time of day. A useful improvement would be to add baseline-aware thresholds (for example, a rolling mean or percentile-based alerting) so the detector can identify deviations relative to expected service behavior rather than fixed absolute numbers alone.

## Task 4: Verify the AIOps event flow

The repository contains a simple event-streaming simulation made up of the following components:

- `Producer`: creates and emits anomaly events after detection.
- `Topic`: stores messages in an in-memory stream for downstream consumers.
- `Consumer`: reads events from the topic.
- `Event/message`: the payload that carries the anomaly context.

The workflow is:

1. A record is analyzed by the anomaly detector.
2. If the record is anomalous, an `ANOMALY` event is created.
3. The producer publishes that event to the topic.
4. The consumer reads the messages from the same topic.
5. The consumed event is processed and passed downstream as part of the AIOps workflow.

Execution result from the project’s pipeline:

```text
AIOps Pipeline Result
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

This confirms that the anomaly was identified, converted into an event, published to the topic, received by the consumer, and moved through the downstream event-processing path as expected.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

