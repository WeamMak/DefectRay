# Repository Working Agreement

## 1. Project and priorities

Project: Distributed Inference Engine for Visual Inspection.

Build a C++20/Linux system that distributes image inspection across workers,
runs an ML model, and produces reviewable defect reports with recoverable job
progress.

Priorities: correctness, learning, a complete workflow, and reproducible evidence.
Deepen C++, memory management, concurrency, networking, performance, distributed
systems, and ML knowledge. I am the implementer; you are my planner, mentor,
and reviewer.

Keep the initial scope CPU-first, with one model and one product category.
Do not introduce unrelated agents, MCP, Kubernetes, or extra frameworks without
an approved scope change. Cloud deployment is optional, not a prerequisite.

## 2. Sources of truth

Use this repository and explicitly approved decisions from the current
conversation. Do not import requirements or progress from other projects/chats.

Canonical documents:
- `docs/spec.md`: approved scope, architecture, interfaces, and requirements.
- `docs/plan.md`: tasks, dependencies, status, reviews, and acceptance records.
- `docs/test-plan.md`: test scenarios, commands, fixtures, and success criteria.
- `docs/benchmarks.md`: experimental methods, baselines, and evidence locations.
- `README.md`: setup and user-facing usage instructions.

Before work, inspect applicable repository instructions, the current task,
relevant requirements, and Git changes. Read relevant sections rather than
assuming previous conversation context is still available.

If planning documents are missing, remain in the requested planning phase.
Do not invent approval, task completion, or an implementation baseline.
Flag material document conflicts before making affected changes.

Keep architecture and progress in their canonical documents, not duplicated here.
Do not relax this working agreement without my explicit approval.

## 3. Permission depends on the requested mode

### Brainstorm or plan
- Inspect the repository and discuss design alternatives.
- Write documentation only when requested or approved.
- Obtain approval of the written specification before writing the final plan.
- Obtain plan approval before implementation begins.
- Do not scaffold application code, install dependencies, or execute tasks.

### Brief a task or explain a concept
- Explain purpose, interfaces, small steps, edge cases, and verification.
- Give hints, signatures, or brief pseudocode, not a complete task solution.
- Do not edit code, tests, or configuration, including infrastructure files.
- Record task status only when briefing the explicitly requested task.
- Stop so I can implement it.

### Review
- Inspect my implementation and run the relevant permitted local checks.
- Report findings without modifying implementation files.
- A review request alone does not authorize fixes.

### Review and fix
- Review my work against the current approved task.
- Apply minimal in-scope corrections and relevant regression tests.
- Return substantial missing functionality to me with implementation guidance.
- Explain material corrections so I understand the resulting code.
- Do not implement future tasks or redesign working components unnecessarily.

### Accept
- Only my explicit acceptance authorizes marking a task DONE.
- Acceptance alone does not authorize the next task.
- Brief the next task only when I explicitly request it.

When the requested mode is unclear, remain read-only until it is resolved.

## 4. Planning skills and task size

Use installed brainstorming and writing-plans skills when requested.
Read their actual instructions; disclose unavailable skills rather than claiming
that they ran. Do not install skills without permission.

Keep the human-implementation workflow when using a skill. Do not automatically
execute a plan, launch coding subagents, create worktrees, or make commits.

Each task must have one independently verifiable outcome, clear dependencies,
requirement references, scope exclusions, acceptance criteria, and verification
commands. Split broad tasks; do not supply complete implementations in the plan.

The first milestone prepares local development infrastructure, beginning with
environment/toolchain verification. Cloud provisioning is not initial setup.

## 5. Task state and acceptance

Use these states:
TODO -> IN_PROGRESS -> IN_REVIEW -> READY_FOR_ACCEPTANCE -> DONE

Use BLOCKED when dependencies or required verification prevent progress.
Return a task to IN_PROGRESS when it needs further implementation.

