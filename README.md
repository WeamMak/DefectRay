# DefectRay

A learning project for a C++20/Linux distributed visual inspection engine: one coordinator, two TCP workers, CPU model inference, recoverable progress, and HTML/JSON reports.

Application implementation and verified build/run commands are not available yet. The project follows small human-implemented tasks with review and explicit acceptance.

## Development environment

Use Ubuntu inside WSL2 or native Linux. Keep the repository, compiler, future Python environment and build outputs inside Linux. Run project commands from the Linux repository root, observed at `/home/weam/projects/DefectRay`.

The user's 2026-11-05 inventory confirmed Ubuntu 26.04 LTS on WSL2, GCC 15.2.0 at `/usr/bin/g++`, and system Python 3.14.4 at `/usr/bin/python3`. Git recognizes the existing repository; reinitialization is unnecessary.

CMake, CTest, GDB, clang-format, clang-tidy and perf were not found on that terminal's PATH. These findings are recorded for later setup tasks; this inventory does not establish a working C++20 build, debugger, sanitizer or profiler.

The project Python version remains unselected pending ML dependency compatibility checks. Do not automatically use system Python 3.14.4 or install packages solely from shell suggestions. There are no verified application configure/build/test commands to run yet.

Full commands and observed results are retained in the [test plan](docs/test-plan.md#t001-environment-inventory-2026-11-05). Current task readiness and acceptance are recorded in the [implementation plan](docs/plan.md).

## Planned project layout

Directories are created when their first task needs them.

| Path | Purpose | Git policy |
|---|---|---|
| `src/` | C++ coordinator, worker and shared components | Track source |
| `tests/` | Native tests and small reproducible fixtures in `tests/fixtures/` | Track source and small fixtures |
| `python/` | Model development, export, reports and experiment tooling | Track source |
| `docs/` | Specification, task progress, verification, benchmark methods and small result summaries | Track documentation |
| `build/` | Generated CMake build trees | Ignore |
| `.venv/` | Project-local Python environment | Ignore |
| `data/` | Acquired datasets and generated bulk inputs | Ignore |
| `models/` | Local model weights and bundles | Ignore |
| `artifacts/` | Generated jobs in `artifacts/jobs/`, full evidence in `artifacts/evidence/` and experiment outputs | Ignore |

Keep acquisition and reproduction instructions, plus small result summaries, in
`docs/`. Put generated jobs and full evidence under `artifacts/`. Keep small test
fixtures under `tests/fixtures/` so ignoring datasets does not hide test inputs.
The ignore rules apply to root directories; adding a rule does not untrack files
already in Git.

## Project documents

- [Specification](docs/spec.md): scope, architecture and requirements.
- [Implementation plan](docs/plan.md): task sequence, budget, reviews and acceptance.
- [Test plan](docs/test-plan.md): checks, actual commands and verification evidence.
- [Benchmarks](docs/benchmarks.md): measurement methods and evidence.
- [Working agreement](AGENTS.md): human implementation and review workflow.



