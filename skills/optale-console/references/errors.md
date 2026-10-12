# Optale MCP errors

A tool failure comes back as a tool result with `isError: true` and one JSON object:

```json
{"error": {"code": "OPTALE_...", "message": "...", "status": 400, "details": {}}}
```

Read `message` and `details` first: they name the field, the allowed values or the next call. Fix what they name and call once more. Never repeat an identical failing call.

## Connection and transport

These arrive as HTTP responses before any tool runs.

| Code | Status | What to do |
| --- | --- | --- |
| `OPTALE_CONNECTION_KEY_INVALID` | 401 | The key is wrong or was revoked. The user creates a new connection under Settings > External agents. |
| `OPTALE_MCP_OVERLOADED` | 429 | The server is busy. Wait the `Retry-After` seconds (also in `retryAfterSeconds`), then retry. |

## Any tool

| Code | Status | Meaning and recovery |
| --- | --- | --- |
| `OPTALE_MCP_TOOL_NOT_FOUND` | 404 | The tool does not exist or this connection was not given it. `details.availableTools` lists what it has. |
| `OPTALE_MCP_ARGUMENT_INVALID` | 400 | An argument does not fit the tool's schema. The message names the field. Tool families also use their own `..._ARGUMENT_INVALID` codes. |
| `OPTALE_MCP_REPEATED_FAILED_CALL` | 409 | You repeated a call that already failed with the same arguments. `details.earlierFailure` holds the first error. Change the arguments. |
| `OPTALE_MCP_TOOL_TIME_BUDGET_EXCEEDED` | 504 | The call ran out of time. Narrow it: a smaller `limit`, fewer fields, a tighter filter. If `details.mayHaveCommitted` is true the tool writes and the write may have landed: read the target back before you retry. |
| `OPTALE_MCP_TOOL_CANCELLED` | 499 | The call was cancelled. Same rule on `mayHaveCommitted`. |

Timeouts, cancellations, rate limits and server errors (status 408, 429, 499 and 5xx) are never treated as repeated failures; you may retry them.

## Objects and Relationships

| Code | Meaning and recovery |
| --- | --- |
| `OPTALE_OBJECT_NOT_FOUND` | No Object with that id or key is visible to you in that scope. Check the id came from a search or query in the same `scope`. |
| `OPTALE_SEARCH_CURSOR_INVALID` | The search cursor no longer applies. Repeat the search without `cursor`. |
| `OPTALE_OBJECT_ENSURE_EXTERNAL_IDENTITY_INVALID` | `object_ensure` needs a complete `source` and `externalId`. |
| `OPTALE_OBJECT_ENSURE_INCOMPLETE` | Some steps of an ensure did not finish. The result lists each step and its status. Repeating the identical call with the same `operationKey` resumes it; a corrected request needs a new `operationKey`. |
| `OPTALE_OBJECT_ENSURE_OPERATION_CONFLICT` | This `operationKey` was already used with a different request. The result has `status: "blocked-by-conflict"` and `conflictObjectId`, the Object that key wrote. Use a new `operationKey` for a new request. |

`relationship_create` without a valid `relationshipDefinitionSlug` is refused with the definitions the two Objects admit. Pick one from the list.

## Functions

| Code | Meaning and recovery |
| --- | --- |
| `OPTALE_FUNCTION_NOT_FOUND` | No Function with that id or name in that scope. List them with `function_list`. |
| `OPTALE_FUNCTION_DEPENDENCY_CHANGED` | A tool or definition the Function was saved against changed. Nothing ran. See the `optale-functions` skill for the repair. |
| `OPTALE_FUNCTION_DEPENDENCY_MISSING` | A step uses a tool this caller cannot run. Nothing ran. |
| `OPTALE_FUNCTION_DEFINITION_INVALID` | A saved definition is not valid; the message names the path. |

## Files

| Code | Meaning and recovery |
| --- | --- |
| `FILE_IN_USE` | The file is still placed somewhere. `details.placements` names every place; remove those placements first. |
| `FILE_CONTENT_INVALID` | `contentBase64` is empty, too large or not canonical base64. |
