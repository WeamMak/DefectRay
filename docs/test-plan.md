# DefectRay test plan and verification register

Status: **PROCEDURES APPROVED ON 2026-11-05 — T001/T002 VERIFICATION RECORDED; NO APPLICATION CHECKS RUN**

Date: 2026-10-25

The [approved specification](spec.md) owns requirements and limits. The [implementation plan](plan.md) owns task status, review findings, fixes, actual hours and user acceptance. This document owns verification recipes, fixture definitions and actual command records as implementation proceeds.

## 1. Present state and command contract

The repository currently has planning documents, environment notes, Git ignore rules and working instructions, with no application CMake configuration, Python project, test suites, executable targets or CI workflow. T001's user-run environment inventory and T002's assistant-run Git hygiene checks are recorded in section 6; they are not evidence of a passing application implementation. Model and dependency acquisition are still gated.

Application/build/test command recipes below are **proposed interfaces for the human to create in their named task**, not existing repository commands. The read-only V-ENV inventory uses existing system tools; its actual user-run results are recorded in section 6. A task must register its tests/targets and verify the command before dependent tasks use it. If the implementation requires a different valid command, update the verified recipe here and its plan references; do not invent a passing run or change required behavior. No CMake presets are assumed.

Notation such as `V-CTEST(frame_codec)` is a documentation reference, not a shell function. Replace its test-name argument with the exact task-specified name. The C++ recipe then uses an anchored CTest name selection with failure on no tests. A new test task also updates `tests/CMakeLists.txt` as needed and assigns the relevant labels.

Run future commands from the actual Linux repository root. The user verified `/home/weam/projects/DefectRay` during T001 on 2026-11-05. `.venv/bin/python` means the project interpreter selected only after T003. `PYTHONPATH=python` makes the planned local modules importable without relying on a global installation. Native dependency versions and interpreter/lock/install commands are intentionally unresolved until the approved compatibility task supplies evidence.

Before recording a pass, retain the task ID, command/cwd/environment, exit status, expected behavioral assertion, code commit plus uncommitted diff identity, fixture/bundle hashes, and output/evidence path. A suite that is absent, empty, skipped, or cannot execute a required case is not passing evidence. Task state/acceptance remains exclusively in plan.md.

## 2. Infrastructure recipes

### V-ENV — T001; toolchain recheck T004

Read-only inventory commands, to be executed by the human during the approved task:

```bash
pwd
git rev-parse --show-toplevel
git status --short
uname -srmo
cat /etc/os-release
lscpu
free -h
command -v g++ cmake ctest gdb clang-format clang-tidy python3
```

Check version output only for tools actually found. Missing-tool output is part of the inventory, not a passing toolchain check. Detect Windows paths/tools and keep them out of the build. Inventory profiler availability without granting performance-tool access or installing anything. No Git initialization is part of this check.

During T004, if GCC is the selected compiler, `g++ -std=c++20 -dM -E -x c++ /dev/null` must accept the mode and report `__cplusplus` as 202002L. Record the equivalent verified command if a different supported compiler is selected. T005 subsequently verifies an actual C++20 build; a version string alone is not that evidence.

### V-HYGIENE — T002

```bash
git check-ignore -v -- build/debug/CMakeCache.txt .venv/pyvenv.cfg data/example.png models/example.onnx artifacts/jobs/example/state.sqlite artifacts/evidence/example/run.json
git check-ignore -- src/main.cpp tests/unit/test_smoke.cpp tests/fixtures/tiny.png tests/fixtures/expected.json python/defectray_ml/train.py docs/benchmarks.md README.md .gitignore
git status --short
```

The generated paths should be ignored. The source/fixture/documentation check should print nothing and return 1 because those paths must remain trackable. Do not interpret this expected non-match as a failed hygiene policy. No large artifact need be created for this check.

### V-MATRIX — T003

Review primary, release-specific support metadata and record the actual compatible versions and source links here before selecting Python. Include torch/vision pairing, ONNX exporter dependencies/opset/IR, Python and native OpenCV, ONNX Runtime CPU wheels/C++ ABI, and native build/test dependencies. A moving latest-docs link alone is insufficient. This is document verification; installation and runtime compatibility are verified later.

### V-DEBUG and V-RELEASE — T005/T006

