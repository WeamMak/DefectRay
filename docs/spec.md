# DefectRay — Distributed AI Visual Inspection Engine

Status: **APPROVED BY THE USER ON 2026-10-25**

Draft date: 2026-10-25

Architecture direction and this written specification were approved in conversation on 2026-10-25. The user's explicit specification approval was: "spec approve". The implementation plan and companion procedures were separately approved on 2026-11-05. No implementation task is authorized to start automatically; current task state is recorded in docs/plan.md.

## Read this first

DefectRay inspects a collection of industrial surface images using one CPU model, distributes images over TCP to two C++ workers, retains recoverable progress, and produces JSON and HTML inspection reports.

- You implement. The assistant plans, explains, and reviews; fixes require an explicit review-and-fix request.
- Budget: approximately 160 focused hours, including learning, debugging, tests, reviews, and documentation. Protect 18 hours for performance work and 20 hours for contingency.
- Keep one active job per coordinator, one in-flight image per worker, batch size one, and one model/product category. Two workers run concurrently on the same Linux/WSL machine for mandatory demonstrations.
- Mandatory: C++ ownership and threading, bounded memory accounting, real TCP, durable scheduling, retries and deduplication, PyTorch adaptation, ONNX/C++ inference, reports, automated tests, and measured evidence.
- Stretch only: dynamic batching, custom buffer pools, Docker, Prometheus/Grafana/alerts, separate-host/cloud deployment, and Terraform.
- Model-weight permission and exact dependency compatibility remain explicit acquisition gates, not verified implementation facts. See section 8.
- Specification, plan and companion procedures are approved. Follow the requested-task implementation/review/acceptance workflow in `docs/plan.md`.

No project code, dependency installation, model execution, or acceptance tests have been performed by writing this document. Git initialization already performed by the user is a prerequisite, not completion of T001.

## 1. Scope and learning contract

**SCOPE-001 — Deliverable.** Provide a reproducible command-line workflow from an input directory through two TCP-connected workers to a report of suspected defects and processing failures. A suspected defect is a successful inference outcome, not a runtime error. Results are for review, not certified industrial decisions.

**SCOPE-002 — Environment.** Implement coordinator, scheduler, protocol, and worker runtime in C++20 on Ubuntu in WSL2 or native Linux. Use CMake and a project-local Python environment. Keep compilers, libraries, virtual environments, and build outputs inside Linux. GPU execution is not required.

**SCOPE-003 — Human implementation.** Tasks follow TODO → IN_PROGRESS → IN_REVIEW → READY_FOR_ACCEPTANCE → DONE, with BLOCKED available. Required checks must actually pass before readiness. Only explicit user acceptance authorizes DONE; acceptance never automatically starts the next task.

**SCOPE-004 — Boundaries.** No LLMs, agents, MCP, distributed training, arbitrary model/device support, large frontend, replicated coordinator, consensus, authentication service, or public network deployment. The initial trust boundary is loopback; explicitly configured isolated/private networks are supported without a claim of secure public operation.

**SCOPE-005 — Budget control.** Treat 160 hours as a spending limit, not a guaranteed completion forecast. Track actual learning, implementation, review correction, test, and documentation time in the future plan. Use checkpoint reviews to spend contingency deliberately. Do not hide overruns, weaken mandatory checks, or claim completion at the time limit if required work remains.

## 2. Product behavior and identifiers

**PROD-001 — Submission.** A submission enumerates regular `.png` files in one directory, without recursive traversal or following symbolic links. Ignore unrelated extensions with a visible count. Reject an empty collection, an existing job destination, invalid configuration, or an unreadable source before acceptance. Initial limits are 10,000 candidate files and 2 GiB of accepted unique encoded images. Reject over-limit submissions as a whole; do not silently truncate them.

**PROD-002 — Durable inputs.** Copy accepted source bytes into a new job directory, one image at a time, and retain them until explicit user removal. Record original relative names, content hashes, and copy outcomes. Deduplicate byte-identical files using the image hash and list their names as aliases; create one task per unique image. An encoded file over the 8 MiB limit rejects the whole submission before acceptance. Invalid-header or unsupported PNG candidates within that encoded limit receive terminal input-error records; their retained bytes count toward the import cap. Do not discard admitted candidates without an outcome.

**PROD-003 — Acceptance boundary.** Mark a job accepted only after its manifest and managed image copies are complete and its initial task records are committed. Interrupted import leaves an unaccepted directory with a diagnostic; resuming import is outside mandatory scope. Do not automatically delete it. Once accepted, recovery does not depend on original source files. Managed inputs are immutable by contract; verify their hashes when reading for dispatch and fail changed/missing inputs explicitly.

**PROD-004 — CLI contract.** The following describes future interfaces, not commands that currently exist or have been tested:

