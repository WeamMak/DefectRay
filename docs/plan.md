# DefectRay implementation plan

Status: **APPROVED BY THE USER ON 2026-11-05 — HUMAN IMPLEMENTATION; T002 EXPLICITLY DELEGATED**

Date: 2026-10-25

> You implement one approved task at a time. The assistant briefs, reviews, and makes minimal corrections only on an explicit review-and-fix request. This plan contains outcomes and verification contracts, not complete implementations. There is no automatic execution handoff.

**Goal:** Deliver the inspection workflow and engineering evidence required by the [approved specification](spec.md) within the approximately 160-hour budget.

**Architecture:** One foreground C++20 CLI/coordinator, two TCP worker processes, one active job and one outstanding image per worker. SQLite retains authoritative state; PyTorch adaptation exports one CPU ONNX model; JSON and static HTML expose outcomes. Detailed architecture remains in spec.md.

**Stack:** Linux/WSL, C++20/CMake, standalone Asio, SQLite, OpenCV, Python/PyTorch/TorchVision, ONNX and ONNX Runtime. Exact dependency versions and pretrained artifact permission remain the early specification gates.

This applies the installed [writing-plans skill](../.agents/skills/writing-plans/SKILL.md), adapted to the user's explicit human implementation workflow: no complete solutions, automatic commits, worktrees, coding agents, or automated execution. Canonical plan location is this file.

## 1. Current approval and implementation state

- Architecture direction: approved on 2026-10-25.
- Written specification: explicitly approved on 2026-10-25; user message: “spec approve”.
- This implementation plan, companion verification/benchmark procedures, and working agreement: approved on 2026-11-05; user said “i approve all” and requested starting tasks and practice.
- T001: DONE after the user explicitly accepted it on 2026-11-05: “accept T001”. T002: READY_FOR_ACCEPTANCE after the user delegated its implementation and the assistant completed V-HYGIENE; T003–T076 remain TODO.
- Actual task time: T001 start reported as 17:17 on 2026-11-05 (Asia/Jerusalem); finish time and elapsed effort not yet reported. Estimates are not completed hours.
- Last accepted implementation task: T001 — Verify the existing Linux environment (2026-11-05). Git initialization remains a separate pre-planning prerequisite.
- Existing working-tree changes to AGENTS.md and spec.md are preserved. No application build/test configuration exists yet.

The planning documents introduce proposed paths/commands only. Do not run implementation commands during planning. Approval alone never starts a task; the user's 2026-11-05 request to start practicing authorizes the T001 briefing. The user performs the read-only checks; the assistant maintains progress/evidence records from actual results so documentation does not become a separate reading exercise. Code, tests and configuration remain human implementation work except where explicitly delegated. On 2026-11-05, the user delegated T002 only: “can you do it? i want to work on the real implementation”.

## 2. Global constraints and review contract

All requirements in spec.md apply. Particularly:

- One active job per coordinator; at most two workers; one outstanding attempt per worker; batch size one; CPU-only mandatory demonstration on a single Linux/WSL host.
- Real TCP image-byte transfer, bounded allocations/queues, explicit ownership and thread cleanup; use libraries for building blocks rather than outsourcing scheduler/protocol/recovery.
- Three total attempts by default; authoritative results committed before ACK; repeated execution is possible; at most one committed result per task.
- Fixed model/preprocessing/threshold within an accepted job; protected held-out test data; no fabricated metrics or promised improvement.
- 18 hours for benchmarking/profiling/one optimization attempt and 20 hours of unallocated contingency.
- No package/data/weight acquisition, global settings changes, commits, pushes, branches/worktrees or cloud actions without the necessary user instruction/permission.
- Required checks must actually pass at the reviewed code state; compilation or skipped/empty suites are insufficient. Recheck relevant later code changes.
- No future-task implementation during a review. Preserve unrelated staged/unstaged/untracked work.

The task state sequence is TODO → IN_PROGRESS → IN_REVIEW → READY_FOR_ACCEPTANCE → DONE. Use BLOCKED for an unmet dependency/required check; return to IN_PROGRESS when implementation is needed. Only explicit user acceptance authorizes DONE. Milestone acceptance can be recorded with acceptance of its final task; no second redundant approval loop is needed.

For each requested task: explain its purpose/interfaces/edge cases, let the human implement, run/review authorized checks, record findings/minimal requested fixes and actual evidence, then stop for acceptance. The next task is never started or briefed automatically.

## 3. Budget and checkpoint schedule

Hours include learning, human implementation, tests, debugging, review corrections and documentation. These are constrained estimates/timeboxes, not a promise that first-time work will finish that quickly.

| Milestone | Tasks | Hours | Acceptance boundary |
|---|---|---:|---|
| M1 Local infrastructure | T001–T016 | 20 | Minimal Linux build/test/debug, formatting, analysis, separate sanitizers, reproducible Python, and actual fast CI evidence. |
| M2 Inference feasibility | T017–T023 | 12 | Artifact permissions/compatibility resolved; real-image and fixture parity passes across PyTorch, Python ONNX and C++. |
| M3 First complete workflow | T024–T035 | 24 | CLI → two TCP workers → actual candidate inference → consistent JSON/basic HTML report. |
| M4 Durable scheduling and recovery | T036–T048 | 26 | Bounded retries, session/epoch fencing, worker and coordinator crash windows, and deduplication pass. |
| M5 Model adaptation and evaluation | T049–T056 | 16 | Validation quality gate, frozen final bundle, held-out evaluation and final-model end-to-end evidence. |
| M6 Integration and robustness | T057–T063 | 14 | Adversarial TCP, resource/cleanup limits, final error matrix and all required supported sanitizers/checks pass. |
| M7 Performance investigation | T064–T071 | 18 | Reproducible baseline, profiling, one attempted optimization, repeated comparison and honest evidence report. |
| M8 Reproduction and presentation | T072–T076 | 10 | Clean-source reproduction, demonstration, verified usage, traceable CV evidence and final acceptance packet. |
| Debugging contingency | Unallocated; no implementation task | 20 | Spend only against a recorded mandatory-work overrun |
| **Total** | **76 tasks + reserve** | **160** | Explicit user acceptance with required evidence |

Allocated task estimates sum to 140 hours. Cumulative targets are 20 / 32 / 56 / 82 / 98 / 112 / 130 / 140 hours before reserve. The first complete workflow is targeted at hour 56; final model evaluation happens later.

At the end of M1, M2 and M3, compare actual time and remaining work with the budget before proceeding at the user's request. Review both artifact feasibility gates before investing in distributed complexity. A blocked artifact decision never authorizes an undocumented model substitution.

No stretch work is funded in these 140 hours: Docker, dashboards/alerts, cloud/Terraform, dynamic batching and custom pools remain outside the plan. Extra report styling/formats/fine-tuning may be dropped before they consume time, but removing already-unfunded extras does not create additional hours. If mandatory work plus remaining checks no longer fits, explicitly present the gap and seek a scope/budget decision. Do not silently extend time, weaken tests or claim DONE.

## 4. Planned file and interface map

These are future human-created files/directories, not a scaffold created by the assistant. Each task names its concrete files; a new test also registers its name/labels in tests/CMakeLists.txt. Dependency wiring is added only when its first real consumer needs it.

| Area | Planned locations | Responsibility |
|---|---|---|
| Build/tooling | CMakeLists.txt, cmake/Dependencies.cmake, cmake/Sanitizers.cmake, .vscode/, .github/workflows/ci.yml | Reproducible Linux build/debug/check configuration |
| Common contracts | src/common/, src/config/ | Opaque IDs, validated settings, owned-byte accounting and structured events |
| Inference | src/inference/, tools/infer_one.cpp | Bundle validation, preprocessing, one native inference call |
| Networking | src/protocol/, src/net/ | Typed messages, bounded frame codec and asynchronous connection lifetime |
| Worker | src/worker/ | One-slot handoff, network/inference thread ownership and shutdown |
| Coordinator/storage | src/coordinator/, src/store/, src/job/ | Scheduling, transactional state, recovery, import and exclusive job ownership |
| Product commands/reports | src/cli/, src/main.cpp, src/report/, python/defectray_report/ | CLI wiring and consistent JSON/static HTML |
| ML tooling | python/defectray_ml/ | Acquisition-derived manifests, model adaptation, export and evaluation |
| Test/experiment support | tests/, python/tests/, python/defectray_tools/, python/tools/ | Behavioral fixtures, owned processes, parity, benchmarks and demo |
| Evidence/config outputs | data/, models/, artifacts/, build/, .venv/ | Generated/local outputs excluded from Git |
| Canonical docs | spec.md, plan.md, test-plan.md, benchmarks.md under docs/; README.md | Requirements, progress, checks, experiments and user instructions |

Shared contract vocabulary:

- ModelBundle: immutable ONNX/preprocessing/threshold identity and declared shapes.
- Prediction: finite defect score plus stage durations; ProcessingError: categorized failure.
- TaskKey: job ID plus image ID; AttemptKey: TaskKey plus persisted attempt number and coordinator epoch.
- RuntimeConfig: validated immutable-model and mutable operational settings, with effective settings recorded per run.
- FramedConnection: owns pending read/write buffers and emits typed messages or connection errors.
- BoundedQueue: a one-slot ownership handoff with explicit closure; no detached consumers.
- JobStore: owns transactional state transitions, conditional result commits and consistent snapshots.
- Scheduler: owns current session/attempt/lease decisions on the coordinator thread.
- Process harness: launches/records/reaps only its own child processes and emits reproducible fault timestamps.

Concrete signatures are intentionally left for the task briefing/implementation rather than giving a complete solution. Neighboring tasks must preserve the above contracts or record a justified interface change before updating consumers.

## 5. Verification convention and recordkeeping

[docs/test-plan.md](test-plan.md) defines the V-* verification recipes and which task must first create each command/target/test. [docs/benchmarks.md](benchmarks.md) defines V-BENCH and experiment evidence. They clearly distinguish proposed commands from actual verified ones; there are no assumed CMake presets or existing scripts.

Task outcomes below include implementation of their behavioral tests, not merely writing code that compiles. For a task adding native tests, register the named case and label and update tests/CMakeLists.txt. A task using V-ML also supplies its CLI operation and any configuration it references. Configuration/output paths are documented contracts, not files already present.

For every task, record the actual hours and review/verification/acceptance below when work occurs. Initially no findings, fixes, commands run, limitations discovered in implementation, or acceptance are recorded. Preserve meaningful failures and distinguish pre-existing issues from regressions.

A review record must include task ID, reviewed commit/diff identity, findings with consequences, minimal explicitly requested corrections, each actual command/result/evidence path, outstanding limitations, readiness and the user's explicit acceptance quote/date. Keep task state in its own task header consistent with that record.