Proposed commands after the corresponding CMake configuration exists:

```bash
cmake -S . -B build/debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build/debug --parallel 2
cmake -S . -B build/release -DCMAKE_BUILD_TYPE=Release
cmake --build build/release --parallel 2
```

T005 verifies only the Debug pair; T006 adds Release. Preserve separate output directories. The planned application executable is `build/debug/defectray` (or `build/release/defectray`). No working application is inferred from the minimal smoke executable.

### V-CTEST(name), V-FAST — T007 and each producing test task

The single-test recipe is `ctest --test-dir build/debug -R '^name$' --output-on-failure --no-tests=error`, with `name` replaced by the exact registered name in the task. It is notation, not a literal test named `name`.

```bash
cmake --build build/debug --parallel 2
ctest --test-dir build/debug -N
ctest --test-dir build/debug -L fast --output-on-failure --no-tests=error
```

Verify the selected CMake/CTest supports these options in T007. Registration/listing confirms nonempty suites. C++ unit checks remain active in Release. `fast` includes generated-fixture unit and short loopback checks; expensive model/recovery/resource suites have separate labels. Do not use no-tests success as evidence.

### V-DEBUGGER — T008

Build Debug, launch the Linux debugger through the workspace VS Code configuration, hit a breakpoint in the minimal program, inspect a variable and call frame, then exit cleanly. Retain the launch configuration/tool paths and the human's observed result. This manual check cannot be replaced by compilation alone.

### V-FORMAT and V-ANALYZE — T009/T010

These targets are proposed and do not exist yet:

```bash
cmake --build build/debug --target format-check
cmake --build build/debug --target analyze
```

T009/T010 provide the targets and verified tool-specific commands. Check maintained source only; record intentional third-party exclusions. A formatting check must be read-only. Demonstrate a relevant seeded failure before restoring the clean source; do not keep a failing fixture in the passing suite.

### V-ASAN and V-TSAN — T011/T012; full checks T060

The human introduces the following CMake option values in the named tasks:

```bash
cmake -S . -B build/asan -DCMAKE_BUILD_TYPE=Debug -DDEFECTRAY_SANITIZER=address-undefined
cmake --build build/asan --parallel 2
ctest --test-dir build/asan -L fast --output-on-failure --no-tests=error
cmake -S . -B build/tsan -DCMAKE_BUILD_TYPE=Debug -DDEFECTRAY_SANITIZER=thread
cmake --build build/tsan --parallel 2
ctest --test-dir build/tsan -L fast --output-on-failure --no-tests=error
```

Keep these configurations separate. First verify detection with controlled temporary negative fixtures. Then restore passing fixtures and run clean tests. Include the queue/network/lifecycle checks as they become available. A task referring to one test under a sanitizer uses the same directory with an anchored name selection. Record uninstrumented-library limitations and actual runtime restrictions; unavailable mandatory checks block readiness.

### V-PYENV and V-PYFAST — T013/T014

After T003 selects the interpreter and T013 creates/locks the environment:

```bash
.venv/bin/python --version
.venv/bin/python -m pip check
PYTHONPATH=python .venv/bin/python -m unittest discover -s python/tests/fast -p 'test_*.py'
```

Record the exact environment-creation, approved dependency installation and lock-reproduction commands here when known. Do not substitute system Python 3.14 by default. Require discovery of the expected nonzero test count and no acquisition/training side effects. The lock method must pin transitive dependencies and their sources reproducibly.

### V-FOUNDATION — T016

Use the now-verified local recipes, Debug/Release smoke behavior, manual debugger observation, the Python environment reconstruction, and the actual user-authorized CI run. A local pass is not a claim of CI success. Record all results and actual M1 time in plan.md. The fresh-source check uses a user-provided checkout or an explicitly scoped source-only temporary copy; no automatic branch/worktree/commit is authorized.

## 3. ML recipes and acquisition gates

### V-ACQUIRE — T017

Resolve artifact permission before download. Record official source, exact weight identity, license/terms, attribution, educational-use and redistribution conclusions, archive/weight SHA-256, layout and label mapping. The human authorizes any acquisition separately; then retain the actual download/integrity commands. Unknown permission is a blocker, not a license assumption. Store full datasets/weights outside Git.

### V-ML(operation) — T018 onward

