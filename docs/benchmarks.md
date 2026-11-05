# DefectRay benchmark method and evidence register

Status: **METHOD APPROVED ON 2026-11-05 — NO PERFORMANCE OR ML RESULTS EXIST**

This document owns experiment procedures and results. The approved [specification](spec.md), especially PERF-001 through PERF-005, owns mandatory measurement definitions. [plan.md](plan.md) owns the protected 18-hour allocation, actual task time, reviews and acceptance. No optimization improvement is promised.

## 1. Measurement questions

1. What are CPU model-only inference throughput and latency after warm-up?
2. What are full-pipeline unique-success throughput and p95 admission-to-commit latency with one versus two worker processes?
3. What are each process's peak RSS and the application's separately accounted owned-buffer high-water marks?
4. Does a measured thread/configuration or small allocation change help under the same workload/model/hardware budget?
5. How long does the interrupted task take to commit successfully after a controlled worker crash?
6. What held-out precision, recall, confusion counts and error examples describe the frozen detector?

The mandatory initial experiments run on one Linux/WSL host. They establish process-distributed TCP behavior, not separate-host performance or high availability.

## 2. Prerequisites and immutable inputs

Performance work begins after required correctness/resource checks and final-model evaluation. Retain the reviewed commit plus uncommitted diff identity; source modifications used in an experiment must be identifiable even if the user has not committed them. Never create commits to satisfy this bookkeeping without instruction.

Each experiment manifest records:

- Experiment ID, date, purpose and code-state identity.
- Linux/kernel/WSL status, CPU model, logical/physical core information, available memory, compiler/version/build flags and dependency versions.
- Actual allowed CPU resource budget, worker count, application thread counts, ONNX intra/inter-op settings, execution provider and any affinity settings.
- Model/ONNX/preprocessing/bundle hashes, fixed threshold and model input/output shape.
- Unique image IDs/content hashes, order, count, byte/dimension distribution and provenance. Label generated/transformed workloads and keep them separate from held-out ML evaluation.
- All operational configuration, including queue limits, timeouts, attempts, SQLite durability and logging settings.
- Warm-up, measurement boundaries, repetitions, failure accounting, memory collection method and profiler limitations.

Record the actual environment rather than copying earlier preliminary CPU/RAM observations. Pin a fixed workload with at least 100 unique completion samples per run. Repeating the same image through retries must not inflate throughput or sample counts. Use the same content and total CPU resource allowance in a baseline/candidate comparison.

## 3. Baseline and repetition protocol

Use the Release build with the frozen final bundle. Perform 20 warm-up model inferences before each measured run and five measured repetitions per declared configuration. Warm-up calls/results are not job completions. Keep the fixed workload order identical across configurations.

Start with an explicitly recorded reference configuration. Record separate one-worker and two-worker cases if investigating concurrency; they use the same machine and declared CPU budget. A second process can add overhead or compete for CPU, so speedup is an experimental result. Extra-host work needs its own scale-out label and is outside the mandatory budget.

Do not weaken input validation, retry rules, logging boundaries or SQLite durability to make a candidate appear faster. Report any changed operational setting and hold unrelated controls fixed.

For every run retain raw per-task/process data, unique success/failure counts, retries, elapsed time and warm-up confirmation. Summarize each run before reporting the median and range across all five. Preserve failures/outliers with explanations; never present only the best run.

## 4. Metric definitions and arithmetic

| Metric | Computation and boundary |
|---|---|
| Model-only throughput | Successful batch-one ONNX calls on prepared tensors divided by their measured elapsed execution time after warm-up; excludes network, decode and queueing |
| Pipeline throughput | Unique successful tasks committed divided by time from workload release to its final terminal outcome; failed tasks remain in outcome accounting and retries do not add successes |
| Task p95 | In no-failure runs, nearest-rank 95th percentile of successful tasks' admission-to-durable-commit durations; includes queueing, transfer, preprocessing and inference; report population and failures |
| Stage timing | Worker preprocessing/inference and transport/commit observations with clear clock ownership; do not subtract timestamps from unrelated hosts or runs |
| Memory | Each worker/coordinator's peak RSS with units/method, plus owned-buffer high-water counters; sampled maxima are labeled as sampled rather than exact kernel high-water values |
| Recovery | Fault-harness monotonic timestamp at kill to the interrupted task's successful durable commit; detection and reassignment are separate measurements |
| Precision/recall | Confusion counts from the untouched official test partition at the validation-selected frozen threshold; undefined denominators are explicit |

Report model loading, import, rendering and whole-command wall time separately so excluded costs remain visible. Independent per-process peaks cannot be summed and called a simultaneous system peak. Buffer limits do not bound library allocations or total process memory.