## 6. Tasks

## M1 — Local infrastructure (20 hours)

### T001 — Verify the existing Linux environment

**State:** DONE. **Estimate:** 1 h. **Actual:** start 2026-11-05 17:17 Asia/Jerusalem (user-reported); duration not yet reported.

**Dependencies:** Approved plan and explicit task briefing request. **Requirements:** SCOPE-002, SCOPE-003, BUDGET-003, TEST-007.

**Files:** `docs/test-plan.md`, `README.md`.

**Interface:** Consumes the actual Ubuntu terminal/workspace; produces an environment inventory and an explicit missing-tool list.

**Outcome/scope:** Verify Linux/WSL identity, repository root/Git state, CPU/RAM, tool locations and versions. Explain Linux versus Windows paths and record evidence without changing setup.

**Exclusions:** No installation, Git initialization, source scaffold, or inference.

**Acceptance:** Every required tool is recorded as available or missing; Git is inspected at the actual root. User-performed Git initialization is acknowledged without marking infrastructure complete.

**Verification:** V-ENV; inspect the recorded outputs and missing-tool list.

**Briefing record (2026-11-05):** User requested starting tasks and practice. Briefed the purpose and read-only Ubuntu-terminal inventory; user will return command output and observations. No installation, environment check execution by the assistant, test result, readiness, or task acceptance is implied. Documentation will be updated from the supplied evidence.

