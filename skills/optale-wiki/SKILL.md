---
name: optale-wiki
description: Use when the user asks to read, write, edit, append to, move, rename, archive, delete, restore or compare pages in the Optale Wiki, look at a page's history, publish a report or HTML document there, or attach a page to an Object. Covers whole-page writes versus patches, expectedRevision, document revisions and restore, and Object links. Not for searching across everything; use optale-find-and-read. Not for binary files or images; use optale-files.
license: MIT
---

# Work in the Optale Wiki

The Wiki is a tree of Markdown documents in folders. Changed saves append database revisions; autosaves may fold together in the history list without removing their saved versions.

Paths are relative to a Space's Wiki root, such as `projects/launch/plan.md`. Resolve a named Space with `space_list` and pass its `spaceId`. A document id follows the document across moves.

## Find and read

- `wiki_tree` browses folders and pages below a path. `wiki_list` lists assets (files) in a folder.
- `wiki_search` searches titles, headings, metadata and body text. It pages like every list: follow `nextCursor` while `partial` is true. Use a result's `disambiguation.groups[].drillDown` to narrow by page type.
- `wiki_read` with `path` returns `found`, and for a page: `page.content` (Markdown without frontmatter), `page.source` (the exact stored bytes), `page.frontmatter` and `page.revision`. A missing page returns `found: false`.

## Change a page

Read before you write, and keep the `revision` you read.

| Change                                | Tool                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Add text at the end                   | `wiki_patch` with `operation: "append"` and `content`                                            |
| Replace one exact passage             | `wiki_patch` with `operation: "replace_text"`, `oldText` (must occur exactly once) and `newText` |
| Replace one section under a heading   | `wiki_patch` with `operation: "replace_section"`, `heading` and `content`                        |
| Create a page, or rewrite most of one | `wiki_write` with `path`, `content` and optional `frontmatter`                                   |

- `wiki_patch` requires `expectedRevision`: the `revision` from `wiki_read`. It changes only the bytes you name and leaves frontmatter alone. Prefer it for small edits to long pages.
- `wiki_write` replaces the whole page. When the page exists, pass `expectedRevision`. Use `page.content` from `wiki_read` as your starting text so you do not write the frontmatter into the body.
- A stale `expectedRevision` is refused because someone else changed the page. Read it again, reapply your change to the new text, and write once more.
- Pass a short `message`; it describes the saved version.

## Move, archive, delete

- `wiki_move` with `path` and the complete `newPath` moves or renames a page or folder. History and links move with it. A page path keeps its `.md`.
- `wiki_archive` hides a leaf page reversibly; `wiki_unarchive` brings it back. Both take the page's `expectedRevision`.
- `wiki_delete` removes a leaf page from the live Wiki, retains a deletion revision and ends its Object links. A revision restore can bring the document back. A page held by an Object's Wiki Property cannot be deleted this way.

## History and restore

1. `wiki_history` with `documentId` lists that document's revisions, including moved or deleted documents. Alternatively, `path` looks up the document currently there; do not pass both. Omit both for the scoped Space. Results are newest first (`limit` up to 50); continue with `offset` from `nextOffset`.
2. `wiki_diff` with `revisionId` compares it with the preceding saved revision. Pass `compareToRevisionId` to choose another version. `truncated: true` means the diff was cut at its byte bound.
3. `wiki_read` with `revisionId` reads that exact version; no path is needed. revisionId comes from wiki_history; page.revision is only for expectedRevision. To edit the current page afterwards, read it without `revisionId` and use that current `page.revision`.
4. `wiki_restore` with `revisionId` appends a new revision containing that version, including bringing back a deleted document. Optional `documentId` or `path` selects its current target. The document stays in its current Space; moving it elsewhere uses `wiki_move`.
5. `wiki_restore` with `asOf` set to an ISO timestamp restores the whole scoped Space's document contents, deletion states and locations at that time. Documents created later become deleted revisions. Path collisions skip that document and are listed; the rest are restored.

All versions, diffs and historical reads follow the document's **current Space access**, not its Space at the time of a version. Restores use the ordinary current-Space write checks; restoring a recorded location in another Space uses the ordinary move checks on both Spaces. Commit hashes and the old `commit`, `hash` and `revision` inputs are refused; use `revisionId`.

For a discovered SourceRef, use `source_read` with its unchanged `source`, or follow its `fetch` recipe. Retained Git pins still open through that native source reader, not through `wiki_read`.

## Pages and Objects

There are two ways a page belongs to an Object:

- **A Wiki-page Property.** Some Object types have a Property of type Wiki page, whose value is `{"kind":"document","id":"<document id>"}`. Write the page with `wiki_write`, then attach its returned document id with `object_update` (see `optale-objects`). The declared Property also accepts `document:<id>` as tool input. Renaming or moving the document does not change the Property value; its current Space controls access. Do not send a path in the Property value. `object_read` lists the exact shape in `writeShapes`.
- **A Wiki link.** `object_wiki_link_create` with `objectId` and `wikiPath` (optional `role`) links any Object to any page. `object_wiki_link_list` lists an Object's linked pages; `wiki_object_backlink_list` lists the Objects linked to a page.

Read an Object's own Properties with `object_read`, not `wiki_read`.

A whole-Space as-of restore can delete a held document without detaching its id: the Wiki Property reads as empty with a deleted-document note until a later revision restore brings it back. Ordinary `wiki_delete` still refuses a held page.

## Documents and assets

- `wiki_html_write` places one durable HTML document (an interactive report, a briefing) in a Wiki folder next to the pages it belongs with.
- For images, PDFs and other files in a Wiki folder, use `file_place` (see `optale-files`).
- Images and files uploaded inside a page live beside its Markdown file in `<page>.md.assets/`. Folder pages use `<folder>/index.md.assets/`. The page tree hides these folders. `wiki_list` can list them by path; `file_get`, `file_read_bytes` and `wiki_read` can still read the files. The page body uses stable native File URLs.
- Moving or renaming a leaf page moves its current assets folder and file placements, including a move to another Space. The page's legacy path links become stable File URLs during the move. Deleting a page removes its asset placements and assets folder permanently. Document history restores Markdown only, not historical asset bytes; those remain in the Git tarball. Files uploaded directly to a Wiki folder stay ordinary file entries.
- The Wiki file viewer keeps metadata under its information button. "Create page from this" creates and opens a page beside the file, with an image or a link to the original file.