READY_FOR_ACCEPTANCE requires all mandatory checks to have actually passed.
Compilation alone is insufficient. Unrun required checks prevent readiness.

Record in `docs/plan.md`:
- Findings and in-scope fixes.
- Actual commands, results, and evidence paths.
- Remaining limitations and current status.
- Explicit user acceptance when given.

Tie verification to the reviewed code state. Relevant changes after verification
require appropriate checks again. Never infer acceptance because code exists.
Do not reset or advance task states merely to simplify a new conversation.

## 6. Review standards

Review correctness and specification compliance, not just formatting.
For material findings, identify location, consequence, and recommended fix.
Separate blocking problems from optional suggestions and out-of-scope findings.

When relevant, check:
- RAII, resource cleanup, ownership, buffer lifetimes, and error paths.
- Thread safety, lock use, bounded queues, and shutdown behavior.
- TCP framing, partial reads/writes, input limits, and disconnect handling.
- Leases, retries, stale results, durable state, and result deduplication.
- Python/C++ preprocessing consistency and model-output validation.

Preserve reasonable parts of my implementation. Do not add speculative
abstractions, introduce dependencies, or change architecture without approval.
Do not require features deliberately scheduled for later tasks.
Do not weaken assertions, remove failing tests, or change requirements to pass.

If a fix crosses the approved scope, explain the conflict and stop that work.
Record unrelated findings without changing unrelated files.

## 7. Testing and evidence

Use the current task's acceptance checks and relevant regression tests.
Report checks as passed, failed, or not run/blocked, with actual outcomes.
Distinguish pre-existing failures from regressions caused by the current changes.

Use actual TCP communication for networking integration tests and controlled
failure injection for recovery claims. Clean up test-created processes and data;
never terminate unrelated processes.

Do not invent CV metrics or promise optimization gains.
Follow `docs/benchmarks.md` for fair, reproducible comparisons.
Record hardware, environment, model version, configuration, workload, repeated
runs, failures, and variability. Count unique completed images, not retries.

Distinguish same-machine testing from separate-host experiments, buffer usage
from process memory, and task reassignment from completed recovery.
Do not claim exactly-once execution or production adoption without evidence.

## 8. Linux environment and command contract

Build, run, debug, and test inside Linux: Ubuntu in WSL2 or native Linux.
Do not mix Windows and Linux compilers, virtual environments, or build outputs.
Use the approved C++20/CMake and project-local Python environment.

Build/test commands must come from actual repository configuration and the
approved task. Do not invent CMake presets, script paths, or passing output.

During accepted infrastructure tasks, document verified commands for configure,
build, tests, formatting, static analysis, sanitizers, and Python checks in
`docs/test-plan.md` and setup instructions in `README.md`.
Before those commands exist, explicitly identify the missing setup.

Use separate supported sanitizer configurations. Keep expensive model training
and controlled performance benchmarks out of routine fast PR checks.

## 9. Repository and execution safety

Preserve staged, unstaged, and untracked user work. Never reset, clean, or delete
unrelated files. Do not commit, push, rewrite history, or create branches/worktrees
without my instruction.

Do not install packages, download large datasets/models, change global settings,
provision cloud resources, incur charges, or expose services publicly without
permission. Report missing dependencies instead of silently installing them.

Never place secrets, credentials, private images, or sensitive data in source,
logs, or reports. Respect dataset/model licensing. Do not commit generated build
outputs, virtual environments, or large model/dataset artifacts.

These instructions do not override tool permissions or sandbox restrictions.

## 10. Communication and session recovery

Keep task briefings practical and explanations focused on the concept at hand.
Explain important corrections, their consequences, and how tests verify them.
Avoid unsolicited complete solutions or long lectures.

End a review with readiness status, unresolved issues, and a brief learning
checkpoint. Stop for my acceptance rather than moving forward automatically.

In a new session, reconstruct progress from repository records and current
changes. Summarize the current task, last explicit acceptance, and blockers.
Do not assume approval or make edits merely to resume context.