| Interface | Behavior |
|---|---|
| `defectray submit --input DIR --job-dir DIR --model-bundle DIR` | Import a job, then run its coordinator in the foreground |
| `defectray resume --job-dir DIR` | Recover and continue the same accepted job |
| `defectray worker --id NAME --connect HOST:PORT --model-bundle DIR` | Run one worker process; repeat with a different ID for the second worker |
| `defectray status --job-dir DIR` | Read a consistent progress snapshot from local durable state |
| `defectray report --job-dir DIR` | Export a fresh JSON snapshot and invoke the small Python HTML renderer |

Submission and resume accept an explicit listen endpoint; the default is `127.0.0.1:47070`. A busy port is an error, not permission to terminate another process. CLI status/report access is local to the coordinator filesystem. There is no separate remote submission server. The report renderer must receive arguments without shell interpolation.

**PROD-005 — Configuration.** Version the resolved configuration and store it with the job. It includes endpoint, limits, heartbeat/deadline settings, attempt limit, model manifest, and runtime thread settings. CLI flags override documented defaults; unsupported values fail before dispatch. Resume may change operational endpoints and timing/thread settings, recording the new run configuration, but cannot change job inputs, model, preprocessing, or decision threshold.

**PROD-006 — Identity.** Use a collision-checked random 128-bit job ID; SHA-256 of original encoded bytes as image ID; `(job_id, image_id)` as task identity; and a monotonically increasing persisted attempt number per task. `worker_id` is an explicit name; each worker process also has a fresh random session ID. The coordinator increments a durable run epoch on each start/resume. `model_id` is the ONNX file SHA-256; `preprocessing_id` is the SHA-256 of the exact versioned preprocessing file. Hash the exact model-bundle manifest to bind model, preprocessing, labels, and threshold together. Use an existing hashing implementation rather than writing cryptography.

**PROD-007 — Job outcomes.** Jobs move from IMPORTING to RUNNING. Graceful interruption or exhaustion of the no-worker waiting deadline yields PAUSED when durable state can be updated. Resume returns PAUSED/interrupted RUNNING jobs to RUNNING after recovery. With no unfinished tasks, derive one terminal outcome:

| Outcome | Meaning |
|---|---|
| SUCCEEDED | Every admitted unique image was processed successfully, whether flagged or not |
| PARTIAL_FAILED | At least one successful inference and at least one terminal processing/input failure |
| FAILED | No successful inference and all admitted tasks are terminal failures |

A progress snapshot with unfinished tasks is explicitly partial, never a terminal success. Invariants: `total = pending + active + succeeded + failed`; `flagged <= succeeded`. Aliases do not increase the unique-image count. A no-worker pause leaves unattempted work pending rather than inventing image failures.

**PROD-008 — Errors and exit status.** Errors include a stable category, relevant identifiers, and a useful message without sensitive data. Exit codes: 0 for a successful command/job, 2 for usage or pre-acceptance validation errors, 3 for PARTIAL_FAILED, 4 for FAILED, 5 for an operational/reporting failure or no-worker pause, and 130 for SIGINT. Failure to generate a report does not undo committed inference outcomes; report generation is retryable. Infrastructure errors stop new dispatch and preserve recoverable state where storage remains usable.

**PROD-009 — Reports.** JSON schema version 1 contains job/run IDs, configuration/artifact identities, progress and outcome counts, input aliases, and each task's score/flag or error, committed attempt, retry count, and available timing fields. HTML contains a summary, flagged-image previews or links, scores and threshold, and clearly separated processing failures. Link to managed images using generated safe relative paths; escape original filenames and error text. A paused-job report labels unfinished tasks. Report generation uses a consistent snapshot and streams rows instead of retaining all images. Write temporary output and rename on successful completion; a report can be regenerated from the database.

**PROD-010 — Artifact layout.** An accepted job directory contains `state.sqlite`, an immutable `manifest.json`, managed `inputs/<image_id>.png`, structured `events.jsonl`, and generated `results.json`/`report.html`. SQLite is authoritative for changing task state; the manifest describes immutable inputs and bundle identity. A model bundle contains `model.onnx`, `preprocessing.json`, and a manifest recording their hashes, input/output shapes, label meaning, threshold, dependency versions, and training/split provenance. Generated jobs, datasets, weights, and full evidence outputs are excluded from Git; retain small summaries and acquisition/reproduction instructions in the repository.

## 3. Architecture and responsibility boundaries

**ARCH-001 — Modules.** Keep concrete modules with narrow interfaces; no plugin system or generalized distributed-computing framework.

| Module | Owns | Depends on |
|---|---|---|
| CLI/job importer | Argument validation, managed input manifest, process entry points | Filesystem, hashing, job store |
| Coordinator/scheduler | Worker availability, task selection, deadlines, authoritative transitions | Transport events, job store |
| Job store | SQLite schema, transactions, conditional result commits, snapshots | SQLite |
| Framed connection | Socket lifecycle, parser/writer state, size/deadline checks | Standalone Asio, JSON library |
| Worker runtime | One-slot handoff queue, inference thread, completion handoff, shutdown | Transport, inference module |
| Inference module | PNG validation/decode, preprocessing, ONNX session, score checks | OpenCV, ONNX Runtime |
| ML tools | Split manifests, adaptation, evaluation, export, reference fixtures | PyTorch/TorchVision, ONNX, Python ONNX Runtime, OpenCV |
| Report renderer | Static escaped HTML from exported JSON | Python standard library |

