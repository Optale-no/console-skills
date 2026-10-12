---
name: optale-functions
description: Use when the user asks to run, list, inspect or repair a saved query or saved action in an Optale Console, or when a standing question ("open items by owner", "this month's totals") probably has a saved Function already. Covers function_list, function_read, function_execute, paging rows to completeness, action step results, revisions, and recovery from OPTALE_FUNCTION_DEPENDENCY_CHANGED. Not for ad-hoc filters on one Object type; use optale-find-and-read. Not for writing Objects directly; use optale-objects.
license: MIT
---

# Saved Functions

A Function is saved in the Console and has a stable `id`, a `kind` and an integer `revision`:

- `kind: "query"`: a typed, read-only query over Objects with declared parameters and a fixed result schema.
- `kind: "action"`: a saved chain of tool calls run in order.

The people who run the Console save Functions for the questions they ask often. Prefer an existing query Function over rebuilding the same filter with `object_query`: it encodes their definitions of words such as "open" or "active".

## Find one

1. `function_list` returns each Function's `id`, `kind`, `name`, `description`, current `revision`, `parameters` and, for queries, `resultSchema`. It pages: follow `nextCursor` until `partial` is false before you say a Function does not exist.
2. `function_read` with `id` (or a query's `name`) returns the full definition, its parameters and the dependencies it is pinned to. Omit `revision` to read the current head.

## Run a query

Call `function_execute` with `id`, the current `revision` from `function_list` or `function_read`, and `arguments` matching the declared parameters. Leave `arguments` out when there are none.

The result is a `page`:

- `rows`: each row's `values` follow `page.schema`. Grouped counts return one row per group with the count in `value`.
- `page.coverage`: `enumeration: "complete"` means every row is here.
- `page.nextCursor`: when it is not `null`, call `function_execute` again with the same `id`, `revision` and `arguments` plus `cursor`. Stop when it is `null`.

Always execute the head revision unless the user asks for an older one. An older revision stays pinned to the ontology as it was when it was saved.

If the connection lists `query_execute`, it runs a saved query by name in one step: `definition: {name}` plus `arguments`, then `mode: "continue"` with `cursor` for the next page.

## Run an action

Read the action with `function_read` first, so you know which tools its steps call and which arguments it needs. Then call `function_execute` with its `id`, `revision` and `arguments`.

- Each step is a real tool call, checked as if you had called it yourself.
- The run stops at the first failed step and never rolls back. Steps that ran stay done.
- The result reports `status`, every step with `succeeded`, `failed` or `not_run`, and a `summary`. Report which steps ran and which did not.

## When a Function refuses

| Code | What happened | Do this |
| --- | --- | --- |
| `OPTALE_FUNCTION_DEPENDENCY_CHANGED` | A Property, type or tool the Function was saved against changed. Nothing ran. | Repair below. |
| `OPTALE_FUNCTION_DEPENDENCY_MISSING` | An action step uses a tool this connection cannot run. Nothing ran. | Tell the user which tool the connection needs. |
| `OPTALE_FUNCTION_NOT_FOUND` | Wrong `id`, `name` or `scope`. | List again in the right `scope`. |
| `OPTALE_FUNCTION_REVISION_REQUIRED` | `revision` was missing or not a positive integer. | Pass the head revision. |

## Repair a changed dependency

Saving a Function binds it to the current definitions. When only the surroundings changed and the definition itself is still right:

1. `function_read` the Function without `revision` to get the head definition and revision.
2. `function_save` with the same `id`, `kind`, `name`, `description` and `definition`, and `expectedRevision` set to that head revision.
3. `function_execute` the new revision the save returns.

If the save is refused because the definition no longer fits (a Property was removed or renamed), the definition itself has to change. Fix the paths the error names against the current `object_definition_list` and save again.

`expectedRevision` protects against overwriting someone else's save: if it is not the current revision the save is refused. Read again and repeat. `expectedRevision: 0` creates a new Function.