The producing task adds the named CLI operation to `python/defectray_ml/__main__.py`. The following are proposed commands and output conventions:

| Recipe | Producing task | Proposed command suffix after `PYTHONPATH=python .venv/bin/python -m defectray_ml` |
|---|---|---|
| splits | T018 | `splits --data data/kolektorsdd2 --out artifacts/ml/splits` |
| fixtures | T019 | `fixtures --out artifacts/fixtures/parity` |
| candidate-train | T020 | `train --config artifacts/config/candidate-train.json` |
| export-candidate | T021 | `export --checkpoint artifacts/ml/candidate/checkpoint.pt --out artifacts/models/candidate` |
| final-train | T049 | `train --config artifacts/config/final-train.json` |
| validation | T050 | `evaluate --config artifacts/config/validation.json` |
| validation-gate | T051 | `validation-gate --report artifacts/evidence/ml-validation/summary.json` |
| export-final | T052 | `export --checkpoint artifacts/ml/final-candidate/checkpoint.pt --out artifacts/models/final` |
| heldout | T054 | `evaluate --config artifacts/config/heldout.json` |

Each task creates and documents its own configuration/schema before invoking it. Configuration records actual local dataset/weight paths from acquisition, manifests, seed, CPU settings and output directory; it cannot trigger a hidden weight download. Candidate and final evidence are separate. Evaluation configs pin the already selected artifact and split; heldout config cannot choose a threshold from test results.

### V-PYMODEL(test file or all) — T021 onward

Model suites require a real, verified local bundle. Select it explicitly rather than falling back to another model:

```bash
DEFECTRAY_TEST_BUNDLE="$PWD/artifacts/models/candidate" PYTHONPATH=python .venv/bin/python -m unittest discover -s python/tests/model -p 'test_export.py'
```

For the final model, use `artifacts/models/final`. Replace the pattern with `test_evaluation.py` or `test_*.py` only when those required tests have been created. Missing required artifacts/test cases must fail the explicitly requested model suite, not silently skip it. Fast PR checks never select this suite by accident.

### V-PARITY(candidate or final) — T023/T053

T022 creates `build/debug/infer_one`; T023 creates the comparison tool. Proposed candidate command:

```bash
PYTHONPATH=python .venv/bin/python python/tools/check_parity.py --bundle artifacts/models/candidate --fixtures artifacts/fixtures/parity --cpp build/debug/infer_one --out artifacts/evidence/m2-parity
```

T053 selects `artifacts/models/final` and `artifacts/evidence/m5-parity`. Compare at least eight generated fixtures plus a real image. Assert the approved tensor/score tolerances and record near-threshold cases separately. Record Python/runtime/native library versions, graph shape/opset/IR, hashes and actual output. The early checkpoint does not establish final ML quality.

## 4. Functional, crash and resource suites

Each `V-CTEST(...)` reference in plan.md names a test registered by the task that introduces it. `model_inference`, `worker_runtime_model`, `e2e_candidate`, `e2e_final`, and `worker_crash_model` require the corresponding explicit `DEFECTRAY_TEST_BUNDLE` environment variable. Use the final bundle from T052 onward when checking the completed system. The CMake harness must pass this selection to its Python/native children.

### V-HARNESS — T045

Use `V-CTEST(process_harness)`. Verify exact event barriers, tracked child PIDs, isolated job directories, monotonic fault/event timestamps and cleanup on both success and failure. Store failure injection only in test builds/adapters; normal production inference has no deliberate delay. Temporary fixture cleanup cannot delete unrelated evidence or data.

### V-RECOVERY — T046–T048; final rerun T063

```bash
ctest --test-dir build/debug -L recovery --output-on-failure --no-tests=error
```

Required cases are five controlled worker kills, five real-model worker kills, worker re-registration, coordinator termination after assignment commit/before send, termination after result commit/before ACK, duplicate/stale/conflicting results, retry exhaustion, and all-workers-unavailable pause/resume. The interrupted task's successful durable commit is the recovery endpoint. For model cases, set the explicit candidate/final bundle as above. No claim of power-loss durability or multi-host recovery follows from these checks.

### V-RESOURCE — T058–T059

```bash
ctest --test-dir build/debug -L resource --output-on-failure --no-tests=error
```

