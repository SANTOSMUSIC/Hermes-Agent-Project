---
name: hermes-orchestration
description: "Orchestrate workflows with Hermes Kanban and delegation."
version: 1.0.0
author: Hermes Agent
platforms: [linux, macos, windows]
---

# Hermes Orchestration Skill

This skill provides patterns for orchestrating complex workflows using Hermes Agent's
Kanban system and delegation capabilities. It covers task creation, dispatching,
monitoring completion, and handling common obstacles.

## Core Workflow Pattern

When implementing a feature or fixing a bug, follow this orchestration pattern:

1. **Create Kanban tasks** for each unit of work with appropriate assignees
2. **Set up dependencies** between tasks using parent/child relationships
3. **Dispatch the Kanban board** to start processing ready tasks
4. **Monitor task progress** through status changes
5. **Handle blocked tasks** by identifying root causes
6. **Validate completion** through review and testing steps
7. **Document handoffs** for knowledge transfer

## Step-by-Step Procedures

### 1. Creating Task Chains

For sequential work, create tasks with parent relationships:

```bash
# Create initial task
hermes kanban create "Database migrations" --assignee backend-dev

# Create dependent task
hermes kanban create "Review migrations" --assignee reviewer --parents t_xxxxxx

# Create documentation task
hermes kanban create "Document handoff" --assignee documenter --parents t_yyyyyy
```

### 2. Dispatching Work

After creating tasks, dispatch the board to start processing:

```bash
# Through delegation (when direct terminal access unavailable)
delegate_task(goal="Execute hermes kanban dispatch", context="Current Kanban board has pending tasks")

# Or directly if terminal access is available
# (Note: In some environments, direct terminal tool may not be available)
```

### 3. Monitoring Progress

Check task status regularly:

```bash
# List all tasks
hermes kanban list

# Show specific task details
hermes kanban show t_xxxxxx

# Watch for status transitions: todo -> ready -> running -> blocked/done
```

### 4. Handling Blocked Tasks

When a task is blocked, identify the cause and resolve:

- **Dependency waiting**: Wait for parent tasks to complete
- **Needs input**: Provide required information or clarification
- **Capability issues**: Escalate to human if agent lacks required access/tools
- **Transient failures**: Retry after brief pause

Use `kanban_block` with appropriate kind when work cannot proceed:

```bash
hermes kanban block t_xxxxxx --reason "Waiting for API credentials" --kind needs_input
```

### 5. Completing Tasks and Handoffs

When work is complete, properly close tasks:

```bash
hermes kanban complete t_xxxxxx \
  --summary "Completed database migrations for Stockify" \
  --metadata '{"changed_files":["database/migrations/xxxx.php"],"tests_passed":8}' \
  --artifacts "/path/to/generated/file.sql"
```

### 6. Review and Testing Pattern

For code-related tasks, follow this validation sequence:

1. **Implementation** by developer (backend-dev, web-dev, etc.)
2. **Review** by reviewer (check conventions, correctness)
3. **Testing** by tester (run automated tests, verify functionality)
4. **Documentation** by documenter (create handoff notes)
5. **Deployment** by devops (only after review and testing pass)

If review or testing fails, return task to implementer with specific feedback:

```bash
hermes kanban request_changes t_xxxxxx --reason "Missing validation for price field"
```

## Environment-Specific Considerations

### Limited Terminal Access

In environments where the `terminal` tool is not available:

- Use `delegate_task` to spawn subagents that can execute commands
- The subagent can use `computer_use` to interact with terminal windows if needed
- Alternatively, run Hermes CLI commands directly through the subagent's context

Example pattern:

```bash
# Instead of direct terminal command:
delegate_task(
  goal="Run hermes kanban dispatch",
  context="Need to process Kanban board but direct terminal access unavailable"
)
```

### Windows CLI Environment

In Windows CLI/Hermes environments:

- Direct text input to terminal may require `delivery_mode: "foreground"`
- Mouse clicks may work in `background` mode but keyboard input often needs foreground
- Sequence: click to focus -> type with foreground -> press Enter with foreground

## Common Pitfalls and Solutions

**Pitfall**: Assuming direct terminal access is always available
**Solution**: Design orchestration to work through delegation first, fall back to direct only when confirmed available

**Pitfall**: Not setting up proper task dependencies
**Solution**: Always use `--parents` flag when creating tasks that depend on others' completion

**Pitfall**: Missing handoff documentation
**Solution**: Create documentation task as dependent on implementation and review tasks

**Pitfall**: Deploying before review/testing completion
**Solution**: Enforce dependency chain: implement -> review -> test -> deploy

**Pitfall**: Losing track of task status
**Solution**: Regularly run `hermes kanban list` and check specific tasks with `hermes kanban show`

## Stockify Project Example

This pattern was used successfully for the Stockify project:

1. Created migration task -> review task -> documentation task
2. Dispatched board to start work
3. Waited for migration completion
4. Monitored review process and confirmed approval
5. Verified documentation handoff was created
6. Created API implementation task as next phase

This ensured proper sequencing and knowledge transfer between phases.

## References

For detailed tool specifications, see:
- `references/kanban-tool-reference.md`
- `references/delegation-patterns.md`