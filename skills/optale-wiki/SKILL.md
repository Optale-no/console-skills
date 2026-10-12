---
name: optale-wiki
description: Use when the user asks to read, write, edit, append to, move, rename, archive, delete, restore or compare pages in the Optale Wiki, look at a page's history, publish a report or HTML document there, or attach a page to an Object. Covers whole-page writes versus patches, expectedRevision, Git history and restore, and Object links. Not for searching across everything; use optale-find-and-read. Not for binary files or images; use optale-files.
license: MIT
---

# Work in the Optale Wiki

The Wiki is a tree of Markdown pages in folders, with Git history. Every write is a commit.

Paths are relative to the Wiki root, such as `projects/launch/plan.md`. `scope` is `company` by default; use `user` only when the user asks for their private Wiki.

## Find and read

- `wiki_tree` browses folders and pages below a path. `wiki_list` lists assets (files) in a folder.
- `wiki_search` searches titles, headings, metadata and body text. It pages like every list: follow `nextCursor` while `partial` is true. Use a result's `disambiguation.groups[].drillDown` to narrow by page type.
- `wiki_read` with `path` returns `found`, and for a page: `page.content` (Markdown without frontmatter), `page.source` (the exact stored bytes), `page.frontmatter` and `page.revision`. A missing page returns `found: false`.

## Change a page

Read before you write, and keep the `revision` you read.

| Change | Tool |
| --- | --- |
| Add text at the end | `wiki_patch` with `operation: "append"` and `content` |
| Replace one exact passage | `wiki_patch` with `operation: "replace_text"`, `oldText` (must occur exactly once) and `newText` |
| Replace one section under a heading | `wiki_patch` with `operation: "replace_section"`, `heading` and `content` |
| Create a page, or rewrite most of one | `wiki_write` with `path`, `content` and optional `frontmatter` |

- `wiki_patch` requires `expectedRevision`: the `revision` from `wiki_read`. It changes only the bytes you name and leaves frontmatter alone. Prefer it for small edits to long pages.
- `wiki_write` replaces the whole page. When the page exists, pass `expectedRevision`. Use `page.content` from `wiki_read` as your starting text so you do not write the frontmatter into the body.
- A stale `expectedRevision` is refused because someone else changed the page. Read it again, reapply your change to the new text, and write once more.
- Pass a short `message`; it becomes the commit message.

## Move, archive, delete

- `wiki_move` with `path` and the complete `newPath` moves or renames a page or folder. History and links move with it. A page path keeps its `.md`.
- `wiki_archive` hides a leaf page reversibly; `wiki_unarchive` brings it back. Both take the page's `expectedRevision`.
- `wiki_delete` removes a leaf page permanently, keeps a tombstone and ends its Object links.

## History and restore

1. `wiki_history` with `path` lists commits newest first (`limit` up to 50; continue with `offset` from `nextOffset`). Without `path` it covers the whole Wiki.
2. `wiki_diff` with a `commit` (and optional `path`) shows what that commit changed. `truncated: true` means the diff was cut at its size bound.
3. `wiki_read` with `revision` set to a commit hash returns the page exactly as it was then.
4. `wiki_restore` with `commit` and `path` restores that page to that commit. Without `path` it restores the whole Wiki scope to that commit, so pass `path` when you mean one page.

## Pages and Objects

There are two ways a page belongs to an Object:

- **A Wiki-page Property.** Some Object types have a Property of type Wiki page, whose value is `{"kind": "wiki-page", "scope", "path"}`. Write the page with `wiki_write`, then set the Property with `object_update` (see `optale-objects`). `object_read` lists the exact shape in `writeShapes`.
- **A Wiki link.** `object_wiki_link_create` with `objectId` and `wikiPath` (optional `role`) links any Object to any page. `object_wiki_link_list` lists an Object's linked pages; `wiki_object_backlink_list` lists the Objects linked to a page.

Read an Object's own Properties with `object_read`, not `wiki_read`.

## Documents and assets

- `wiki_html_write` places one durable HTML document (an interactive report, a briefing) in a Wiki folder next to the pages it belongs with.
- For images, PDFs and other files in a Wiki folder, use `file_place` (see `optale-files`).
