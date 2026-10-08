---
name: project-orchestration
description: "Orchestrate projects with Hermes kanban and delegate_task."
version: 1.0.0
author: Hermes Agent
platforms: [linux, macos, windows]
---

# Project Orchestration

Guide for advancing projects using Hermes kanban board and delegate_task for multi-agent coordination.

## Typical Workflow

1. **Inspect Kanban Board**
   - Use `kanban_list` to see tasks, status, assignee, dependencies.
   - Identify ready, blocked, done tasks.

2. **Dispatch Pending Tasks**
   - If ready tasks exist, spawn a background agent via `delegate_task` to run `hermes kanban dispatch`.
   - Use `delegate_task(background=true)` to avoid blocking your turn.
   - Wait for async delegation result (returns as new message).

3. **Monitor Progress**
   - Periodically re-run `kanban_list` to see status changes (running, blocked, done).
   - For review/test tasks, ensure reviewer/tester agents are spawned appropriately.

4. **Handle Review and Testing**
   - After implementation task completes, move to review via `kanban_request_review` or wait for reviewer agent.
   - If reviewer requests changes, use `kanban_request_changes` with specific, actionable reason.
   - After review passes, move to testing similarly.
   - Only after both reviewer and tester approve, consider task done (or trigger deploy via devops).

5. **Create Documentation Handoffs**
   - For each completed task, delegate a documenter agent to create a handoff markdown under `docs/handoff/`.
   - Include summary, changed files, next steps.
   - Use `kanban_complete` with metadata and summary to close task.

6. **Iterate Until Completion**
   - Repeat until all project tasks are done.
   - Break larger features into smaller kanban tasks with clear parent/child dependencies.

## Pitfalls

- **Do not trust subagent self-reports alone** — verify outcomes by checking artifacts, file changes, or re-running validation (e.g., run migrations, tests).
- **Avoid polling** — use async delegation and wait for the result to re-enter as a new message; do not loop checking transcripts.
- **Always use clarify for user decisions** — when needing clarification, approval, or choice, use clarify tool with at most one question per call unless independent.
- **Preserve role alternation** — never send two assistant messages in a row; only tool results can repeat.
- **Do not hand-edit config.yaml** — use `hermes config set KEY VAL` to avoid corrupting live gateway.

## Example: Stockify Database Migration Orchestration

- Created kanban tasks: t_7b91a2b2 (buat database migrasi dan models), t_ea483857 (review), t_38a5b2a9 (catat dokumentasi handoff).
- Dispatched via delegate_task; reviewer started review.
- Reviewer approved; task moved to done.
- Documenter created handoff at `docs/handoff/database_migration_models_summary.md`.
- All tasks done, ready for next phase (e.g., Product CRUD API).

## References

- See `hermes-agent` skill for general Hermes usage, configuration, and tool details.
- See `sdlc-review` skill for reviewing kanban handoffs and routing verified outcomes.