**ARCH-002 — What you implement.** Scheduling, framing and message validation, connection state, buffer lifetimes, bounded queues/backpressure, thread synchronization, leases/retries, stale-result handling, deduplication, and the failure harness remain human-implemented learning work. Libraries supply socket readiness/timers, serialization primitives, image codecs/resizing, inference kernels, hashing, and database transactions.

**ARCH-003 — Process model.** The foreground CLI is also the coordinator. One coordinator owns one job directory and admits at most two worker sessions. A process-held exclusive lock prevents concurrent coordinators for that directory; its crash must release the lock automatically. Each worker owns one network event loop and one inference thread. One outstanding attempt per worker includes receiving, queued, running, and result-awaiting-acknowledgment states. The surviving worker can finish a job alone after failure.

**ARCH-004 — Transport choice.** Use standalone Asio for asynchronous sockets and timers, with explicit read/write state. POSIX sockets plus `poll()` would expose more Linux readiness mechanics but require more supporting code; a custom `epoll` reactor is outside this budget. Asio does not replace protocol, scheduling, ownership, or recovery work. Its documentation describes asynchronous composition and lifetime tradeoffs [S1].

**ARCH-005 — Data flow.** Import → durable PENDING task → committed assignment → encoded image bytes over TCP → worker decode/preprocess/infer → result → conditional durable commit → acknowledgment → next assignment. Model bundles are provisioned locally on both workers; registration rejects mismatched bundle IDs. Neither image nor model paths are assumed to exist on another machine.

## 4. Ownership, concurrency, and limits

**CPP-001 — Ownership.** Use RAII for sockets, files, SQLite handles/statements/transactions, locks, threads, and ONNX resources. Buffers have an identifiable owner; non-owning views cannot outlive that owner. Pending asynchronous operations retain their buffers until completion, including cancellation completion. Move ownership across thread boundaries where appropriate. Sharing ownership requires an actual shared lifetime, not a blanket replacement for reasoning about cleanup.

**CPP-002 — Thread boundaries.** Coordinator scheduling and database mutation run on one coordinator thread. Worker connection state is confined to its network thread. The inference thread consumes one bounded request slot using a mutex and condition variable; its completion is posted back to the network thread. At most one inference completion is outstanding. Condition waits use predicates and respond to queue closure. Inference never blocks the network event loop, so health messages remain possible while the model runs.

**CPP-003 — Application limits.** Enforce limits before allocation/dispatch and account for staging copies as well as queued and active buffers:

| Resource | Initial default and behavior |
|---|---|
| Encoded PNG | At most 8 MiB per image |
| PNG dimensions | Each dimension at most 4096; at most 4,194,304 pixels |
| PNG representation | 8-bit grayscale or RGB, non-animated; reject unsupported forms explicitly |
| Metadata/control JSON | At most 16 KiB |
| Worker request handoff | One slot; at most one total outstanding attempt |
| Worker decoded RGB buffer | At most 12 MiB under the pixel cap |
| Per-worker owned bulk buffers | At most 64 MiB across encoded, decoded, resize, tensor, and transport buffers |
| Coordinator owned image/transport buffers | At most 64 MiB total across both connections and input staging |
| Per-connection output | At most one image frame plus four small control frames, including the active write |

Byte accounting covers live application-owned bulk allocations, including vector capacity and retained asynchronous buffers. Inspect PNG header dimensions before invoking full decode; validate decoded dimensions again. Coalesce unsent heartbeats rather than queueing them indefinitely. Backpressure stops dispatch/reads before capacity is exhausted. Overflow or an unexpected assignment while busy closes the offending connection and produces a diagnostic; do not silently drop assigned work.

**CPP-004 — Honest memory claims.** These limits do not bound ONNX Runtime/OpenCV internal allocations, SQLite caches, allocators, thread stacks, kernel socket buffers, or total RSS. Measure peak RSS separately. A native decoder/inference allocation failure may terminate a worker; the coordinator must still recover its task. Bulk-buffer accounting must be exposed in structured logs for tests and measurements.

**CPP-005 — Inference threads.** Start with sequential ONNX graph execution and one explicitly configured intra-op thread per worker. Record all runtime settings. Change this setting only through recorded configuration for measurements; ONNX Runtime can create internal thread pools, so worker count alone does not describe CPU concurrency [S2]. No mandatory dynamic batching, generic thread pool, or custom allocator.

**CPP-006 — Shutdown.** On graceful stop, cease dispatch, cancel timers/socket operations, close the handoff queue, wake blocked waits, and join owned threads before releasing their resources. Do not detach inference threads or free their inputs while a call is running. Finish an active native inference call before joining; forced cancellation inside ONNX is not required. If a native call hangs, an external controlled termination may be necessary; there is no bounded graceful-shutdown claim for that case. Normal shutdown with a test inference bounded to two seconds must exit within five seconds without leaked application descriptors or deadlock.