Use synthetic bounded PNG fixtures, including 1,000 distinct byte contents/tasks, near-limit and over-limit headers, bounded request/output saturation, repeated connect/disconnect and shutdown. Assert owned allocation/descriptor behavior separately from RSS. Do not preload the generated collection into worker memory to run the test.

### Fixture ownership

| Fixture | Purpose | Source/retention |
|---|---|---|
| Tiny synthetic frames/tensors | Parser, state, queue, metric tests | Small source-controlled definitions; no dataset download |
| Deterministic generated PNGs | Preprocessing, invalid-header, cap and volume checks | Generation code in source; generated bulk data ignored |
| Candidate/final model bundle | Native parity, real inference/end-to-end/recovery | Verified local artifact; acquisition permission/provenance required |
| Labeled dataset samples | ML evaluation and error inspection | Official split/terms; do not put restricted images in Git |
| Isolated job directories | Import, resume, corruption and crash checks | Owned by harness; retain diagnostics, clean only its temporary data |

## 5. Benchmarks, demonstration and final checks

**V-BENCH(operation)** is defined in [benchmarks.md](benchmarks.md); T064 introduces its tool. Measurements are not fast PR correctness tests.

### V-REPRO — T072

Use a source-only clean environment and the actual verified README commands. Confirm lock/dependency acquisition, native build, fast checks and explicitly invoked model suites. Record the real transcript. Any download/install or remote checkout action needs the user's instruction; the assistant does not perform it as a side effect of planning/review.

### V-DEMO(normal or worker-crash) — T073

Proposed commands after `python/tools/demo.py` is implemented:

```bash
PYTHONPATH=python .venv/bin/python python/tools/demo.py --scenario normal --bundle artifacts/models/final
PYTHONPATH=python .venv/bin/python python/tools/demo.py --scenario worker-crash --bundle artifacts/models/final
```

The launcher reuses real CLI and owned-process lifecycle behavior. Its fixture collection and expected outcomes are documented by T073. No public listener or unrelated process termination is permitted.

### V-DOCS, V-EVIDENCE, V-FINAL — T074–T076

V-DOCS checks documented examples against actual execution transcripts and reconciles links/paths with the implementation. V-EVIDENCE checks file presence/hashes, code/config identity, counts, raw-to-summary arithmetic and whether claimed measurements follow their declared boundaries. Do not confuse either with executing the application.

V-FINAL reconciles all mandatory checks and their code-state identities. Later relevant code changes require affected checks again; unrelated documentation edits do not justify repeating expensive experiments. Required failing/unrun/skipped cases prevent readiness. Present limitations and actual time, then wait for explicit user acceptance.

## 6. Verification record

At creation: no environment task, compilation, application test, model evaluation, crash trial or benchmark has been executed for this plan. Only the planning documents themselves have been checked for references, dependency order, hour totals and formatting. Populate actual command/version/evidence records during the authorized tasks; retain task review/acceptance in plan.md.

### T001 environment inventory (2026-11-05)

Source: commands run by the user in their Ubuntu terminal and pasted into the conversation. Start time reported as 17:17, interpreted in the user's Asia/Jerusalem timezone. Finish time and elapsed effort were not supplied. The assistant reviewed and recorded the transcript without rerunning the environment inventory or installing tools. Numeric shell exit statuses were not supplied; the table records actual visible output and the inventory conclusion. This verifies inventory facts, not a working build environment.

| Actual command | Observed output / inventory result |
|---|---|
| `pwd` | `/home/weam/projects/DefectRay` — Linux project path confirmed |
| `git rev-parse --show-toplevel` | `/home/weam/projects/DefectRay` — Git recognizes the repository at that path |
| `git status --short` | ` M AGENTS.md` and `?? docs/` — existing uncommitted documentation work; preserve it |
| `uname -srmo` | `Linux 6.6.87.2-microsoft-standard-WSL2 x86_64 GNU/Linux` |
| `cat /etc/os-release` | Ubuntu 26.04 LTS, Resolute Raccoon, `ID=ubuntu`, `VERSION_ID=26.04` |
| `lscpu` | AMD Ryzen 9 8940HX with Radeon Graphics; x86_64; 32 logical CPUs; 16 reported cores; 2 threads/core; 1 socket; Microsoft hypervisor; 1 NUMA node |
| `free -h` | RAM total 15 GiB, used 2.5 GiB, available 12 GiB; swap total 4.0 GiB, used 0 B at the observation time |
| `command -v g++ cmake ctest gdb clang-format clang-tidy python3` | Only `/usr/bin/g++` and `/usr/bin/python3` printed; the other requested tools were not found on this PATH |
| `command -v perf` | No output — perf was not found on this PATH |
| `g++ --version` | `g++ (Ubuntu 15.2.0-16ubuntu1) 15.2.0` |
| `cmake --version` | Command not found. Shell package suggestions were displayed; no installation was performed |
| `python3 --version` | `Python 3.14.4` |

