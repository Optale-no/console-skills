---
name: optale-console
description: Use when an agent is connected to an Optale Console through the hosted Optale MCP server, or the user mentions Optale, the Console, Objects, the Optale Wiki or saved Functions and the task is not yet clear. Explains the connection, discovery of the tenant's types, how to read responses and errors, and which Optale skill to load next. Not for a task whose kind is already known; use optale-find-and-read, optale-objects, optale-functions, optale-wiki or optale-files.
license: MIT
---

# Optale Console

The Optale Console is a company's shared workspace. It holds:

- **Objects**: typed records such as a company, a person or a contract. Each type has its own Properties.
- **Relationships**: typed, directed links between two Objects.
- **The Wiki**: Markdown pages in folders, with Git history.
- **Files**: stored bytes that can be placed in Wiki folders or attached to Objects.
- **Functions**: saved queries over Objects and saved actions that chain tool calls.

You reach all of it through the hosted Optale MCP server. Every call runs as the person who created the connection key and sees only what that person can see.

## Connection

A person creates a named connection in the Console under Settings > External agents. There they tick the tools the agent may use and copy the MCP URL and the key. The key is shown once.

Tool names in these skills are the bare MCP names, such as `object_query`. Your client adds its own prefix: Claude Code with a server named `optale` calls it `mcp__optale__object_query`. The server's `tools/list` is the authority on what this connection may call. A call to a tool the connection was not given is refused with `OPTALE_MCP_TOOL_NOT_FOUND`, and `details.availableTools` lists what it has.

## Discover before you assume

Every Console has its own ontology. Never assume a type, a Property key or a Relationship exists because another Console had it.

1. `object_definition_list` without arguments: every Object type with its slug, label, description and Property keys.
2. `object_definition_list` with `objectSlug`: one type in full, including Property types, options, which Properties are required and which are computed.
3. `relationship_definition_list`: Relationship types, the Object types each one joins, direction and cardinality.

Your client may already show the tenant's type slugs and writable Properties inside the tool schemas. The definitions are still where descriptions, options and meaning live. Read them before you write.

Most tools take `scope`: `company` (the shared space, the default) or `user` (the caller's private space). Stay in `company` unless the user asks for their own space.

## Reading responses

Results are JSON. Long lists come in pages, and a page that is not the full set starts with a line such as `PARTIAL PAGE: 3 of 20 matching rows are on this page (15%)`, often followed by a `DIGEST:` line that summarises the page. In the JSON, `partial: true`, `hasMore: true` and a `nextCursor` mean more rows exist.

Before you state a count, a ranking, "all" or "none", keep calling with `cursor` set to the last `nextCursor` while one is returned, or use a count. A result that stays partial without a cursor cannot be paged: follow its `hint` (open the items it names, or narrow the call), never repeat the same call, and never claim it is complete. See `references/concepts.md` for coverage blocks, IDs and links.

## Errors

A failed call returns `isError: true` with `{"error": {"code", "message", "status", "details"}}`. Codes start with `OPTALE_` (file tools also use short codes such as `FILE_IN_USE`). The message and `details` name what to change. Fix that and call again once.

Calling the same tool with the same arguments after it failed is refused with `OPTALE_MCP_REPEATED_FAILED_CALL` and the earlier failure; change the arguments instead. A timeout on a write may still have written: read the target back before you retry. See `references/errors.md`.

## Which skill next

| The user wants to | Load |
| --- | --- |
| Find, look up, list, count or explain what is in the Console | `optale-find-and-read` |
| Create, update, link, archive or delete Objects and Relationships | `optale-objects` |
| Run or repair a saved query or saved action | `optale-functions` |
| Read, write, patch, move or restore Wiki pages, or link a page to an Object | `optale-wiki` |
| Import, store, place, attach, view or export files | `optale-files` |

`tool_batch` runs several tools in one call, in dependency order when items name `dependsOn`. Each item is checked and governed as if called on its own.