## 5. TCP protocol and connection handling

**NET-001 — Frame format.** Version 1 uses a 16-byte header: 4 ASCII magic bytes `DFRY`, unsigned 16-bit version, unsigned 16-bit type, unsigned 32-bit metadata length, unsigned 32-bit binary length. Integers use network byte order. UTF-8 JSON metadata follows, then optional raw binary bytes. Only ASSIGN carries binary PNG bytes. Header, type, size, arithmetic-overflow, and per-message field checks precede body allocation. No C++ struct memory is sent directly.

**NET-002 — Message vocabulary.** Use HELLO, WELCOME, READY, ASSIGN, HEARTBEAT, RESULT, ACK, and ERROR. HELLO identifies worker/process session and bundle; WELCOME supplies coordinator epoch and timing settings. READY advertises capacity for one assignment. ASSIGN identifies job/image/task/attempt/epoch and encoded-image digest. RESULT repeats these identities and contains either one finite score in [0,1] with stage durations or a categorized processing error. ACK identifies the disposition: committed, already_committed, stale, or rejected. A worker sends READY only after the previous attempt has been acknowledged or abandoned during reconnection. HEARTBEAT identifies the process session and current attempt, if any.

**NET-003 — Stream correctness.** Parsing must tolerate a frame split across arbitrary reads and multiple frames in one read. Writes must handle partial progress without interleaving frames. Permit at most one read and one serialized write operation per connection; multiple logical messages share the bounded writer. Treat EOF, reset, cancellation, malformed JSON, missing/wrong-typed required fields, unsupported version/type, digest mismatch, and unexpected state transitions explicitly. Unknown optional metadata fields may be ignored in version 1; unsupported versions are rejected.

**NET-004 — Timing defaults.** Use monotonic clocks for running deadlines: connect/registration 5 s, full-frame receive/write 5 s from its first byte/start, heartbeat every 2 s, worker unavailable after 10 s without valid health traffic, attempt deadline 30 s from durable assignment, and no-compatible-worker pause after 30 s. Partial-byte trickling does not extend the whole-frame deadline. Reconnect with delays 1, 2, 4, then at most 5 s until stop; never busy-loop. Timers and a stalled connection must not prevent serving the other worker.

**NET-005 — Sessions and reconnect.** Reject duplicate live worker names and excess sessions. Expire/disconnect the previous session before accepting a replacement with the same name. Registration checks the exact bundle before READY. A disconnect abandons that session's assignment; retries use a new attempt. On reconnect, the worker discards any result belonging to the old session and finishes/discards any old native inference before advertising capacity. Connection success alone is not proof that an interrupted task recovered.

## 6. Scheduling, durability, and recovery

**DIST-001 — Task state machine.** Tasks start PENDING, become ACTIVE when an assignment is committed, and become SUCCEEDED or FAILED on a committed terminal outcome. A retryable loss returns ACTIVE to PENDING if attempts remain. An invalid admitted input may transition directly to FAILED without a worker attempt. Terminal tasks never return to PENDING during ordinary resume.

**DIST-002 — Attempt state machine.** Each attempt is ASSIGNED, then exactly one of SUCCEEDED, FAILED, LOST, TIMED_OUT, or ABANDONED_ON_RESTART. Commit the new attempt, increment its count, and set ACTIVE atomically before send. Failure before send still consumes that attempt; this simple conservative rule must be tested. The default maximum is three total attempts, not three retries. Input decode/format errors are permanent; disconnects, expired leases, or native-worker loss are retryable. Reject invalid model output and record a permanent task error rather than silently accepting it.

**DIST-003 — Selection and capacity.** Assign the oldest PENDING task to a compatible READY worker, breaking ties deterministically by task ID. Keep scheduling demand-driven; load image bytes only after selecting a task for an available worker. Page task metadata from the store instead of loading the complete dataset into memory. Do not hold a SQLite transaction open while transferring bytes or running inference.

**DIST-004 — Leases and failure detection.** An attempt's lease is the fixed assignment deadline. Heartbeats report process health but do not renew this deadline. On disconnect or health expiry, record LOST; on attempt deadline expiry, record TIMED_OUT and close that session so it cannot be assigned another task while still computing. Retry if allowed, otherwise commit terminal failure. Persist attempt counts across restarts.

**DIST-005 — Result commitment.** The coordinator validates task, attempt, epoch, worker session, model bundle, and lease validity in its serialized state owner. Only the current live attempt can commit. Write the result and terminal task transition in one SQLite transaction with a unique task-result constraint; send committed ACK only after transaction success. An identical retransmission of a committed result receives already_committed without changing counters; a conflicting retransmission is rejected. Older attempts never overwrite the authoritative outcome. SQLite supplies transaction atomicity, not these application rules [S3].