Available tool/version inventory:

| Tool | Status at observation | Path/version evidence |
|---|---|---|
| Git | Working at the actual repository root | Repository queries succeeded visibly; Git version was not requested/reported |
| GCC C++ compiler | Found | `/usr/bin/g++`, 15.2.0 |
| System Python | Found | `/usr/bin/python3`, 3.14.4; not selected as the project version |
| CMake | Not found on PATH | Discovery omitted it; direct version invocation reported command not found |
| CTest | Not found on PATH | Absent from combined discovery output |
| GDB | Not found on PATH | Absent from combined discovery output |
| clang-format | Not found on PATH | Absent from combined discovery output |
| clang-tidy | Not found on PATH | Absent from combined discovery output |
| perf | Not found on PATH | Direct discovery produced no output |

All mandatory T001 inventory categories were provided and reviewed. Missing packages do not invalidate the inventory; toolchain setup belongs to later tasks. No C++20 compilation, application configure/build/test, debugger/sanitizer execution, package compatibility, ML or performance result is established here. Profiler availability is distinct from usable profiling support; neither is established for perf by this output.

Reviewed repository HEAD: `bffdd74063faba11c261432c3a890c8da0c2fb9c`, read by the assistant for review identity. The user's status snapshot above precedes these evidence/README updates. The earlier missing-workspace observations do not justify altering Git: both this transcript and current read-only repository queries identify the valid DefectRay root.


### T002 layout and Git hygiene (2026-11-05)

The user delegated T002 to the assistant. Added the planned directory map to README.md and root-directory rules to .gitignore. The assistant ran these checks in `/home/weam/projects/DefectRay`; all paths are illustrative and no artifact directories or fixture files were created. V-HYGIENE passed, including generated-job and small-fixture coverage.

```bash
git check-ignore -v -- build/debug/CMakeCache.txt .venv/pyvenv.cfg data/example.png models/example.onnx artifacts/jobs/example/state.sqlite artifacts/evidence/example/run.json
```

Exit status: 0. Observed output:

```text
.gitignore:2:/build/	build/debug/CMakeCache.txt
.gitignore:3:/.venv/	.venv/pyvenv.cfg
.gitignore:6:/data/	data/example.png
.gitignore:7:/models/	models/example.onnx
.gitignore:8:/artifacts/	artifacts/jobs/example/state.sqlite
.gitignore:8:/artifacts/	artifacts/evidence/example/run.json
```

```bash
git check-ignore -- src/main.cpp tests/unit/test_smoke.cpp tests/fixtures/tiny.png tests/fixtures/expected.json python/defectray_ml/train.py docs/benchmarks.md README.md .gitignore
```

Exit status: 1. No output; this is the expected non-match for paths that must remain trackable.

```bash
git status --short
```

Exit status: 0. Observed output:

```text
 M AGENTS.md
?? .gitignore
?? README.md
?? docs/
```

Reviewed HEAD: `bffdd74063faba11c261432c3a890c8da0c2fb9c`. Reviewed working-file identity from `sha256sum .gitignore README.md` (exit 0):

```text
02ff0c495801dba920f626d9ea4e8727fd291a240b3410ce28543ebdd4fe3219  .gitignore
dad10534a7033c40b11608d83bc6c9c5deb6a70debacf66488dae8784ecd3826  README.md
```

`git diff --check` returned 0 with no output. The new/untracked files were also checked against their pre-task copies (or /dev/null for .gitignore) for added whitespace errors. Existing AGENTS.md, spec.md and benchmarks.md contents were preserved. No application configure/build/test or dependency compatibility result is claimed; task acceptance remains in plan.md.
