# Optale Console concepts

Background for every Optale skill. Load the part you need.

## Objects and definitions

- An **Object definition** (an Object type) has a slug such as `company`, labels, a description, a label Property (usually `displayName`) and Properties. Read it with `object_definition_list`.
- A **Property** has a key, a type (text, number, date, select, actor reference, file, Wiki page and others) and sometimes options. A select Property accepts only its listed option values. An actor Property lists the people it accepts in `choices`; write their ids, not their names.
- A **computed** Property is derived by the server, for example a sum over related Objects. You can read and filter it; you cannot write it.
- Every Object has an `id`, an `objectSlug` and a `displayName`. Some types also give each Object a human key; `object_read` accepts it as `key`.
- **Lifecycle**: an archived Object is hidden from normal lists and can be unarchived. A deleted Object is gone; its Relationships end.

## Relationships

- A **Relationship definition** has a slug, a source Object type, a target Object type, an inverse slug and a cardinality. Read them with `relationship_definition_list`.
- A Relationship always runs from a source Object to a target Object. In filters, `outgoing` means the Object you are querying is the source.
- A Relationship can carry metadata, but only the scalar Properties its definition declares. Point at another Object with a Relationship, never with metadata.
- Ending a Relationship keeps it in history (`relationship_list` with `lifecycle: "ended"`); it can be reactivated. Deleting removes it.

## Scopes

`company` is the shared space everyone in the Console works in, and the default. `user` is the caller's own private space, when it is enabled. Objects, Wiki pages and Functions live in one scope; pass the same `scope` on every call about them.

## Revisions

Writes are guarded by revisions so two writers do not silently overwrite each other.

- **Objects** record every change as a history event. `object_update` answers with `replacedValues`, each naming the old value and the event to restore it from.
- **Wiki pages** have a `revision` (a content checksum) and Git commits. Pass the revision you read as `expectedRevision` when you write; a stale one is refused.
- **Files** have numbered revisions. `expectedRevision` works the same way.
- **Functions** have integer revisions. You execute an exact revision; a save names the revision it replaces.

When a write is refused because the target changed, read it again, apply your change to what is there now, and write once more.

## Pages, cursors and coverage

- Lists return `requested`, `returned`, `hasMore`, `partial`, `nextCursor` and sometimes `total` and `coveragePercent`. Continue with `cursor`. Keep every other argument the same while you page.
- A text header line before the JSON says the same thing in words: `PARTIAL PAGE: ...`. A `DIGEST:` line lists which fields the rows carry and which stored Properties you did not ask for.
- Saved query results carry `page.coverage`. `enumeration: "complete"` and `nextCursor: null` mean you have every row.
- Ranked searches scan a bounded number of candidates (`hardCap`, `capped`). If a search is capped, narrow it.
- Results can include `disambiguation` groups when a name matches several kinds or types, each with a `drillDown` to repeat the search narrower.

## IDs and links

Rows carry a `link` in Markdown form such as `[Name](object:<id>)` or `[Title](doc:company:<path>)`. Quote these links to the user; they open in the Console. Use the raw `id` in tool arguments, never the link text.

## Files and placements

A file is stored once. It can be **placed** in one or more Wiki folders and **attached** to Object file Properties without copying its bytes. Deleting a file that something still places is refused. My Files holds the files a person keeps.

## Functions

A **query Function** is a saved, typed query over Objects with declared parameters and a fixed result schema. An **action Function** is a saved chain of tool calls. Both pin the definitions they depend on. When the ontology changes under a Function, executing it can be refused until a new revision is saved.