**DIST-006 — Restart.** Acquire the exclusive job lock, verify database schema/configuration and bundle identities, increment the run epoch, and transactionally mark unfinished old attempts ABANDONED_ON_RESTART. Requeue tasks with remaining attempts; mark exhausted ones FAILED. Do not compare a persisted monotonic deadline with a clock from a new process/boot. Old results cannot match the new epoch. Already committed outcomes remain intact even if their ACK was lost before the crash. Workers reconnect to the resumed coordinator.

**DIST-007 — Storage guarantees.** Use SQLite transactions with foreign keys and uniqueness constraints enabled; keep rollback journaling and synchronous FULL rather than disabling durability for benchmark speed. Store the database on local Linux storage. Recovery acceptance covers coordinator/worker process termination with the job directory intact. Disk loss, filesystem corruption, power-loss durability of image imports, replicated availability, and live migrations are excluded. The import/database boundary must never advertise an accepted job that references an incompletely copied image.

**DIST-008 — Execution semantics.** An image can execute again after an uncertain result, lease expiry, or restart. Claim at most one committed result per task, not exactly-once execution. A returning worker can rejoin, but automatic spawning/restarting of worker processes is outside the coordinator's responsibility.

## 7. Model and preprocessing contract

**ML-001 — Dataset.** Use one industrial surface-inspection dataset, with KolektorSDD2 as the selected dataset direction. Its official page lists 3,335 images, labeled defective/nondefective train/test partitions, and CC BY-NC-SA 4.0 terms [S4]. The public download entry point and stated license were checked on 2026-10-25; no archive was downloaded or inspected. Verify the archive layout, labels, integrity, access terms, and reproducible acquisition steps during the authorized acquisition task. Derive image-level labels from official annotations, never from model predictions. Do not redistribute dataset images in Git or evidence bundles without satisfying their terms.

**ML-002 — Adaptation.** Candidate: TorchVision MobileNetV3-Small with a binary classification head, frozen feature extractor, and CPU training of the head. Keep the frozen backbone in evaluation mode so its batch-normalization state does not drift. Export the complete feature-extractor/head graph, including sigmoid, as one model. The artifact choice is conditional on GATE-002; no pretrained weights have been approved for acquisition. Full-backbone fine-tuning and architecture searches are not mandatory. A replacement model requires an explicit recorded decision rather than an unnoticed dependency change.

**ML-003 — Splits and threshold.** Preserve the official test partition. Before training, create a reproducible 80/20 stratified training/validation split of the official training partition with seed 42; keep related views of the same item in one partition where identifiers permit. Check exact duplicates across partitions and disclose limitations if independent physical-item grouping cannot be established. Fit training transformations/statistics on training data only. Select the threshold by maximum validation F1, breaking ties by greater recall and then lower threshold. Freeze model, preprocessing, threshold, and selection procedure before held-out evaluation. Never tune on held-out results.

**ML-004 — Quality evidence.** Save confusion counts, precision, recall, F1, class counts, false-positive examples, and missed-defect examples. Treat zero-denominator metrics explicitly rather than fabricating values. Report the validation comparison against always-normal and always-flagged baselines. The validation quality gate requires positive precision and recall and F1 better than the always-flagged baseline; failure triggers review within the ML/contingency budget, not a promise of a particular test-set accuracy. Held-out precision/recall have no invented minimum or guaranteed improvement. Label a sigmoid output a defect score; calibration is not claimed.

**ML-005 — Preprocessing v1.** Decode to RGB uint8; grayscale inputs repeat their channel. Preserve the full image: resize with bilinear interpolation to fit width 256 and height 640, preserving aspect ratio; round each scaled dimension with `floor(value + 0.5)` and clamp to at least 1. Center on a black RGB 256×640 canvas, placing any odd extra padding on the bottom/right. Convert to float32 in [0,1], normalize by RGB means [0.485, 0.456, 0.406] and standard deviations [0.229, 0.224, 0.225], and produce contiguous NCHW shape [1,3,640,256]. Use the same documented OpenCV decode/resize recipe in Python and C++. No default center crop that discards the image surface. Record this recipe in a versioned file; changing it changes preprocessing/bundle identity and requires re-evaluation.

**ML-006 — Export and runtime checks.** Export a fixed batch-one float32 graph with one input of the declared shape and one sigmoid score of shape [1,1]. Pin ONNX opset and IR versions compatible with the selected runtime; run the ONNX checker. Compare PyTorch, Python ONNX Runtime, and C++ ONNX Runtime on reference fixtures. Require tensor max absolute error ≤ 1e-5 and score `abs(actual-reference) ≤ 1e-4 + 1e-4*abs(reference)`. Require matching decisions for reference scores more than 1e-3 away from the frozen threshold; count/report near-threshold cases separately. The decision rule is score ≥ threshold. Reject wrong shapes/dtypes and NaN/infinite/out-of-range output before committing a result. Do not relax tolerances solely to pass a broken preprocessing implementation. Export/runtime compatibility is an explicit check, not implied by successful package installation [S5, S6].

