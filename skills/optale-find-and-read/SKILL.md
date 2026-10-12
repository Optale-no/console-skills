---
name: optale-find-and-read
description: Use when the user asks to find, look up, list, filter, count or explain anything stored in an Optale Console, such as "which customers", "show me the contract for", "how many open" or "what is linked to". Picks the right read tool, pages to completeness and opens Objects, Relationships and search hits. Not for changing data; use optale-objects, optale-wiki or optale-files. Not for running a saved query; use optale-functions.
license: MIT
---

# Find and read in the Optale Console

Every tool here only reads. Read the `optale-console` skill first if you have not discovered this Console's types yet.

## Pick the tool

| You know | Use |
| --- | --- |
| The Object type, and conditions on its Properties ("open deals over 100k", "people at this company") | `object_query` |
| The Object type, and a name or alias to look up | `object_search` |
| Only a name, not its type | `object_graph_retrieve` with `query`, or `source_search` |
| One Object's id or human key, and you need all its Properties | `object_read` |
| One Object, and you need what it is linked to | `relationship_list`, or `object_graph_retrieve` with `objectId` for neighbours with their values |
| A word or phrase that could be anywhere: Objects, Wiki pages, files, chats, activity | `source_search`, then `source_read` on a hit |
| A question a saved query already answers | `function_list`, then the `optale-functions` skill |

Wiki pages have their own reads; see `optale-wiki`.

## Query a type

1. Check the type's Property keys and option values with `object_definition_list` and `objectSlug`. Filter on the stored option value, not on a label you guessed.
2. Call `object_query` with `objectSlug`, `filters`, `sorts` and `fields`.
   - `filters` is one AND/OR group of conditions `{field, op, value}`. Use `in` and `not_in` for option lists, `contains` for a phrase inside text, `before`/`after`/`on` for dates, `is_empty`/`is_not_empty` without a value.
   - A relationship condition filters by linked Objects: `{path: [{relationship, direction}], op: "any" | "none", type, where}`. `direction: "outgoing"` means the queried Object is the source.
   - Name the Properties you need in `fields`. Rows always carry `id`, `objectSlug` and `displayName`; leave `fields` out when that is enough. Use `["*"]` only when you must see every Property.
   - Without `sorts` the order is internal. For "latest" or "oldest", sort by `createdAt`, `updatedAt` or a date Property.
3. Every page reports `total`, the number of matches across all pages. For "how many", a call with a small `limit` is enough; read `total`.

## Search by name

`object_search` matches the Object's key, display name and aliases as contiguous text, so a short word also hits inside longer words. It does not filter by business status: an Object that is not archived may still be finished work. For status questions use `object_query`.

A name that matches nothing returns `nearMatches` with close spellings. Try those before you tell the user nothing exists. An empty `object_query` result describes what was searched, so you can say exactly what was not found.

## Read one Object

`object_read` takes `objectId` or `key`, not both. Every declared Property is present; one with no value reads `null`. `sameName` lists archived Objects with the same display name, which helps when the user means an older one.

## Follow links

- `relationship_list` pages one Object's Relationships. Use `lifecycle: "ended"` or `"all"` for history.
- `object_graph_retrieve` returns a neighbourhood: `depth: 1` gives each neighbour's values, `depth: 2` adds a further ring of identity rows. A dense Object returns counts per Relationship type in `meta.neighbourhood`; page with `cursor` or narrow with `relationshipDefinitionSlugs`.
- `object_query` and `object_search` take `contextDepth: 1` to attach related Objects to each row.

## Search everything

`source_search` takes a `query` and optional `kinds` (`object`, `wiki`, `wiki-property`, `file`, `chat`, `activity` and others) and `mode` (`lexical`, `semantic` or `combined`). It separates exact matches from ranked evidence. Kinds still indexing are listed in `coverage.pending`; keep paging to include them.

Open a hit with `source_read`, passing its `source` object exactly as returned. It opens the revision that was found and refuses if a current-only source changed since, so search again rather than guess.

## Be complete before you conclude

- A page with `partial: true` is not the whole set. Before you claim "all", "none", a ranking or a total from rows, call again with `cursor` set to the last `nextCursor` while one is returned, keeping every other argument the same.
- Some results stay partial with no cursor to continue, such as a `depth: 2` `object_graph_retrieve` that shows only a few edges per neighbour. Follow the result's `hint`: open the neighbours it names, or narrow the search. Do not repeat the same call, and do not claim the set is complete.
- If the client cuts a long result short, ask for fewer `fields`, a smaller `limit`, or read `total` instead of listing.
- A timeout (`OPTALE_MCP_TOOL_TIME_BUDGET_EXCEEDED`) means the call was too wide. Narrow the filter or the page size.
- Cite what you found with the `link` each row carries.