**Review record (2026-11-05):** User submitted the requested terminal output; reviewed as IN_REVIEW and advanced to READY_FOR_ACCEPTANCE. V-ENV inventory requirements passed based on user-executed evidence. Linux/WSL identity, actual repository/Git state, CPU/RAM, available tool paths/versions and missing tools are all recorded in [test-plan.md, T001 environment inventory](test-plan.md#t001-environment-inventory-2026-11-05). README.md now records the verified environment and missing setup. The assistant maintained these documentation records as requested; no implementation/configuration fixes or installations were made.

**Findings and limitations:** GCC 15.2.0 and system Python 3.14.4 are on Linux paths. CMake, CTest, GDB, clang-format, clang-tidy and perf were not found on the user's PATH; `cmake --version` also failed with command not found. These are expected prerequisite findings, not blockers to an inventory task. Native C++20 compilation, Python dependency compatibility, application builds/tests, debugger/sanitizer execution and profiler capability are not verified by T001 and remain later-task work. Python 3.14.4 is not selected as the project interpreter. The observed dirty Git state is the existing documentation work; no reset or initialization is needed. Missing elapsed time remains a bookkeeping follow-up, not an unrun inventory check.

**Reviewed state/evidence:** Repository `/home/weam/projects/DefectRay`, HEAD `bffdd74063faba11c261432c3a890c8da0c2fb9c`; user's pre-recording status was ` M AGENTS.md` and `?? docs/`. Exact commands and relevant output are retained in test-plan.md; evidence source is the user's transcript, not an assistant rerun. No numeric shell exit codes were supplied. No blocking T001 findings; user acceptance is recorded below.

**Acceptance record (2026-11-05):** User explicitly said “accept T001”. Marked DONE based on the recorded V-ENV review. Elapsed effort remains unreported; acceptance does not imply one hour was spent. T002 remains TODO and requires an explicit briefing request.

**Learning checkpoint:** GCC compiles C++; CMake prepares build configuration; CTest runs registered tests. Recording a missing prerequisite completes its inventory check without proving the toolchain is ready to build.

### T002 — Define layout and Git hygiene

**State:** READY_FOR_ACCEPTANCE. **Estimate:** 1 h. **Actual:** briefing requested on 2026-11-05; start time and elapsed effort not yet reported.

**Dependencies:** T001. **Requirements:** SCOPE-004, PROD-010, BUDGET-003.

**Files:** `README.md`, `.gitignore`.

**Interface:** Produces the documented future directory map and ignore rules.

**Outcome/scope:** Document source/test/data/artifact locations; ignore generated builds, .venv, datasets, model weights, jobs, and full evidence. Keep small reproducible fixtures eligible for Git.

**Exclusions:** No application source, containers, or deleting existing files.

**Acceptance:** git check-ignore identifies representative generated paths and leaves source/fixture paths trackable; existing user work is preserved.

**Verification:** V-HYGIENE; review path examples without creating large artifacts.

**Briefing record (2026-11-05):** User explicitly requested “start T002”. T001 is accepted. README.md exists and .gitignore is absent. Briefed a small README directory map and root .gitignore rules for generated builds, the Python environment, datasets, models, jobs and full evidence; preserve trackable source, documentation and small fixtures. Explain one root-relative directory ignore example without supplying the complete configuration. Verification uses V-HYGIENE plus representative generated-job and small-fixture paths, which need not exist. The human edits README.md and .gitignore, runs the checks and returns output for review. No implementation/configuration files were changed by the assistant; no T002 checks, readiness or acceptance are claimed.

**Delegation and review record (2026-11-05):** User explicitly asked “can you do it? i want to work on the real implementation”, authorizing the assistant to implement T002. Added a planned directory map to README.md and five root-directory ignore rules for build/, .venv/, data/, models/ and artifacts/; generated jobs and full evidence live under artifacts/. Reviewed through IN_REVIEW to READY_FOR_ACCEPTANCE. V-HYGIENE passed: all six generated path examples matched the intended rules; source, test fixtures, Python tooling, documentation and .gitignore remained trackable. No blocking findings or further fixes are required. Actual commands, exit statuses, outputs and reviewed file hashes are in [test-plan.md](test-plan.md#t002-layout-and-git-hygiene-2026-11-05).

**Limits and acceptance:** The example paths were checked without creating their directories or contents. Existing AGENTS.md, spec.md and benchmarks.md changes were preserved; no Git metadata, application code, installations or future tasks were changed. No application build/test result is established. User acceptance is pending; no human elapsed effort is inferred from the assistant's work.

**Learning checkpoint:** Root-directory ignore rules exclude generated artifacts while allowing small PNG/JSON fixtures elsewhere. Ignore rules do not untrack files already in Git.

### T003 — Verify dependency compatibility

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T001. **Requirements:** GATE-001, GATE-003, BUDGET-003.

**Files:** `docs/test-plan.md`.

**Interface:** Produces a release-specific compatibility decision used by all dependency tasks.

**Outcome/scope:** Check primary release metadata for Python/torch/torchvision/onnx/onnxruntime/exporter/NumPy/OpenCV and C++ ABI requirements. Record native Asio, SQLite, JSON, hashing and test-framework choices and acquisition sources. Select Python only after evidence supports the set.

**Exclusions:** No installs, downloads of weights/dataset, or automatically selecting system Python.

**Acceptance:** Matrix has versions, CPU/Linux support, torch/vision pairing, exporter/runtime opset/IR support, native ABI notes and source links. Unresolved conflicts block dependent setup.

**Verification:** V-MATRIX (document/source review); no package smoke test is claimed here.

### T004 — Verify or complete the authorized native toolchain

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T001, T003. **Requirements:** SCOPE-002, GATE-003, BUDGET-003.

**Files:** `README.md`, `docs/test-plan.md`.

**Interface:** Produces a verified Linux compiler/build/debug/lint toolchain.

**Outcome/scope:** Verify existing compiler, CMake, test runner support, debugger and formatting/analysis tools; the human installs only approved missing tools. Record Linux paths and verified version commands.

**Exclusions:** No Python environment, application features, global configuration changes, or unattended installs.

**Acceptance:** Compiler supports the intended C++20 build; CMake, debugger and checks are available from Linux paths. Missing prerequisites are resolved or the task is BLOCKED.

**Verification:** V-ENV restricted to the approved toolchain; record any separately authorized installation commands.

### T005 — Build the minimal C++20 Debug executable

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T002, T004. **Requirements:** SCOPE-002, BUDGET-003.

**Files:** `CMakeLists.txt`, `src/main.cpp`, `docs/test-plan.md`, `README.md`.

**Interface:** Produces build/debug/defectray from a minimal C++20 executable.

**Outcome/scope:** Create the smallest out-of-source Debug build and document the configure/build commands. Give the initial executable a clearly limited smoke behavior.

**Exclusions:** No coordinator, worker, networking or ML logic.

**Acceptance:** Fresh Debug configuration and compilation succeed; the executable runs; generated outputs stay under build/. No inspection capability is claimed.

**Verification:** V-DEBUG then run the minimal executable's documented smoke behavior.

### T006 — Add an independent Release build

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T005. **Requirements:** BUDGET-003, TEST-007.

**Files:** `CMakeLists.txt`, `docs/test-plan.md`, `README.md`.

**Interface:** Produces separate build/release output using the same source.

**Outcome/scope:** Add and verify the Release configuration; keep Debug artifacts independent. Record optimization/debug settings without making a performance claim.

**Exclusions:** No benchmark or architecture changes.

**Acceptance:** Debug and Release both build and run; neither configuration overwrites the other.

**Verification:** V-DEBUG and V-RELEASE.

### T007 — Register the C++ test framework and smoke test

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T005, T003. **Requirements:** TEST-001, TEST-007, BUDGET-003.

**Files:** `CMakeLists.txt`, `tests/CMakeLists.txt`, `tests/unit/test_smoke.cpp`, `docs/test-plan.md`.

**Interface:** Produces a CTest-registered test named smoke and the fast label.

**Outcome/scope:** Integrate the selected small C++ test framework, with checks active in Release, and one meaningful executable/library smoke test. Establish explicit failure on an empty test selection.

**Exclusions:** No domain behavior or large test abstraction framework.

**Acceptance:** CTest lists smoke; the expected behavior passes in Debug/Release; an intentionally failing assertion is observed once and restored. Empty selections cannot masquerade as success.

**Verification:** V-CTEST(smoke), V-FAST; record the deliberate failure/restoration evidence.

### T008 — Verify VS Code Linux debugging

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T005, T004. **Requirements:** BUDGET-003.

**Files:** `.vscode/tasks.json`, `.vscode/launch.json`, `README.md`, `docs/test-plan.md`.

**Interface:** Produces a repeatable VS Code-to-Linux debugger workflow.

**Outcome/scope:** Configure the editor to build and debug the Linux executable with paths matching the actual workspace. Verify a breakpoint and variable inspection.

**Exclusions:** No Windows-native compiler/environment or editor-wide settings.

**Acceptance:** A breakpoint in the minimal program is hit inside Linux and a variable/call frame is inspected. The human records the observation.

**Verification:** V-DEBUGGER (manual IDE check) plus V-DEBUG.

### T009 — Add a read-only formatting check

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T005, T004. **Requirements:** BUDGET-003, TEST-007.

**Files:** `.clang-format`, `CMakeLists.txt`, `docs/test-plan.md`.

**Interface:** Produces the format-check target.

**Outcome/scope:** Configure a narrow C++ formatting policy and a read-only formatting check using the verified formatter. Check only maintained project files.

**Exclusions:** No unrelated formatting, automatic commit hooks, or third-party source edits.

**Acceptance:** Deliberately misformatted fixture text fails the check; restored code passes; checks do not modify files.

**Verification:** V-FORMAT.

### T010 — Add focused static analysis

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T005, T004. **Requirements:** BUDGET-003, CPP-001, TEST-007.

**Files:** `.clang-tidy`, `CMakeLists.txt`, `docs/test-plan.md`.

**Interface:** Produces the analyze target using actual compile commands.

**Outcome/scope:** Enable a small useful static-analysis set and a compilation database. Demonstrate detection of one relevant misuse and then remove it.

**Exclusions:** No blanket suppression or exhaustive warning-policy project.

**Acceptance:** Analysis covers maintained C++ code and detects the seeded issue; clean code passes with documented third-party exclusions.

**Verification:** V-ANALYZE.

### T011 — Verify separate ASan and UBSan instrumentation

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T007, T004. **Requirements:** CPP-001, TEST-006, BUDGET-003.

**Files:** `CMakeLists.txt`, `cmake/Sanitizers.cmake`, `docs/test-plan.md`.

**Interface:** Produces a separate address-undefined sanitizer configuration.

**Outcome/scope:** Add the address/undefined sanitizer option for supported compilers; verify runtime instrumentation with a temporary negative fixture outside maintained tests.

**Exclusions:** No thread sanitizer in the same configuration.

**Acceptance:** The negative fixture triggers instrumentation; restored smoke tests pass under ASan+UBSan. Unsupported environments are recorded as blockers.

**Verification:** V-ASAN.

### T012 — Verify separate TSan instrumentation

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T007, T004. **Requirements:** CPP-002, TEST-006, BUDGET-003.

**Files:** `cmake/Sanitizers.cmake`, `CMakeLists.txt`, `docs/test-plan.md`.

**Interface:** Produces a separate thread sanitizer configuration.

**Outcome/scope:** Add TSan separately and verify detection with a controlled race fixture before running clean smoke tests.

**Exclusions:** No combining TSan with ASan, no weakening tests to hide host restrictions.

**Acceptance:** Race instrumentation is demonstrated; restored tests pass. If the host cannot run TSan, document the actual failure and resolve the prerequisite before acceptance.

**Verification:** V-TSAN.

### T013 — Create the verified local Python environment

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T003, T004. **Requirements:** GATE-001, GATE-003, BUDGET-003.

**Files:** `pyproject.toml`, `requirements.lock`, `README.md`, `docs/test-plan.md`.

**Interface:** Produces .venv using the matrix-selected interpreter and a reproducible dependency lock.

**Outcome/scope:** The human creates the project-local environment and installs only the approved package set. Pin direct/transitive versions and acquisition sources; use a supported resolver/locking method verified in T003.

**Exclusions:** No weights, full dataset, training, or use of system Python merely because it exists.

**Acceptance:** Interpreter resolves inside .venv at the selected version; dependency consistency passes; lock reproduces the environment. Native OpenCV compatibility is recorded.

**Verification:** V-PYENV; record the actual environment-creation and lock-generation commands selected from T003.

### T014 — Add the fast Python smoke suite

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T013. **Requirements:** BUDGET-003, TEST-007.

**Files:** `python/tests/fast/test_smoke.py`, `pyproject.toml`, `docs/test-plan.md`.

**Interface:** Produces a fast Python unittest discovery suite.

**Outcome/scope:** Add a small CPU tensor/NumPy conversion and dependency-import smoke check with no network/model acquisition. Configure local package imports for later ML/report modules.

**Exclusions:** No training or dataset access.

**Acceptance:** The selected interpreter discovers and passes a nonempty suite, runs on CPU, and performs no downloads.

**Verification:** V-PYFAST and V-PYENV.

### T015 — Verify minimal PR CI

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T006, T007, T009, T010, T011, T012, T014. **Requirements:** BUDGET-003, TEST-007.

**Files:** `.github/workflows/ci.yml`, `docs/test-plan.md`, `README.md`.

**Interface:** Produces a fast PR workflow composed from accepted local commands.

**Outcome/scope:** Configure minimal Linux CI for build, formatting/basic analysis, C++ fast tests and Python fast tests. Use a documented compatible runner/toolchain; keep expensive suites separate.

**Exclusions:** No pushes or workflow execution by the assistant without authorization; no full dataset/weights/training.

**Acceptance:** Workflow command definitions match the verified local commands; a user-authorized run passes and its URL/log is retained. If remote execution is unavailable, readiness remains blocked.

**Verification:** V-FAST V-PYFAST V-FORMAT V-ANALYZE, plus the actual authorized CI run.

### T016 — Reproduce and accept the local foundation

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T001, T002, T003, T004, T005, T006, T007, T008, T009, T010, T011, T012, T013, T014, T015. **Requirements:** SCOPE-003, SCOPE-005, BUDGET-001, BUDGET-002, BUDGET-003, BUDGET-004, BUDGET-005, TEST-007.

**Files:** `README.md`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Produces M1's reproducible setup instructions and acceptance evidence.

**Outcome/scope:** Walk through setup from a user-provided clean checkout or temporary source-only copy; verify existing setup rather than reinstalling it. Record actual M1 time and all required checks.

**Exclusions:** No autonomous clone/worktree/commit, starting T017, or claiming completed application behavior.

**Acceptance:** Build, tests, checks, debugger and CI evidence are reproducible; no hidden global dependency remains. User explicitly accepts M1 after reviewing time and remaining risk.

**Verification:** V-FOUNDATION; use the verified README commands, not an invented preset.

## M2 — Inference feasibility (12 hours)

### T017 — Resolve permissions and acquire approved ML artifacts

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T016. **Requirements:** ML-001, ML-002, GATE-002, GATE-003.

**Files:** `docs/test-plan.md`, `README.md`.

**Interface:** Produces verified acquisition provenance and local dataset/weight artifacts after authorization.

**Outcome/scope:** First resolve the exact pretrained-weight terms and educational use. Then, only with download permission, obtain the official dataset and selected weights, verify hashes/layout/labels, and record attribution/redistribution constraints.

**Exclusions:** No ImageNet dataset, substitutions, training, or hidden downloads in model constructors.

**Acceptance:** Permission evidence and exact artifact identities exist; dataset archive integrity and documented label mapping are verified. Unclear terms block acquisition and require a recorded model decision.

**Verification:** V-ACQUIRE; inspect hashes/metadata and permissions before any acquisition command.

### T018 — Freeze reproducible dataset splits

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T017. **Requirements:** ML-001, ML-003.

**Files:** `python/defectray_ml/data.py`, `python/tests/fast/test_splits.py`, `python/defectray_ml/__init__.py`, `python/defectray_ml/__main__.py`.

**Interface:** Consumes official labels; produces immutable train/validation/test manifests.

**Outcome/scope:** Implement the seeded 80/20 stratified split from the official training partition; preserve test membership. Check content duplicates and group related views where identifiers permit.

**Exclusions:** No fitting, threshold selection, or reading test labels for tuning.

**Acceptance:** Manifest hashes repeat with seed 42; partitions are disjoint; class counts and grouping limitations are recorded; deliberate duplicate leakage is detected.

**Verification:** V-PYFAST; V-ML(splits), producing artifacts/ml/splits/.

### T019 — Build Python preprocessing reference fixtures

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T017, T014. **Requirements:** ML-005, ML-007, CPP-003.

**Files:** `python/defectray_ml/preprocess.py`, `python/defectray_ml/fixtures.py`, `python/tests/fast/test_preprocess.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces RGB float32 NCHW tensors and versioned preprocessing.json.

**Outcome/scope:** Implement the exact full-image resize/pad/normalization contract and generate at least eight deterministic fixtures with expected tensors. Include RGB/grayscale, odd padding and extreme aspect ratios.

**Exclusions:** No augmentation, center cropping, or recipe changes.

**Acceptance:** Shapes, RGB order, padding placement, values and dimension limits match ML-005; fixture generation repeats byte-for-byte where applicable.

**Verification:** V-PYFAST; V-ML(fixtures), producing artifacts/fixtures/parity/.

### T020 — Train an initial candidate classification head

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T018, T019. **Requirements:** ML-002, ML-003, ML-007.

**Files:** `python/defectray_ml/model.py`, `python/defectray_ml/train.py`, `python/defectray_ml/metrics.py`, `python/tests/fast/test_training_contract.py`, `python/defectray_ml/__main__.py`.

**Interface:** Consumes training/validation manifests; produces an initial trained candidate head and recorded threshold.

**Outcome/scope:** Adapt the selected network to one binary logit and train its head with a frozen/eval backbone. Save seed/config, labels and validation-selected threshold for an initial real-model demonstration.

**Exclusions:** No held-out evaluation, full-backbone tuning, model search or final-quality claim.

**Acceptance:** Head parameters change while frozen backbone parameters/buffers do not; loss and validation predictions are finite; outputs identify the candidate and its split. Later M5 supplies the final quality gate.

**Verification:** V-PYFAST; V-ML(candidate-train), producing artifacts/ml/candidate/.

### T021 — Export and validate the candidate ONNX bundle

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T020, T003. **Requirements:** ML-006, PROD-006, PROD-010.

**Files:** `python/defectray_ml/export.py`, `python/defectray_ml/bundle.py`, `python/tests/model/test_export.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces artifacts/models/candidate/ with ONNX, preprocessing and hashed manifest.

**Outcome/scope:** Export the complete batch-one graph including sigmoid, pin the compatible opset/IR, run the ONNX checker, and compare Python ONNX outputs with PyTorch. Bind threshold/labels into bundle identity.

**Exclusions:** No dynamic batch shapes, quantization or arbitrary model support.

**Acceptance:** The bundle has the exact declared input/output shapes and identity checks; Python runtime scores satisfy the tolerance; corrupt or mismatched bundle contents fail explicitly.

**Verification:** V-ML(export-candidate); V-PYMODEL(test_export.py).

### T022 — Run one image through C++ inference

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T021, T007. **Requirements:** ML-005, ML-006, CPP-001, CPP-003, CPP-005.

**Files:** `src/inference/model_bundle.hpp`, `src/inference/model_bundle.cpp`, `src/inference/preprocess.hpp`, `src/inference/preprocess.cpp`, `src/inference/engine.hpp`, `src/inference/engine.cpp`, `tools/infer_one.cpp`, `tests/model/test_inference.cpp`, `CMakeLists.txt`, `cmake/Dependencies.cmake`.

**Interface:** ModelBundle identifies the immutable model; InferenceEngine consumes encoded PNG bytes and returns Prediction(score, stage timings) or ProcessingError.

**Outcome/scope:** Integrate verified native OpenCV/ONNX Runtime dependencies. Implement one-image CPU decode/preprocess/inference with RAII, size checks and explicit runtime threads; provide a small inspection executable.

**Exclusions:** No TCP, scheduler, concurrency pool or custom neural operations.

**Acceptance:** A real candidate image runs in C++; shapes and finite score bounds are checked; oversized/wrong-format images and malformed model metadata fail safely.

**Verification:** V-DEBUG; V-CTEST(model_inference); record the actual infer_one invocation in test-plan.

### T023 — Prove three-runtime parity

**State:** TODO. **Estimate:** 1 h. **Actual:** not started.

**Dependencies:** T022, T019, T021. **Requirements:** ML-007, ML-006, GATE-001, GATE-002, BUDGET-001.

**Files:** `python/tools/check_parity.py`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Consumes reference fixtures and infer_one; produces a three-runtime parity report.

**Outcome/scope:** Compare PyTorch, Python ONNX and C++ ONNX outputs on a real image and all eight or more fixtures. Inspect tensor differences before score differences and review M2 spending.

**Exclusions:** No complex scheduler or accepting an untrained model as completed defect detection.

**Acceptance:** Tensor/score tolerances and decision-margin rules pass with recorded versions/hashes/commands. M2 acceptance requires actual compatibility and acquisition evidence.

**Verification:** V-PARITY(candidate); preserve artifacts/evidence/m2-parity/; request explicit milestone acceptance.

## M3 — First complete workflow (24 hours)

### T024 — Define shared identities and runtime contracts

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T023. **Requirements:** ARCH-001, ARCH-002, PROD-005, PROD-006, CPP-001.

**Files:** `src/common/identity.hpp`, `src/common/identity.cpp`, `src/config/runtime_config.hpp`, `src/config/runtime_config.cpp`, `src/protocol/messages.hpp`, `src/protocol/messages.cpp`, `cmake/Dependencies.cmake`, `tests/unit/test_contracts.cpp`.

**Interface:** Produces JobId/ImageId, TaskKey, AttemptKey, worker-session identity, RuntimeConfig and typed messages; reuses ModelBundle.

**Outcome/scope:** Add only required Asio/SQLite/JSON/hash dependency wiring and shared domain types. Validate configuration and message field types; keep immutable model identity distinct from mutable run settings.

**Exclusions:** No scheduler implementation or general-purpose framework.

**Acceptance:** Invalid limits/endpoints/field types and modified bundle identities are rejected; defaults match spec; identifiers round-trip without numeric precision loss.

**Verification:** V-DEBUG; V-CTEST(contracts); V-FAST.

### T025 — Implement the bounded frame codec

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T024. **Requirements:** NET-001, NET-002, NET-003, CPP-003, TEST-001.

**Files:** `src/protocol/frame.hpp`, `src/protocol/frame.cpp`, `tests/unit/test_frame_codec.cpp`.

**Interface:** FrameDecoder consumes arbitrary byte spans; frame encoder produces the specified header/metadata/binary sequence.

**Outcome/scope:** Implement bounded version-1 framing with network-order integers and type-specific validation. Keep parser state independent of sockets.

**Exclusions:** No connection management or inference.

**Acceptance:** Every split point, two concatenated frames, unknown type/version, oversized lengths and arithmetic overflow have explicit expected outcomes before body allocation.

**Verification:** V-CTEST(frame_codec); V-FAST.

### T026 — Carry framed messages over real asynchronous TCP

**State:** TODO. **Estimate:** 3 h. **Actual:** not started.

**Dependencies:** T025. **Requirements:** ARCH-004, NET-003, CPP-001, CPP-003, TEST-002.

**Files:** `src/net/framed_connection.hpp`, `src/net/framed_connection.cpp`, `tests/integration/test_tcp_stream.cpp`.

**Interface:** FramedConnection emits validated Message or ConnectionError and owns serialized asynchronous writes.

**Outcome/scope:** Connect the codec to real Asio TCP sockets. Preserve buffer lifetime through completion/cancellation, handle partial progress, serialize writes, and bound output queues.

**Exclusions:** No heartbeat/retry policy yet; no custom epoll reactor.

**Acceptance:** Real loopback tests transfer fragmented/coalesced frames, simulate slow readers and cancellation, and prove frames do not interleave or exceed byte caps.

**Verification:** V-CTEST(tcp_stream); V-FAST.

### T027 — Implement the one-slot ownership handoff

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T023, T007. **Requirements:** CPP-001, CPP-002, CPP-003, TEST-001.

**Files:** `src/worker/bounded_queue.hpp`, `tests/unit/test_bounded_queue.cpp`.

**Interface:** BoundedQueue owns one request slot; close wakes waiters and future operations report closure.

**Outcome/scope:** Implement the one-slot mutex/condition-variable handoff with explicit ownership moves and predicate waits.

**Exclusions:** No lock-free queue, arbitrary pool or blocking the network thread on queue capacity.

**Acceptance:** Empty/full/closed cases, producer-consumer handoff and close-while-waiting terminate deterministically without lost requests or deadlock.

**Verification:** V-CTEST(bounded_queue); V-TSAN on the bounded_queue selection.

### T028 — Connect the worker's network and inference threads

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T026, T027, T022. **Requirements:** ARCH-003, CPP-001, CPP-002, CPP-005, CPP-006.

**Files:** `src/worker/runtime.hpp`, `src/worker/runtime.cpp`, `tests/integration/test_worker_runtime.cpp`, `src/common/buffer_accounting.hpp`.

**Interface:** WorkerRuntime consumes ASSIGN through FramedConnection, owns InferenceEngine on its inference thread, and posts one result back.

**Outcome/scope:** Connect a worker's network and inference threads using the queue and one completion handoff. Reject work while busy; keep basic stop/cancel/join behavior correct. Implement explicit reservation/release accounting across encoded, decoded, tensor and transport-owned buffers; reject work before exceeding the worker cap and expose high-water counters.

**Exclusions:** No worker pool, retry decisions, detached threads or production fake-inference mode.

**Acceptance:** A worker returns real model results while its network loop remains responsive; a deterministic test adapter verifies ownership and idle shutdown; a second simultaneous assignment is rejected. Reservations include staging/capacity and return to the defined idle baseline.

**Verification:** V-CTEST(worker_runtime); V-CTEST(worker_runtime_model); V-FAST.

### T029 — Persist a fresh job with SQLite

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T024. **Requirements:** ARCH-001, DIST-007, PROD-007, PROD-010, CPP-001.

**Files:** `src/store/job_store.hpp`, `src/store/job_store.cpp`, `src/store/schema.sql`, `tests/unit/test_job_store.cpp`.

**Interface:** JobStore owns SQLite transactions/snapshots for jobs, images, tasks, attempts and committed results.

**Outcome/scope:** Create version-1 schema with identities, epoch/attempt counters, foreign keys and unique task results. Provide basic fresh-job persistence and consistent snapshots; retain rollback journal/synchronous FULL.

**Exclusions:** No restart recovery or claim that schema constraints alone implement leases.

**Acceptance:** Close/reopen preserves records; uniqueness/foreign-key violations fail; rolled-back transactions leave no partial state; snapshots satisfy basic count invariants.

**Verification:** V-CTEST(job_store); V-FAST.

### T030 — Import bounded immutable image collections

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T029, T024. **Requirements:** PROD-001, PROD-002, PROD-003, PROD-010, DIST-007.

**Files:** `src/job/importer.hpp`, `src/job/importer.cpp`, `tests/integration/test_import.cpp`.

**Interface:** Importer consumes a source directory and creates an accepted immutable job manifest/store only after copies complete.

**Outcome/scope:** Implement bounded nonrecursive PNG import, content hashing/deduplication and aliases, safe managed paths, collection/file limits, and terminal input-error records.

**Exclusions:** No recursive directories, symlink following or resumable import.

**Acceptance:** Duplicate bytes create one task; oversized/empty/unreadable submissions reject; corrupt in-limit PNG gets a recorded input failure; interrupted copying never produces an accepted job.

**Verification:** V-CTEST(import_job); V-FAST.

### T031 — Assign work to two compatible workers

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T028, T029, T025. **Requirements:** ARCH-003, ARCH-005, DIST-003, NET-002, NET-005.

**Files:** `src/coordinator/scheduler.hpp`, `src/coordinator/scheduler.cpp`, `src/coordinator/worker_session.hpp`, `src/coordinator/worker_session.cpp`, `tests/integration/test_assignment.cpp`.

**Interface:** Scheduler consumes READY and store snapshots; produces one ASSIGN per compatible worker and initial successful outcomes.

**Outcome/scope:** Implement HELLO/WELCOME/bundle verification and deterministic demand-driven assignment for a fresh, failure-free job. Page tasks and load bytes only for available capacity. Reuse owned-buffer accounting for both coordinator connections and input staging.

**Exclusions:** No automatic retries/restart; do not claim full failure correctness before M4.

**Acceptance:** Two real sessions receive distinct tasks; a busy/mismatched/duplicate/excess worker is rejected; no path-only image transfer or whole-dataset RAM preload occurs.

**Verification:** V-CTEST(assignment); V-FAST.

### T032 — Expose the foreground CLI workflow

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T030, T031. **Requirements:** PROD-004, PROD-005, PROD-007, PROD-008, SCOPE-001.

**Files:** `src/cli/commands.hpp`, `src/cli/commands.cpp`, `src/main.cpp`, `tests/integration/test_cli.cpp`, `src/common/events.hpp`, `src/common/events.cpp`.

**Interface:** Produces submit, worker and status entry points using the foreground coordinator.

**Outcome/scope:** Wire the actual application workflow and validated options; expose progress and clear operational errors. Preserve model/configuration identities in output. Emit structured events.jsonl with job/task/attempt/worker-session IDs and monotonic event fields, and a separate human-readable progress display.

**Exclusions:** No remote submission API or pretending resume works before T042.

**Acceptance:** User can start two workers, submit a valid directory and inspect durable progress; bad arguments, occupied port and rejected inputs return documented failures without touching other processes.

**Verification:** V-CTEST(cli); V-FAST; record actual CLI examples in README.

### T033 — Export consistent JSON results

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T032. **Requirements:** PROD-007, PROD-009, PROD-010.

**Files:** `src/report/json_export.hpp`, `src/report/json_export.cpp`, `tests/unit/test_report_json.cpp`.

**Interface:** Consumes a consistent JobStore snapshot; produces schema-version-1 results.json.

**Outcome/scope:** Export progress, unique image/alias counts, task scores/errors, committed attempts and model/configuration identity. Stream rows and atomically replace completed output.

**Exclusions:** No HTML or changing job outcomes because export fails.

**Acceptance:** Counts reconcile for mixed outcomes; pending tasks are labeled partial; invalid output paths fail without changing results; re-export is repeatable.

**Verification:** V-CTEST(report_json); V-FAST.

### T034 — Render the readable HTML report

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T033, T014. **Requirements:** PROD-004, PROD-008, PROD-009.

**Files:** `python/defectray_report/__main__.py`, `python/defectray_report/render.py`, `python/tests/fast/test_report_html.py`, `src/cli/commands.cpp`, `python/defectray_report/__init__.py`.

**Interface:** Consumes results.json; produces escaped static report.html via a safely invoked renderer.

**Outcome/scope:** Build the minimal readable summary and flagged-image/error sections; link managed images with safe relative paths. Add the report command and explicit renderer-failure handling.

**Exclusions:** No frontend framework, dashboard, custom charts or shell-string invocation.

**Acceptance:** HTML and JSON totals match; hostile filenames/error text are escaped; partial reports are obvious; rendering failure preserves durable results and is retryable.

**Verification:** V-PYFAST; V-CTEST(report_command); inspect one generated report.

### T035 — Demonstrate the first complete workflow

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T034. **Requirements:** SCOPE-001, ARCH-005, TEST-003, BUDGET-001.

**Files:** `python/tests/model/test_end_to_end.py`, `docs/test-plan.md`, `docs/plan.md`, `README.md`.

**Interface:** Produces the first complete two-worker inspection demonstration.

**Outcome/scope:** Create a process-owning harness for the real candidate model with valid images, duplicate bytes and corrupt PNG. Register it in CTest, and capture the actual launch/report workflow.

**Exclusions:** No recovery claim or held-out model-quality claim.

**Acceptance:** Two actual workers perform inference over TCP; every admitted unique image has one terminal outcome; aliases and HTML/JSON counts agree. All harness-owned processes/data are cleaned up.

**Verification:** V-CTEST(e2e_candidate); V-FAST V-PYFAST; request M3 acceptance with cumulative time.

## M4 — Durable scheduling and recovery (26 hours)

### T036 — Define legal task and attempt transitions

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T035. **Requirements:** DIST-001, DIST-002, PROD-007, TEST-001.

**Files:** `src/coordinator/task_state.hpp`, `src/coordinator/task_state.cpp`, `tests/unit/test_task_state.cpp`.

**Interface:** Produces pure task/attempt transition decisions consumed by scheduler/store.

**Outcome/scope:** Model legal task and attempt transitions, permanent versus retryable failures, and three total attempts. Keep transitions testable without sockets/database.

**Exclusions:** No production persistence changes or clocks yet.

**Acceptance:** Transition table tests cover every terminal path, retry boundary, direct input failure and illegal transition. Terminal task outcomes cannot reopen.

**Verification:** V-CTEST(task_state); V-FAST.

### T037 — Commit assignments before transmission

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T036. **Requirements:** DIST-002, DIST-003, DIST-007.

**Files:** `src/store/job_store.hpp`, `src/store/job_store.cpp`, `src/coordinator/scheduler.cpp`, `tests/integration/test_assignment_commit.cpp`.

**Interface:** Produces atomic assignment records before any ASSIGN send.

**Outcome/scope:** Commit ACTIVE state, attempt identity/count and assignment metadata transactionally before dispatch. Use a controlled send failure to exercise the boundary.

**Exclusions:** No retry refund when transmission never begins; no network I/O inside a transaction.

**Acceptance:** An intercepted outbound assignment always has a durable attempt; pre-send failure consumes one attempt; rollback cannot leave split task/attempt state.

**Verification:** V-CTEST(assignment_commit); V-FAST.

### T038 — Commit one authoritative result before ACK

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T037. **Requirements:** DIST-005, DIST-008, NET-002, PROD-006.

**Files:** `src/store/job_store.hpp`, `src/store/job_store.cpp`, `src/coordinator/scheduler.cpp`, `tests/integration/test_result_commit.cpp`.

**Interface:** JobStore conditionally commits a current AttemptKey; scheduler returns explicit ACK dispositions.

**Outcome/scope:** Fence results by current task/attempt/epoch/session/bundle and lease; atomically commit authoritative result/state before ACK. Recognize identical retransmissions and reject conflicts.

**Exclusions:** No exactly-once execution claim or accepting stale results merely because their score looks valid.

**Acceptance:** Duplicate deliveries create one result/count change; conflicting/stale identities cannot overwrite it; failed commit emits no successful ACK.

**Verification:** V-CTEST(result_commit); V-FAST.

### T039 — Enforce network and health deadlines

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T038. **Requirements:** NET-004, NET-003, CPP-003.

**Files:** `src/net/framed_connection.cpp`, `src/coordinator/worker_session.cpp`, `src/worker/runtime.cpp`, `tests/integration/test_network_deadlines.cpp`.

**Interface:** Produces frame deadlines and heartbeat/health events from monotonic timers.

**Outcome/scope:** Apply connect/registration/full-frame deadlines and worker health timing. Heartbeats remain responsive during inference and are coalesced when unsent.

**Exclusions:** No renewing inference leases via heartbeats or extending a deadline on each byte.

**Acceptance:** Trickled frames expire, a healthy busy worker remains responsive, stalled writes stay bounded, and the other connection progresses. Defaults match NET-004.

**Verification:** V-CTEST(network_deadlines); V-FAST.

### T040 — Expire leases and bound retries

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T039. **Requirements:** DIST-004, DIST-002, NET-004.

**Files:** `src/coordinator/scheduler.cpp`, `src/store/job_store.cpp`, `tests/integration/test_leases_retries.cpp`.

**Interface:** Consumes disconnect/health/lease events; produces durable retry or terminal failure.

**Outcome/scope:** Enforce the fixed assignment deadline independently of worker heartbeat. Close timed-out sessions, persist terminal attempt reason, and requeue only with remaining attempts.

**Exclusions:** No unlimited retries or automatic worker-process restart.

**Acceptance:** A heartbeating stuck worker still expires; exactly three assigned attempts are allowed; permanent input/output errors fail without pointless retry; persisted counts survive reopen.

**Verification:** V-CTEST(leases_retries); V-FAST.

### T041 — Reconnect with session fencing

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T040. **Requirements:** NET-005, NET-004, ARCH-003.

**Files:** `src/worker/runtime.cpp`, `src/coordinator/worker_session.cpp`, `tests/integration/test_reconnect.cpp`.

**Interface:** Produces new session registration after bounded-backoff reconnect.

**Outcome/scope:** Handle reconnect delays and new-session fencing. Discard abandoned results and wait for old native inference to finish before advertising READY. Reject duplicate live IDs and excess sessions.

**Exclusions:** No parallel attempts on one worker or busy-loop reconnect.

**Acceptance:** Old-session results cannot commit, duplicate IDs do not evict unrelated healthy workers, and a restarted worker rejoins and receives work without exceeding capacity.

**Verification:** V-CTEST(reconnect); V-FAST.

### T042 — Resume under an exclusive job lock and new epoch

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T041. **Requirements:** DIST-006, DIST-007, PROD-004, PROD-005, ARCH-003.

**Files:** `src/job/job_lock.hpp`, `src/job/job_lock.cpp`, `src/coordinator/recovery.hpp`, `src/coordinator/recovery.cpp`, `src/store/job_store.cpp`, `src/cli/commands.cpp`, `tests/integration/test_resume.cpp`.

**Interface:** Produces exclusive ownership and resume with a new persisted coordinator epoch.

**Outcome/scope:** Acquire a process-held lock; validate schema/bundle; increment epoch and abandon old unfinished attempts in one recovery transaction. Requeue or fail exhausted tasks; add the real resume command.

**Exclusions:** No cross-boot monotonic deadline comparison, HA, or resuming half-imported directories.

**Acceptance:** A second coordinator is rejected; process death releases the lock; resume preserves committed results and attempt limits, rejects changed immutable configuration, and invalidates old epochs.

**Verification:** V-CTEST(resume_job); V-FAST.

### T043 — Derive paused and terminal job outcomes

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T042. **Requirements:** PROD-007, PROD-008, NET-004, DIST-001.

**Files:** `src/coordinator/scheduler.cpp`, `src/store/job_store.cpp`, `src/cli/commands.cpp`, `tests/integration/test_job_outcomes.cpp`.

**Interface:** Produces PAUSED and terminal job outcomes/exit statuses from authoritative task state.

**Outcome/scope:** Implement no-compatible-worker deadline and accurate partial/all-failed/all-success counts. Propagate operational storage/report failures without erasing accepted progress.

**Exclusions:** No marking unattempted images FAILED solely because no worker is available.

**Acceptance:** No-worker jobs pause with pending work; resume continues; mixed/permanent failures produce correct totals and exit codes; export failure does not change completed inference outcomes.

**Verification:** V-CTEST(job_outcomes); V-FAST.

### T044 — Shut down threads and resources safely

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T043. **Requirements:** CPP-006, CPP-001, CPP-002, PROD-008.

**Files:** `src/worker/runtime.cpp`, `src/coordinator/scheduler.cpp`, `src/cli/commands.cpp`, `tests/integration/test_shutdown.cpp`.

**Interface:** Produces ordered graceful shutdown and resumable interruption.

**Outcome/scope:** Handle SIGINT through owned stop paths: cease dispatch, cancel transport/timers, close queue, wake waits and join threads. Retain buffers until operations/calls finish.

**Exclusions:** No unsafe forced cancellation inside native inference or bounded-shutdown claim for a hung native call.

**Acceptance:** Idle/waiting/in-flight controlled cases exit safely; a two-second test inference permits exit within five seconds; SIGINT yields 130 and a resumable accepted job.

**Verification:** V-CTEST(shutdown); V-FAST.

### T045 — Create deterministic process fault controls

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T044. **Requirements:** TEST-004, TEST-005, TEST-007.

**Files:** `python/defectray_tools/processes.py`, `tests/support/fault_points.hpp`, `docs/test-plan.md`, `python/defectray_tools/__init__.py`.

**Interface:** Produces a harness that owns process IDs, synchronization events and isolated job data.

**Outcome/scope:** Add explicit test-only barriers at inference start, assignment-before-send and result-before-ACK. Launch/stop only child processes and record monotonic events with task/attempt IDs.

**Exclusions:** No sleep-and-hope timing, production delays, or broad process termination commands.

**Acceptance:** Harness waits for the exact targeted state, distinguishes timeout from success, retains failure evidence and reliably reaps its own children even when a check fails.

**Verification:** V-HARNESS; register process_harness with CTest.

### T046 — Prove interrupted-task recovery after worker death

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T045. **Requirements:** TEST-004, DIST-004, DIST-008.

**Files:** `python/tests/recovery/test_worker_crash.py`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Produces repeatable worker-kill/recovery evidence tied to an interrupted TaskKey.

**Outcome/scope:** Run five controlled stub trials and five real-candidate trials after an inference-start event. Verify surviving-worker completion and then worker re-registration.

**Exclusions:** No substituting reassignment for completed recovery or presenting stub timings as real-model results.

**Acceptance:** Controlled trials meet the six-second spec bound; real trials commit each interrupted task once and report observed durations; all eligible job work finishes; cleanup is verified.

**Verification:** V-CTEST(worker_crash_stub); V-CTEST(worker_crash_model); preserve artifacts/evidence/m4-worker-crash/.

### T047 — Prove coordinator crash-window recovery

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T045, T046. **Requirements:** TEST-005, DIST-006, DIST-007.

**Files:** `python/tests/recovery/test_coordinator_crash.py`, `docs/test-plan.md`.

**Interface:** Produces crash-window evidence for restart recovery.

**Outcome/scope:** Kill the coordinator at each specified commit boundary, resume the same directory, and inspect authoritative outcomes and attempt history.

**Exclusions:** No power-loss/disk-corruption/replicated-availability claims.

**Acceptance:** Assignment-before-send consumes its attempt; committed-result-before-ACK survives; unfinished tasks recover without duplicate commits and completed tasks are not rerun merely because ACK was lost.

**Verification:** V-CTEST(coordinator_crash); preserve artifacts/evidence/m4-coordinator-crash/.

### T048 — Reject stale and conflicting result traces

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T047. **Requirements:** TEST-005, DIST-005, DIST-008, BUDGET-001.

**Files:** `tests/integration/test_stale_results.cpp`, `python/tests/recovery/test_recovery_matrix.py`, `docs/plan.md`.

**Interface:** Produces final M4 evidence for duplicate/stale/conflicting delivery and recovery invariants.

**Outcome/scope:** Inject prior-attempt/session/epoch results and identical/conflicting duplicate results around retry/restart boundaries; combine with exhaustion and no-worker cases.

**Exclusions:** No new feature expansion or ignoring inconvenient failure windows.

**Acceptance:** Every adversarial trace preserves one authoritative task result, bounded attempts and count invariants; all required M4 cases pass at the reviewed code state.

**Verification:** V-CTEST(stale_results); V-RECOVERY; request M4 acceptance with actual time.

## M5 — Model adaptation and evaluation (16 hours)

### T049 — Train the final candidate reproducibly

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T048, T020. **Requirements:** ML-002, ML-003.

**Files:** `python/defectray_ml/train.py`, `python/defectray_ml/model.py`, `python/tests/fast/test_training_contract.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces a reproducible final-candidate head checkpoint from training data only.

**Outcome/scope:** Complete the bounded CPU head-training run, recording seed, loss, class counts, selected training settings and frozen-backbone verification. Use validation data for selection only.

**Exclusions:** No unrestricted architecture search or full-backbone fine-tuning; no test-set-driven choices.

**Acceptance:** A recorded configuration reproduces the training procedure; head updates and backbone immutability are verified; checkpoint provenance identifies every input split.

**Verification:** V-PYFAST; V-ML(final-train), producing artifacts/ml/final-candidate/.

### T050 — Select the threshold on validation data

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T049. **Requirements:** ML-003, ML-004.

**Files:** `python/defectray_ml/metrics.py`, `python/defectray_ml/evaluate.py`, `python/tests/fast/test_metrics.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces validation confusion counts and a deterministic selected threshold.

**Outcome/scope:** Complete threshold enumeration/selection by F1 with the specified tie-breaks. Handle missing predictions, invalid scores, absent classes and zero denominators explicitly.

**Exclusions:** No threshold tuning using test predictions or substituting accuracy for precision/recall.

**Acceptance:** Hand-calculated synthetic cases match counts/precision/recall/F1 and tie-breaks; validation-selected threshold and inputs are retained.

**Verification:** V-PYFAST; V-ML(validation), producing artifacts/evidence/ml-validation/.

### T051 — Meet the validation quality gate

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T050. **Requirements:** ML-004, ML-005, SCOPE-005.

**Files:** `python/defectray_ml/evaluate.py`, `docs/plan.md`, `docs/test-plan.md`, `python/defectray_ml/__main__.py`.

**Interface:** Produces the validation quality-gate decision and bounded error analysis.

**Outcome/scope:** Compare against always-normal/always-flagged baselines and inspect validation false positives/misses for preprocessing issues. Use only the available ML/contingency budget for justified corrections.

**Exclusions:** No weakening the quality gate, test-set peeking or promised precision/recall.

**Acceptance:** Positive precision/recall and F1 above always-flagged pass; otherwise record an actual failure and request a bounded decision. Any correction reruns the affected validation checks.

**Verification:** V-ML(validation-gate); inspect the resulting error examples and baseline report.

### T052 — Freeze and export the final model bundle

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T051. **Requirements:** ML-006, PROD-006, PROD-010.

**Files:** `python/defectray_ml/export.py`, `python/defectray_ml/bundle.py`, `python/tests/model/test_export.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces immutable artifacts/models/final/ with frozen recipe and threshold.

**Outcome/scope:** Export the final selected graph; bind preprocessing/threshold/labels/provenance to a new bundle identity and run graph/runtime checks.

**Exclusions:** No overwriting the candidate evidence or silently changing an accepted job's bundle.

**Acceptance:** ONNX checker and Python runtime checks pass; all identities/hashes resolve; changed threshold/preprocessing changes bundle identity and cannot masquerade as the prior model.

**Verification:** V-ML(export-final); V-PYMODEL(test_export.py).

### T053 — Verify final-model Python and C++ parity

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T052, T023. **Requirements:** ML-006, ML-007, TEST-001.

**Files:** `python/tools/check_parity.py`, `tests/model/test_inference.cpp`, `docs/test-plan.md`.

**Interface:** Produces final-model numerical and decision parity evidence.

**Outcome/scope:** Rerun all preprocessing/tensor/score checks against the frozen final bundle, adding representative validation images and boundary-output cases.

**Exclusions:** No tolerance relaxation to hide mismatched RGB/layout/resize or threshold drift.

**Acceptance:** Exact shapes, finite outputs, tolerances and decision-margin rules pass for the final artifact; rejected model outputs have explicit failures.

**Verification:** V-PARITY(final); V-CTEST(model_inference); retain artifacts/evidence/m5-parity/.

### T054 — Evaluate the untouched held-out test partition

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T053. **Requirements:** ML-003, ML-004, PERF-005.

**Files:** `python/defectray_ml/evaluate.py`, `python/tests/model/test_evaluation.py`, `python/defectray_ml/__main__.py`.

**Interface:** Produces held-out predictions and metrics at the already frozen threshold.

**Outcome/scope:** Evaluate the untouched official test set, preserving sample IDs, true labels, scores, predictions, counts, bundle hash and evaluation settings.

**Exclusions:** No retraining, preprocessing changes or threshold adjustment after observing these results.

**Acceptance:** Every held-out sample has one accounted evaluation outcome; confusion counts reconcile with class counts; precision/recall and undefined cases are reported honestly.

**Verification:** V-ML(heldout); V-PYMODEL(test_evaluation.py); retain artifacts/evidence/ml-heldout/.

### T055 — Explain false positives and missed defects

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T054. **Requirements:** ML-004, PERF-005.

**Files:** `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces a readable account of model limits based on held-out errors.

**Outcome/scope:** Inspect false positives and missed defects, document the practical limitation of the measured detector and any grouping/data-quality caveats. Keep shared examples within dataset terms.

**Exclusions:** No post-test tuning or claiming industrial certification/calibrated confidence.

**Acceptance:** Written conclusions match stored predictions, include misses and false alarms, and make no unsupported quality claim.

**Verification:** V-EVIDENCE for ml-heldout; manually reconcile cited examples with prediction IDs.

### T056 — Verify the final-model inspection workflow

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T055, T035. **Requirements:** TEST-003, ML-004, PROD-009, BUDGET-001.

**Files:** `python/tests/model/test_end_to_end.py`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Produces final-model two-worker/report acceptance evidence.

**Outcome/scope:** Run the full workflow with the frozen final bundle and controlled mixed inputs; compare committed outputs with the model reference and ensure reports identify the final artifact.

**Exclusions:** No treating M3's early candidate demonstration as final model acceptance.

**Acceptance:** Real two-worker inference, error outcomes, aliases and report counts pass; final model identity/threshold matches held-out evidence; M5 quality gate and evaluation are both recorded.

**Verification:** V-CTEST(e2e_final); V-PYMODEL(test_evaluation.py); request M5 acceptance.

## M6 — Integration and robustness (14 hours)

### T057 — Exercise adversarial TCP behavior

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T056. **Requirements:** NET-001, NET-002, NET-003, NET-004, TEST-002.

**Files:** `tests/integration/test_tcp_adversarial.cpp`, `docs/test-plan.md`.

**Interface:** Produces the combined adversarial real-TCP regression suite.

**Outcome/scope:** Extend protocol checks to zero-length/malformed/truncated frames, wrong field types, interrupted writes, slow readers, unsupported versions and trickle-timeout behavior with the final runtime.

**Exclusions:** No custom fuzzing framework or public hostile-network deployment claim.

**Acceptance:** Each invalid case has a deterministic rejection/close outcome, bounded allocation and no impact on the healthy worker; real stream fragmentation/coalescing still succeeds.

**Verification:** V-CTEST(tcp_adversarial); V-FAST.

### T058 — Stress the resource limits and large collection

**State:** TODO. **Estimate:** 2.5 h. **Actual:** not started.

**Dependencies:** T057. **Requirements:** CPP-003, CPP-004, TEST-006.

**Files:** `tests/integration/test_resource_limits.cpp`, `python/tests/resource/test_large_collection.py`, `docs/test-plan.md`.

**Interface:** Produces byte-cap and collection-size stress evidence.

**Outcome/scope:** Exercise near-limit and over-limit metadata/images/dimensions, queue/output saturation and 1,000 distinct generated PNGs. Check live allocation counters including staged/capacity buffers.

**Exclusions:** No assumption that owned bytes equal total RSS or using duplicate files to fake a 1,000-task workload.

**Acceptance:** Declared caps hold at every stage; oversized headers are rejected before full allocation/decode; the large collection completes with bounded application buffers and recorded RSS.

**Verification:** V-CTEST(resource_limits); V-CTEST(large_collection); preserve artifacts/evidence/m6-resources/.

### T059 — Verify resource lifetimes across repeated cycles

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T058. **Requirements:** CPP-001, CPP-002, CPP-006, TEST-006.

**Files:** `tests/integration/test_resource_lifecycle.cpp`, `docs/test-plan.md`.

**Interface:** Produces repeatable resource-ownership and shutdown-cycle checks.

**Outcome/scope:** Repeat queue closure, connect/disconnect and controlled in-flight shutdown; compare owned allocation and descriptor counts with the defined warmed idle baseline.

**Exclusions:** No treating library allocator retention as an application leak without evidence.

**Acceptance:** No deadlock, dangling buffer or application descriptor growth appears; joins and cancellation completions precede destruction; recorded limitations distinguish native hangs.

**Verification:** V-CTEST(resource_lifecycle); V-FAST.

### T060 — Run supported completed-code sanitizer checks

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T059. **Requirements:** TEST-006, CPP-001, CPP-002.

**Files:** `docs/test-plan.md`, `docs/plan.md`, `CMakeLists.txt`.

**Interface:** Produces sanitizer evidence for the completed supported application paths.

**Outcome/scope:** Run separate ASan+UBSan and TSan configurations over ownership, queue, network and lifecycle cases; investigate actual findings and apply only in-task corrections.

**Exclusions:** No blanket suppressions, removing assertions or claiming uninstrumented libraries were checked.

**Acceptance:** Required supported suites pass at the recorded code state; toolchain/runtime restrictions or unresolved findings block readiness and are documented.

**Verification:** V-ASAN and V-TSAN with the established selections.

### T061 — Verify the user-visible failure matrix

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T060. **Requirements:** PROD-001, PROD-003, PROD-007, PROD-008, PROD-009, TEST-003.

**Files:** `python/tests/fast/test_failure_outcomes.py`, `tests/integration/test_cli_errors.cpp`, `docs/test-plan.md`.

**Interface:** Produces a complete user-visible error/outcome matrix.

**Outcome/scope:** Check rejected submissions, interrupted imports, missing inputs, no workers, retry exhaustion, mixed/all failures, bad output locations and renderer failure using controlled fixtures.

**Exclusions:** No new formats or success labels for incomplete work.

**Acceptance:** Each case gives the specified exit/outcome and preserves accepted state; reports account for every admitted unique input and clearly mark unfinished work.

**Verification:** V-CTEST(cli_errors); V-PYFAST; V-CTEST(e2e_final).

### T062 — Reject changed inputs and model identities

**State:** TODO. **Estimate:** 1.5 h. **Actual:** not started.

**Dependencies:** T061. **Requirements:** PROD-003, PROD-005, PROD-006, NET-005, ML-006.

**Files:** `tests/integration/test_artifact_identity.cpp`, `python/tests/fast/test_bundle_identity.py`.

**Interface:** Produces immutable-input/model identity regression evidence.

**Outcome/scope:** Modify copied input bytes, model/preprocessing/threshold metadata and worker bundles in isolated fixtures; attempt dispatch/resume/registration.

**Exclusions:** No silently updating an existing job's frozen model or reclassifying old outputs.

**Acceptance:** Tampered/missing inputs fail explicitly; changed bundle identities reject incompatible workers/resume; no stale result gains acceptance under a new model identity.

**Verification:** V-CTEST(artifact_identity); V-PYFAST.

### T063 — Complete the required verification matrix

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T062. **Requirements:** TEST-001, TEST-002, TEST-003, TEST-004, TEST-005, TEST-006, TEST-007, BUDGET-001.

**Files:** `.github/workflows/ci.yml`, `tests/CMakeLists.txt`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Produces an auditable complete-check matrix and fast/expensive suite separation.

**Outcome/scope:** Verify fast PR selection has real behavioral coverage and no data/model downloads; explicitly run model/recovery/resource suites locally. Re-run affected CI after final configuration changes.

**Exclusions:** No treating skipped model/sanitizer checks or compilation alone as acceptance.

**Acceptance:** All required checks show actual pass/fail evidence at one recorded code state; full suite selections cannot be empty; no mandatory gap remains for M6.

**Verification:** V-FAST V-PYFAST V-PYMODEL(all) V-RECOVERY V-RESOURCE V-FORMAT V-ANALYZE, required sanitizers, and authorized CI evidence; request M6 acceptance.

## M7 — Performance investigation (18 hours)

### T064 — Freeze the benchmark workload and controls

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T063. **Requirements:** PERF-002, PERF-003.

**Files:** `python/tools/benchmark.py`, `python/tests/fast/test_benchmark_manifest.py`, `docs/benchmarks.md`.

**Interface:** Produces a frozen performance workload and environment/configuration manifest.

**Outcome/scope:** Select at least 100 unique images; capture hardware/OS/WSL, compiler/build, model, CPU budget, thread counts, workload order and repetition/warm-up settings before experiments.

**Exclusions:** No mixed training/evaluation workloads, extra hardware disguised as optimization, or performance claim.

**Acceptance:** Manifest validation catches mismatched model/workload/budget; generated/transformed workloads are labeled and kept separate from held-out evaluation.

**Verification:** V-PYFAST; V-BENCH(manifest).

### T065 — Measure throughput and p95 with explicit boundaries

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T064. **Requirements:** PERF-002, PROD-009.

**Files:** `src/common/timing.hpp`, `src/coordinator/scheduler.cpp`, `src/worker/runtime.cpp`, `python/tools/benchmark.py`, `python/tests/fast/test_benchmark_metrics.py`.

**Interface:** Produces model-only and pipeline throughput/latency measurements with declared boundaries.

**Outcome/scope:** Instrument admitted-task-to-commit timing separately from worker preprocessing/inference stages. Compute p95 using a recorded method and unique successful task counts; report startup/import/report times separately.

**Exclusions:** No counting retries as throughput or comparing monotonic timestamps across different hosts/runs.

**Acceptance:** Hand-calculated samples validate p95/counts/durations; failed tasks and retry attempts cannot inflate success throughput; known injected queue delay is visible in end-to-end latency.

**Verification:** V-PYFAST; V-CTEST(timing_boundaries); V-FAST.

### T066 — Measure process memory separately from owned buffers

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T065. **Requirements:** PERF-002, CPP-003, CPP-004.

**Files:** `python/tools/benchmark.py`, `python/tests/fast/test_memory_metrics.py`, `docs/benchmarks.md`.

**Interface:** Produces per-process peak RSS plus application-owned byte high-water marks.

**Outcome/scope:** Measure coordinator/worker RSS with an available verified Linux method and document whether it is sampled or a kernel high-water mark. Record units and process identity.

**Exclusions:** No claiming sampled maximum is an exact high-water mark or summing separate peaks as simultaneous system memory.

**Acceptance:** Known-allocation checks validate units/process attribution; output separates library-inclusive process memory from owned buffers and explains collection limitations.

**Verification:** V-PYFAST; V-BENCH(memory-check).

### T067 — Establish the repeated baseline

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T066. **Requirements:** PERF-002, PERF-003, TEST-007.

**Files:** `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces the reproducible performance baseline before optimization.

**Outcome/scope:** Use Release builds, the frozen final bundle/workload, 20 warm-up inferences and five measured runs per declared configuration. Preserve each run and failures.

**Exclusions:** No shared-runner authoritative timings or comparing unlike CPU budgets.

**Acceptance:** Raw timings, environment, RSS, owned bytes and per-run summaries are complete and independently recomputable; baseline code-state identity is recorded before changes.

**Verification:** V-RELEASE; V-BENCH(baseline); retain artifacts/evidence/perf-baseline/.

### T068 — Profile and record one optimization hypothesis

**State:** TODO. **Estimate:** 3 h. **Actual:** not started.

**Dependencies:** T067. **Requirements:** PERF-001, PERF-004.

**Files:** `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces a supported bottleneck hypothesis with profiler/stage evidence.

**Outcome/scope:** Check available profiling permissions/tools; capture a representative workload. Prefer ONNX/runtime profiling and stage timings if hardware profiling is unavailable. Select one bounded experiment.

**Exclusions:** No installing unavailable profilers without permission or speculative batching/pool work.

**Acceptance:** The hypothesis names measured evidence, expected effect, controlled variables and pass/fail comparison. Tool limitations are explicit.

**Verification:** V-BENCH(profile); record the actual available profiler command and raw output.

### T069 — Attempt one bounded evidence-driven optimization

**State:** TODO. **Estimate:** 3 h. **Actual:** not started.

**Dependencies:** T068. **Requirements:** PERF-001, CPP-005, SCOPE-005.

**Files:** `src/config/runtime_config.cpp`, `src/inference/engine.cpp`, `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces one evaluated optimization candidate under the same budget.

**Outcome/scope:** Prefer a recorded ONNX thread-count/worker-count configuration change. If profiling instead justifies a small reusable buffer change, preserve ownership/caps and add the relevant behavioral regression check.

**Exclusions:** No mandatory dynamic batching/custom pools, weakening durability, model changes or broad redesign.

**Acceptance:** The selected candidate is actually exercised; correctness/regression checks pass before timing. A neutral or regressive attempt still satisfies the attempt requirement when honestly recorded.

**Verification:** V-FAST V-PYFAST V-PARITY(final) as affected; V-BENCH(candidate).

### T070 — Compare the candidate fairly

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T069. **Requirements:** PERF-001, PERF-003, PERF-005.

**Files:** `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces a fair baseline-versus-candidate comparison with uncertainty.

**Outcome/scope:** Run five measured repetitions under identical workload/model/hardware budget and the same warm-up. Preserve output equivalence, failures, variation and setup differences.

**Exclusions:** No cherry-picking a best run or changing concurrency without reporting total resource budget.

**Acceptance:** Reported median/range and any percentage are recomputable from all runs; correctness is equivalent; an unhelpful candidate is rejected without inventing gains.

**Verification:** V-BENCH(compare); retain artifacts/evidence/perf-comparison/.

### T071 — Publish the benchmark evidence and limitations

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T070. **Requirements:** PERF-001, PERF-002, PERF-003, PERF-004, PERF-005, BUDGET-001.

**Files:** `docs/benchmarks.md`, `docs/plan.md`.

**Interface:** Produces the complete M7 evidence report and protected-time accounting.

**Outcome/scope:** Combine throughput, p95, memory and recovery definitions/results with profiling conclusions. Clearly distinguish model-only/pipeline and controlled-stub/real recovery results.

**Exclusions:** No missing failed runs, exactly-once/production claims, or positive optimization promise.

**Acceptance:** Every number links to raw evidence, code/config identity and measurement boundary; limitations and actual 18-hour allocation use are recorded.

**Verification:** V-EVIDENCE for performance and recovery artifacts; request M7 acceptance.

## M8 — Reproduction and presentation (10 hours)

### T072 — Reproduce the final system from clean source

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T071. **Requirements:** BUDGET-003, TEST-007, SCOPE-002.

**Files:** `README.md`, `docs/test-plan.md`, `docs/plan.md`.

**Interface:** Produces final fresh-source reproduction evidence.

**Outcome/scope:** Use a user-provided clean checkout or source-only temporary copy and follow documented dependency/build/test/model-acquisition steps with necessary permissions. Record hidden prerequisites and correct instructions.

**Exclusions:** No assistant-created branch/worktree/commit or silently acquiring large artifacts.

**Acceptance:** The documented workflow reproduces with pinned dependencies and licensed artifacts, or an actual external prerequisite is marked blocked; no reliance on unrelated global files remains.

**Verification:** V-REPRO using recorded README commands; preserve the setup transcript.

### T073 — Provide a repeatable local demonstration

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T072. **Requirements:** SCOPE-001, PROD-004, TEST-004.

**Files:** `python/tools/demo.py`, `README.md`, `docs/test-plan.md`.

**Interface:** Produces one simple local demonstration launcher over the existing CLI and owned-process harness.

**Outcome/scope:** Launch coordinator/two workers, submit a small authorized collection, show reports, and offer an explicit controlled worker-crash demonstration using the existing harness.

**Exclusions:** No Docker/cloud service orchestration, restart manager or broad process kills.

**Acceptance:** Another person can repeat normal and crash demonstrations from the instructions; process cleanup and all displayed claims match recorded results.

**Verification:** V-DEMO(normal); V-DEMO(worker-crash).

### T074 — Finalize verified usage and troubleshooting

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T073. **Requirements:** PROD-004, PROD-007, PROD-008, PROD-009, BUDGET-005.

**Files:** `README.md`, `docs/test-plan.md`, `docs/benchmarks.md`.

**Interface:** Produces final setup/usage/troubleshooting documentation with consistent canonical ownership.

**Outcome/scope:** Explain submit/worker/status/resume/report, artifact locations, outcomes, limits and honest failure/recovery assumptions. Link checks and experiments rather than duplicating their authoritative instructions.

**Exclusions:** No unsupported features, guessed commands or duplicated progress tracking.

**Acceptance:** Every usage example was verified; the guide names same-machine WSL/Linux scope, model limits, acquisition requirements and expected error behavior.

**Verification:** V-DOCS; spot-check documented examples against the saved transcripts.

### T075 — Assemble evidence-backed CV statements

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T074. **Requirements:** PERF-005, ML-004, BUDGET-005.

**Files:** `docs/benchmarks.md`, `README.md`, `docs/plan.md`.

**Interface:** Produces a shareable evidence index and three or four defensible CV bullets.

**Outcome/scope:** Map ML, throughput/latency, memory/concurrency and crash-recovery statements to retained evidence. Exclude restricted images/weights from shared artifacts and clearly mark negative optimization results.

**Exclusions:** No invented improvements, customers, production use, multi-host results or exactly-once claims.

**Acceptance:** Every factual CV statement is traceable to code, test or measurement evidence; any percentage has a comparable baseline; sharing respects recorded artifact terms.

**Verification:** V-EVIDENCE; inspect the proposed CV wording against the evidence index.

### T076 — Review final readiness and request acceptance

**State:** TODO. **Estimate:** 2 h. **Actual:** not started.

**Dependencies:** T075. **Requirements:** SCOPE-001, SCOPE-003, SCOPE-005, TEST-007, BUDGET-001, BUDGET-002, BUDGET-004, BUDGET-005.

**Files:** `docs/plan.md`, `docs/test-plan.md`, `docs/benchmarks.md`, `README.md`.

**Interface:** Produces the final review/acceptance packet without assuming acceptance.

**Outcome/scope:** Review all mandatory requirement coverage, actual hours, unrun checks, final code-state evidence and known limitations. Run only checks invalidated by later changes; present readiness to the user.

**Exclusions:** No automatic DONE, next task, release, push or stretch-goal work.

**Acceptance:** No mandatory check remains unrun/failed for readiness; the user explicitly accepts completion. If the budget ends first, report remaining work accurately instead of marking DONE.

**Verification:** V-FINAL: reconcile the complete evidence matrix and re-run only affected required checks.

## 7. Requirement coverage

The mapping shows planned ownership, not evidence of completion. Requirement definitions remain in spec.md.

| Requirement | Planned task coverage |
|---|---|
| ARCH-001 | T024, T029 |
| ARCH-002 | T024 |
| ARCH-003 | T028, T031, T041, T042 |
| ARCH-004 | T026 |
| ARCH-005 | T031, T035 |
| BUDGET-001 | T016, T023, T035, T048, T056, T063, T071, T076 |
| BUDGET-002 | T016, T076 |
| BUDGET-003 | T001, T002, T003, T004, T005, T006, T007, T008, T009, T010, T011, T012, T013, T014, T015, T016, T072 |
| BUDGET-004 | T016, T076 |
| BUDGET-005 | T016, T074, T075, T076 |
| CPP-001 | T010, T011, T022, T024, T026, T027, T028, T029, T044, T059, T060 |
| CPP-002 | T012, T027, T028, T044, T059, T060 |
| CPP-003 | T019, T022, T025, T026, T027, T039, T058, T066 |
| CPP-004 | T058, T066 |
| CPP-005 | T022, T028, T069 |
| CPP-006 | T028, T044, T059 |
| DIST-001 | T036, T043 |
| DIST-002 | T036, T037, T040 |
| DIST-003 | T031, T037 |
| DIST-004 | T040, T046 |
| DIST-005 | T038, T048 |
| DIST-006 | T042, T047 |
| DIST-007 | T029, T030, T037, T042, T047 |
| DIST-008 | T038, T046, T048 |
| GATE-001 | T003, T013, T023 |
| GATE-002 | T017, T023 |
| GATE-003 | T003, T004, T013, T017 |
| ML-001 | T017, T018 |
| ML-002 | T017, T020, T049 |
| ML-003 | T018, T020, T049, T050, T054 |
| ML-004 | T050, T051, T054, T055, T056, T075 |
| ML-005 | T019, T022, T051 |
| ML-006 | T021, T022, T023, T052, T053, T062 |
| ML-007 | T019, T020, T023, T053 |
| NET-001 | T025, T057 |
| NET-002 | T025, T031, T038, T057 |
| NET-003 | T025, T026, T039, T057 |
| NET-004 | T039, T040, T041, T043, T057 |
| NET-005 | T031, T041, T062 |
| PERF-001 | T068, T069, T070, T071 |
| PERF-002 | T064, T065, T066, T067, T071 |
| PERF-003 | T064, T067, T070, T071 |
| PERF-004 | T068, T071 |
| PERF-005 | T054, T055, T070, T071, T075 |
| PROD-001 | T030, T061 |
| PROD-002 | T030 |
| PROD-003 | T030, T061, T062 |
| PROD-004 | T032, T034, T042, T073, T074 |
| PROD-005 | T024, T032, T042, T062 |
| PROD-006 | T021, T024, T038, T052, T062 |
| PROD-007 | T029, T032, T033, T036, T043, T061, T074 |
| PROD-008 | T032, T034, T043, T044, T061, T074 |
| PROD-009 | T033, T034, T056, T061, T065, T074 |
| PROD-010 | T002, T021, T029, T030, T033, T052 |
| SCOPE-001 | T032, T035, T073, T076 |
| SCOPE-002 | T001, T004, T005, T072 |
| SCOPE-003 | T001, T016, T076 |
| SCOPE-004 | T002 |
| SCOPE-005 | T016, T051, T069, T076 |
| TEST-001 | T007, T025, T027, T036, T053, T063 |
| TEST-002 | T026, T057, T063 |
| TEST-003 | T035, T056, T061, T063 |
| TEST-004 | T045, T046, T063, T073 |
| TEST-005 | T045, T047, T048, T063 |
| TEST-006 | T011, T012, T058, T059, T060, T063 |
| TEST-007 | T001, T006, T007, T009, T010, T014, T015, T016, T045, T063, T067, T072, T076 |

## 8. Review, time and acceptance record

| Date | Scope | Record |
|---|---|---|
| 2026-10-25 | Written specification | Explicit user approval: “spec approve”. |
| 2026-10-25 | Implementation plan | Draft prepared; awaiting separate approval. |
| 2026-10-25 | Implementation tasks | None started, verified, reviewed or accepted. |
| 2026-11-05 | Plan, test/benchmark procedures and working agreement | User: “i approve all, i dont want to spend time on docs, i want to start with tasks and practice”. Approval recorded; prioritize practical briefings and assistant-maintained progress records. |
| 2026-11-05 | T001 | Briefed; IN_PROGRESS. Awaiting the user's read-only Ubuntu-terminal inventory. No checks reviewed or task acceptance recorded. |
| 2026-11-05 | T001 review | User reported start at 17:17 Asia/Jerusalem and supplied V-ENV output. Reviewed through IN_REVIEW to READY_FOR_ACCEPTANCE; inventory evidence recorded in test-plan.md and setup limitations in README.md. No blocking inventory findings; acceptance and elapsed time are pending. |
| 2026-11-05 | T001 acceptance | User: “accept T001”. Marked DONE; elapsed effort remains unreported. T002–T076 remain TODO. |
| 2026-11-05 | T002 briefing | User: “start T002”. Marked IN_PROGRESS; practical README layout and .gitignore exercise briefed. Awaiting human implementation and V-HYGIENE evidence; actual time remains unreported. |
| 2026-11-05 | T002 delegated implementation and review | User requested assistant implementation. README layout and .gitignore added; V-HYGIENE passed. READY_FOR_ACCEPTANCE; human effort unreported, acceptance pending. |

No contingency has been allocated to a task. Record actual use against the affected task and remaining reserve when an overrun occurs; do not alter estimates retroactively to conceal it.

**Next action:** await explicit user acceptance of T002. T003 remains TODO pending a request to start it. Actual human effort can be recorded when reported; assistant execution is not recorded as human practice time.