**ML-007 — Early feasibility.** Before complex scheduling/recovery, demonstrate a real image through the candidate PyTorch graph, ONNX export, and C++ CPU inference, plus at least eight deterministic preprocessing/parity fixtures covering grayscale/RGB, padding, aspect ratios, and range extremes. This establishes interface feasibility, not held-out model quality. Persist inputs/hashes, expected shapes/scores, versions, actual commands, and comparison results. A deliberately untrained adaptation head may prove export mechanics if clearly labeled; it cannot satisfy ML evaluation or final end-to-end acceptance.

## 8. Acquisition and compatibility gates

**GATE-001 — Python/toolchain selection.** Before selecting the project Python version or installing ML dependencies, record a compatibility matrix for Python, torch, torchvision, onnx, onnxruntime, exporter dependencies (including onnxscript if required), NumPy, and Python/C++ OpenCV. Check supported Python ranges, torch/torchvision pairing, Linux x86-64 CPU wheel availability, C++ ONNX Runtime ABI/build prerequisites, and exported opset/IR support. Use release-specific official metadata, not only a moving latest-version page [S6, S7]. The system Python 3.14 observation does not select the project version. Exact versions intentionally remain unselected until this gate; pin the verified set in the authorized infrastructure task and validate it at ML-007.

**GATE-002 — Pretrained weights.** The model candidate is not a finding that its weights are freely redistributable. TorchVision warns that pretrained artifacts can carry terms derived from training data [S8]. Before downloading or using the selected weights, record the exact artifact/source, applicable terms, attribution and redistribution constraints, and whether educational use is permitted. If primary documentation does not establish permission, mark acquisition BLOCKED and ask for a model decision; do not assume the library's code license covers its weights or silently train a different model. Do not require downloading ImageNet itself.

**GATE-003 — Scope of approval.** Approval of this specification may approve the design and these gated decision procedures, but does not certify unverified dependency compatibility or license permission. Acquisition/install actions still follow the user's explicit authorization and environment permissions. Planning must expose these gates as early dependencies with small timeboxes, not postpone them until after networking work. Record resolved versions/artifacts and their evidence in the canonical documents before dependent work.

## 9. Verification and completion criteria

**TEST-001 — Unit coverage.** Verify queue capacity/closure and predicate waits, framing across fragmented/coalesced input, length/overflow checks, legal/illegal state transitions, retry limits, conditional commit/deduplication, preprocessing, output validation, and report escaping/counts. Use behavioral tests; do not mirror implementation internals merely to increase test count.

**TEST-002 — TCP integration.** Use real loopback sockets for registration, encoded-image transfer, malformed/oversized frames, byte fragmentation, partial writes, disconnects, stalled traffic, and reconnection. Cover two workers concurrently and verify that one stalled connection does not stop the healthy worker. A mocked transport alone does not establish networking acceptance.

**TEST-003 — End-to-end.** A controlled collection containing valid images, a duplicate, and one corrupt PNG must produce exactly one terminal task outcome per admitted unique image, correct alias/count accounting, and matching JSON/HTML totals. The real model must run on the two real worker processes. Include known labeled images for inspection of false positives/negatives; deterministic inference fixtures separately validate report and classification mechanics.

**TEST-004 — Recovery harness.** The harness launches and owns the coordinator and two workers, waits for an explicit inference-start event, and sends SIGKILL to the selected worker. Controlled blocking after that event makes the fault reproducible; production runs have no such delay. The surviving worker must commit that interrupted task successfully, finish all eligible work, and produce no duplicate committed results. Repeat five runs. With test lease 3 s, heartbeat interval 250 ms, heartbeat expiry 2 s, and healthy stub inference bounded to 1 s, the interrupted task must commit within 6 s of the kill. Also run five real-model crash trials, recording observed recovery times without substituting reassignment for completion. Restart a worker and demonstrate re-registration. Clean up only harness-owned processes and temporary data; never terminate unrelated processes.

**TEST-005 — Coordinator crash windows.** Inject termination after assignment commit/before send and after result commit/before ACK. Resume the same job, retain committed outcomes, invalidate old attempts, obey persistent attempt limits, and finish without duplicate results. Inject stale and conflicting result deliveries and verify rejection. Test all-workers-unavailable pause and resume. Database durability requires process-crash evidence; task requeueing alone is insufficient.

**TEST-006 — Resource checks.** Exercise queue saturation, maximum accepted payloads, decoder-limit rejection, disconnect/shutdown loops, and a collection of 1,000 distinct generated PNG fixtures. Assert the explicit owned-byte caps throughout. After warm-up, application-owned allocations and descriptors must return to their defined idle baseline after each completed attempt/connection cycle. Record RSS without asserting it equals owned bytes. Run separate ASan+UBSan and TSan builds for supported application code; document limitations for uninstrumented third-party libraries. Unavailable required checks block readiness until resolved or explicitly amended by the user.

**TEST-007 — Commands and evidence.** `docs/test-plan.md` will own actual configure/build/test/format/analysis/sanitizer/Python commands and fixtures once they exist. There are no verified build or test commands in this specification. Fast PR checks must not train models or download the full dataset. Use generated small fixtures for fast tests and an explicitly invoked model/end-to-end suite. Store reviewed commit/diff identity, commands, outcomes, and evidence paths in `docs/plan.md`. A later relevant change invalidates the affected earlier verification.

