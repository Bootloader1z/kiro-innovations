# Subagent Delegation

## When to Delegate

Use subagents to parallelize work when implementing features or making changes that span multiple independent concerns. Delegate when:

- The task involves 2+ independent files or components that don't depend on each other's output
- Research/context-gathering is needed across different parts of the codebase
- Multiple distinct implementation units can proceed simultaneously (e.g., frontend + backend, model + migration + controller)

Do NOT delegate when:

- The task is a single-file change or simple fix
- Steps are sequential and each depends on the previous result
- The overhead of coordination exceeds the benefit (trivial tasks)

## How to Delegate

1. Break the work into independent parallel units based on natural boundaries (component, layer, domain)
2. Spawn subagents for each independent unit — target 2-4 subagents depending on task scope
3. Provide each subagent with clear, self-contained instructions including relevant file paths and expected output
4. After subagents complete, review and integrate their results, resolving any conflicts

## Delegation Boundaries

| Task Scope | Subagents | Example |
|---|---|---|
| Small (1-2 files, single concern) | 0 — do it directly | Bug fix, config tweak |
| Medium (3-5 files, 2 concerns) | 2 | API endpoint + Vue page |
| Large (6+ files, 3+ concerns) | 3-4 | Full feature: model, service, controller, frontend |
| Research/investigation | 1 (context-gatherer) | Understanding unfamiliar code before changes |

## Subagent Types

- **context-gatherer**: Use first when entering unfamiliar code areas. One instance, not parallelized.
- **general-task-execution**: Use for implementation work. Parallelize across independent units.

## Integration After Delegation

After all subagents return, always:

1. Verify no conflicting changes (imports, naming, shared state)
2. Run the build/lint to catch integration issues
3. Fill any gaps the subagents couldn't handle due to cross-cutting concerns
