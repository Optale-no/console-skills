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
- **Wiki documents** have immutable database revisions and a content checksum. `revisionId` comes from `wiki_history`; `page.revision` is only for `expectedRevision` when writing. All history follows the document's current Space access.
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

## Chats and groups

`chat_list` pages through the caller's chats with titles, recent activity, group IDs, settled status and short previews. Use it to propose a grouping without making changes. `chat_project_search` finds groups by name and returns their `projectId`.

`chat_manage` applies one operation to an `items` array. `group_save` creates a group with `name` when `projectId` is null or omitted; an existing native id updates it. A blank id is refused: pass null to create. On update, null `name` or `description` preserves the existing value; an empty description clears it. `assign` moves each `conversationId` to a `projectId`; use `"unassigned"` or null to move a chat out of its group without removing the group or the chat.

For chats, use `rename` with `title`, `settle` or `unsettle`, `archive` or `unarchive`, `duplicate`, and `mark_read`. Groups have `group_settle` and `group_unsettle`. `reorder` changes a sidebar section's sort and row order. Pins have been retired; Active and Settled are the current lifecycle.

`share_create` uses `conversationId`. `share_update` and `share_revoke` accept either `conversationId` or `shareId`. Updating rotates the share ID and URL; use the returned URL, not the previous one. `snapshotFiles` has the same meaning as the UI's share-files choice.

`export` uses `conversationId` and `format` (`text`, `markdown`, `json` or `csv`) and delivers a File with its locator and attachment card. `filename`, `includeOptions`, `exportBranches` and `recursive` match the UI export options. Headless export follows the UI-default newest-sibling branch; the browser export uses the person's current sibling selection. PNG remains a UI-only screenshot of the rendered page.

`import` reads an accessible File by `file_id` through the same native JSON importers as the UI upload: LibreChat, ChatGPT, Claude and ChatbotUI. Its result names the source File and revision and confirms completion.

`fork` uses `conversationId` and `targetMessageId`. `forkOption` is `directPath`, `includeBranches` or `targetLevel` (the default), matching the UI's three choices. `splitAtTarget` starts at the selected message level; optional `latestMessageId` supplies the branch tip, otherwise the headless UI-default tip is used.

Up to 50 items share one Activity. Assignment batches are atomic; content operations retain their native UI execution boundaries with journaled receipts. Known input or target refusals are reported per item; unexpected errors stop the unit. Repeated group-create names in one call reuse the first group, and a same-key retry replays committed identities rather than creating again. These writes use the session's Auto or Read Only mode, without another confirmation step.