## 10. Measurement and CV evidence

**PERF-001 — Protected investigation.** Reserve 18 hours for establishing a baseline, profiling, attempting one evidence-driven optimization, repeating measurements, and reporting the outcome. Prefer tuning ONNX thread counts/worker concurrency under a fixed hardware budget; reduce allocations only if profiles support that hypothesis. Improvement, batching, and custom pools are not acceptance conditions. Record unsuccessful or regressive experiments honestly.

**PERF-002 — Metric boundaries.**

| Metric | Required definition |
|---|---|
| Model-only throughput/latency | Completed batch-one ONNX calls on prepared tensors after warm-up; excludes decode/network/queueing; never presented as full-system throughput |
| Pipeline throughput | Unique successful image tasks committed divided by elapsed processing time, from release of the admitted workload to its final terminal outcome; retries do not increase the numerator |
| p95 task latency | In no-failure runs, monotonic time from admission/release to durable result commit, including queueing, transfer, preprocessing, and inference; report successful-task population and failures separately |
| Peak memory | Worker and coordinator peak RSS reported individually, with units/method, alongside owned-buffer high-water marks; sum of independent per-process peaks is not called a simultaneous system peak |
| Recovery time | Harness monotonic time from fault injection to durable successful commitment of the specifically interrupted task; also report detection/reassignment separately |
| ML quality | Precision/recall and confusion counts on untouched official test data at the validation-selected frozen threshold |

Initial model loading, import, report rendering, and whole-command wall time are measured or disclosed separately. For resume experiments, report downtime and completion separately; do not subtract unrelated monotonic timestamps from different runs. Benchmark unique fixture IDs/content, not repeated attempts disguised as independent images.

**PERF-003 — Experimental controls.** Before comparison, record hardware and CPU budget, OS/kernel and WSL/native status, available memory, compiler/build flags, model/bundle hashes, CPU execution provider, thread counts, workload manifest/order, queue limits, warm-up, repetitions, and failure counts. Use the same model, preprocessing, workload, machine, and CPU resource budget for baseline and optimization. Measure one-worker and two-worker configurations separately; adding hosts is a different scale-out experiment. Fix 20 warm-up inferences and five measured repetitions per configuration, retaining per-image timings and per-run summaries, median, and range. Use enough unique images for at least 100 completion samples per run; record any generated or transformed workload as such and keep it separate from ML test evaluation.

**PERF-004 — Profiling evidence.** Validate profiler availability in the actual Linux/WSL environment. Prefer available CPU profiling tools; if hardware-event access is blocked, disclose it and use ONNX Runtime profiling and instrumented stage timings. Preserve raw observations and state what they can establish. Shared-runner CI timings are not authoritative performance benchmarks. `docs/benchmarks.md` will own reproducible experiment commands and evidence locations after implementation.

Retain each experiment under a generated `artifacts/evidence/<experiment_id>/` directory with workload/artifact manifest, environment and resolved configuration, code-state identity, raw event/timing or prediction files, profiler output when available, and a machine-readable summary. Reports or excerpts shared publicly must meet the underlying data/model terms; sensitive or unlicensed images are not evidence attachments.

**PERF-005 — Future CV claims.** Each claim must link to retained evidence:

| Intended claim | Required evidence |
|---|---|
| Automated defect detection with measured precision/recall | Split hashes, artifact/threshold, held-out predictions and confusion counts |
| C++ throughput and p95 latency | Defined boundaries, workload, hardware/thread budget, repeated raw timings |
| Bounded concurrency/memory usage | Ownership/queue tests, caps, RSS measurements, profiler observations; reduction percentage only with a comparable baseline |
| Distributed worker recovery | Fault-injection timestamps, interrupted task identity, successful commit, retry history, deduplication checks |

State two worker processes on one WSL/native host unless separate-host evidence exists. Do not claim production adoption, customer savings, exactly-once execution, or an improvement percentage unsupported by results.

## 11. Budget envelope and approval sequence

**BUDGET-001 — Allocation.** This is a design budget, not an executable task plan. No milestone is implemented, reviewed, or accepted merely by being listed here.

| Milestone | Hours | Acceptance boundary |
|---|---:|---|
| M1 Local infrastructure | 20 | Minimal Linux C++/Python build/test/debug workflow; formatting, analysis, separate sanitizers, fast CI, documented fresh setup |
| M2 Inference feasibility | 12 | ML-007 parity evidence; compatibility and acquisition gates resolved for the selected artifact |
| M3 First complete workflow | 24 | Actual model over two TCP workers to consistent JSON/basic HTML reports |
| M4 Durable scheduling/recovery | 26 | DIST requirements and worker/coordinator crash, retry, stale-result, and deduplication checks pass |
| M5 Model adaptation/evaluation | 16 | Frozen trained artifact, validation quality/threshold gate, held-out evaluation and error analysis |
| M6 Integration/robustness | 14 | Required unit/integration/end-to-end, saturation, shutdown, and sanitizer checks pass |
| M7 Performance investigation | 18 | Baseline, profile, attempted optimization, five-run comparisons and limitations |
| M8 Reproduction/presentation | 10 | Fresh-checkout reproduction, portable report/evidence bundle, repeatable demonstration |
| Debugging contingency | 20 | Unallocated reserve for mandatory-work overruns |
| **Total** | **160** | All time includes learning, debugging, testing, review corrections, and documentation |