T065/T066 validate calculations on known fixtures: repeated attempts cannot add unique successes, injected queue delay increases full latency, nearest-rank indexing matches hand calculations, units agree, and memory measurements refer to the intended process.

## 5. Profiling and one bounded optimization

Run the baseline before selecting an optimization. Check actual Linux/WSL profiling facilities and permissions. If hardware events are unavailable, use ONNX Runtime profiling and instrumented stage timings and state the resulting limits. Installing a profiler or changing global/kernel settings requires permission.

Record a hypothesis before changing code/configuration: measured evidence, suspected cause, intended effect, fixed controls, and affected correctness checks.

Preferred experiment: configure ONNX thread counts and worker concurrency deliberately to avoid oversubscription. A small reusable-input-buffer change is another option only if allocation evidence supports it. Batching and dedicated pools are stretch goals, not assumed work in this allocation.

Check candidate correctness before timing. Repeat the same five-run protocol and compare all results. Throughput improvement uses `(candidate - baseline) / baseline * 100`; latency/memory reduction uses `(baseline - candidate) / baseline * 100`, naming the exact metric/summary and baseline. If the denominator is zero or runs are incomparable, report no percentage.

An unsuccessful attempt is still evidence. Keep the working configuration and describe what the experiment established without inventing a gain.

## 6. Proposed benchmark CLI recipes

These commands do not exist yet. T064 creates `python/tools/benchmark.py` and its configuration; later tasks add operations. These are command contracts to verify during human implementation, not current runnable setup or results.

Each suffix follows `PYTHONPATH=python .venv/bin/python python/tools/benchmark.py`:

| V-BENCH recipe | Producing task | Proposed suffix | Required output |
|---|---|---|---|
| manifest | T064 | `manifest --config artifacts/config/performance.json` | Validated workload/environment/control manifest |
| memory-check | T066 | `memory-check --config artifacts/config/performance.json` | Known-allocation unit/process/method evidence |
| baseline | T067 | `run --config artifacts/config/performance.json --variant baseline` | Five measured runs after warm-up for each declared reference configuration |
| profile | T068 | `profile --config artifacts/config/performance.json --variant baseline` | Supported profiler/stage observations and limitations |
| candidate | T069 | `run --config artifacts/config/performance.json --variant candidate` | Candidate correctness context and exploratory measured attempt |
| compare | T070 | `compare --config artifacts/config/performance.json` | Same-control five-run comparison; run missing measurements rather than invent them |

T064's configuration captures native executable paths, actual workload/model locations, worker/ONNX settings and output directory. The launcher reuses the owned-process harness from T045, records its child processes, and cleans up only those processes. No string-built shell invocation, public listener or unrelated process termination.

A valid five-run candidate set from T069 can serve T070 if code/configuration/controls are unchanged. Do not repeat runs solely for repetition; a relevant change requires a new affected set, retaining earlier evidence separately.

## 7. Evidence layout

Generated evidence lives under `artifacts/evidence/<experiment_id>/` and is excluded from Git. Retain:

| File or directory | Content |
|---|---|
| `manifest.json` | Workload/artifact/code-state identities, controls and boundaries |
| `environment.json` | Actual hardware/OS/build/dependency/profile-tool observations |
| `config.json` | Fully resolved operational and measurement settings |
| `runs/` | Per-run timings/events, unique outcomes, memory and failure logs |
| `profile/` | Raw profiler output and interpretation, if available |
| `summary.json` | Recomputable statistics and limitations |
| `predictions.csv` | For ML: image ID, truth, score, decision and frozen threshold |

Validation, held-out evaluation, parity, recovery and performance comparisons use distinct IDs. The final index links actual paths and hashes. Small sanitized summaries may be kept in the repository; restricted images, weights or secrets are not public attachments.

## 8. Future CV claim register

| Claim category | Evidence required before describing it as achieved |
|---|---|
| ML defect detection | Held-out predictions/counts, frozen artifact/threshold, split provenance and error analysis |
| C++ performance | Repeated throughput/p95, explicit pipeline/model boundaries, machine and total CPU budget |
| Memory/concurrency | Queue/ownership checks, caps and per-process memory; reduction only with a comparable baseline |
| Worker fault tolerance | Repeated real-model crash traces ending in successful interrupted-task commitment, bounded attempts and no duplicate committed result |

No results are entered yet. Do not fill in a percentage, latency, memory value or recovery time until the corresponding task has evidence. Label same-host WSL experiments accurately; do not claim separate hosts, production adoption or exactly-once execution.
