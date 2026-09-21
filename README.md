# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

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

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