Tests are developed alongside their components; M6 adds combined robustness verification. M3 can use an initial trained candidate before the final M5 evaluation, and its report must identify that candidate. A passing M3 does not establish final model quality. Target the first complete workflow around cumulative hour 56, subject to actual checkpoint evidence.

**BUDGET-002 — Small tasks.** After written-spec approval, use writing-plans to create mostly 1–3-hour tasks with one independently testable outcome, dependencies, requirement references, exclusions, acceptance criteria, and verification instructions grounded in actual configuration. Split tasks expected to exceed four hours. Include estimates per task and totals/actuals per milestone. Do not provide complete task implementations or auto-execute the plan.

**BUDGET-003 — First milestone.** T001 verifies and documents the existing Linux environment/toolchain. Separate later tasks cover Git hygiene/layout, generated-artifact ignores, C++20 CMake, debug/release builds, a test framework/smoke test, VS Code debugging, formatting, static analysis, separate sanitizer builds, a verified local Python environment/smoke test, minimal PR CI, and fresh-checkout instructions. Verify already working setup rather than repeating it. Existing Git initialization is not T001 acceptance.

**BUDGET-004 — Overrun policy.** Remove report styling/extra thumbnails, additional image formats, optional fine-tuning, extra profiling experiments beyond the first complete attempt, and all stretch goals before consuming mandatory evidence work. These extras are not prerequisites for any milestone. Preserve the real two-worker workflow, correct recovery, held-out evaluation, baseline measurements, and required tests. If the remaining mandatory work no longer fits, explicitly report the gap and request a scope/budget decision; do not silently extend or declare success.

**BUDGET-005 — Canonical records.** This file owns scope/architecture/requirements; the future plan owns tasks/status/reviews/acceptance; test-plan owns checks/fixtures/actual commands; benchmarks owns experiments/evidence; README owns setup/usage. AGENTS.md owns the working agreement. Do not create competing design documents. Written-spec approval precedes plan creation; plan approval precedes human implementation.

## 12. Risks and required early checks

| Risk | Required response |
|---|---|
| 160 hours is insufficient for first-time C++ and ML work | Track actual time, protect contingency, review after M1/M2/M3; never count unrun checks as passing |
| Pretrained-weight permission is unclear | Resolve GATE-002 before acquisition; request an explicit alternative if blocked |
| Python/package/ONNX/C++ binary incompatibility | Resolve GATE-001 and prove M2 before distributed complexity |
| Resizing hides small defects or frozen features are weak | Inspect validation errors and quality gate within the ML budget; no test-set tuning or guaranteed accuracy |
| CPU oversubscription creates misleading results | Explicit ORT threads, fixed CPU budget, repeat comparisons |
| File/database boundary or lost ACK loses progress | Test acceptance and both coordinator crash windows; result commit before ACK |
| Library allocations exceed application buffer accounting | Enforce input caps, measure RSS independently, demonstrate recovery from worker loss |
| Workspace configuration points at the old directory | Current repository was observed at `/home/weam/projects/DefectRay`; do not recreate the old path or alter `.git` to repair a tool view |

## 13. Primary-source references

Sources consulted on 2026-10-25. They support library/dataset facts, not proof that this project has implemented or measured the proposed behavior.

- [S1 — Asio asynchronous model](https://think-async.com/Asio/asio-1.36.0/doc/asio/overview/core/async.html)
- [S2 — ONNX Runtime thread management](https://onnxruntime.ai/docs/performance/tune-performance/threading.html)
- [S3 — SQLite atomic commit](https://www.sqlite.org/atomiccommit.html)
- [S4 — Official KolektorSDD2 dataset, split and license](https://www.vicos.si/resources/kolektorsdd2/)
- [S5 — PyTorch ONNX export documentation](https://docs.pytorch.org/docs/stable/onnx.html)
- [S6 — ONNX Runtime compatibility](https://onnxruntime.ai/docs/reference/compatibility.html)
- [S7 — TorchVision version/Python compatibility](https://github.com/pytorch/vision#installation)
- [S8 — TorchVision pretrained-model terms](https://docs.pytorch.org/vision/stable/models.html)

## Approval record

- 2026-10-25: User approved the proposed architecture direction in conversation.
- 2026-10-25: User explicitly approved the written specification: "spec approve".
- 2026-11-05: User approved the implementation plan, companion procedures and working agreement, and requested starting tasks and practice.
- Implementation task states and acceptance records: **see docs/plan.md; document approval does not imply task completion**.
