---
name: optale-files
description: Use when the user asks to upload, store, import, attach, place, rename, move, keep, view, read, edit, export or delete a file in an Optale Console, such as "save this PDF to the contract", "put the logo in the brand folder", "what does this scan say" or "import the attachment from that email". Covers file_place, file_import_source, file_get, file_view, file_read_bytes, file_patch, file_move, file_delete and the code-workspace tools. Not for Markdown Wiki pages; use optale-wiki.
license: MIT
---

# Files in the Optale Console

A file is stored once and identified by a locator: `{"kind": "file", "file_id": "..."}`. The same file can be **placed** in several Wiki folders and **attached** to Object file Properties without copying its bytes. My Files holds the files a person keeps.

## Find a file

- `file_get` with `path` (a file in a Wiki folder) or `locator` returns metadata and every placement, never bytes. With `list: true` it pages through My Files.
- `source_search` with `kinds: ["file"]` finds files by name and content.
- An Object's file Properties hold locators; read them with `object_read`.

## Read a file

- `file_view` shows images and PDF pages to you as images, and returns PDF text where it exists. It is the first choice for scans, screenshots and photos of documents. Use `pages` (one-based) for specific PDF pages. Images need a model that can see.
- `file_read_bytes` returns up to 10 MiB as base64, for files you process yourself. `revision` reads an earlier version.

## Store bytes you have

`file_place` with `contentBase64` writes a whole file. Encode the bytes as canonical base64; the cap is 10 MiB per file.

- **Into a Wiki folder**: add `parentPath` and `filename`, or an exact `path`. Writing to a path that already holds a file replaces that file; pass the `expectedRevision` you last read to avoid replacing someone else's change.
- **As a new revision of a known file**: add `locator`.
- **Onto an Object**: add `recordId` (the Object id) and `targetProperty` (a file Property key from `object_definition_list`).

The response lists `replacedValues` when an Object's Property value was replaced; tell the user what was replaced. Pass a stable `fileOperationKey` when you may retry the same upload.

## Place or attach a file that already exists

- Into a Wiki folder: `file_place` with `uploadFileId` set to its `file_id`, plus `parentPath` and `filename` (or `path`). This adds a placement; it never copies the bytes.
- Onto an Object file Property: `file_place` with `locator`, `recordId` and `targetProperty`.
- An Object type without a file Property takes the file through the Wiki: place the file at a Wiki path, then `object_wiki_link_create` with the `objectId` and that `wikiPath`.

## Import from another system

`file_import_source` stores a file from a source reference and returns its locator: `{"kind": "url", "url"}` for an attachment URL a connected provider returned, or `{"kind": "upload", "fileId"}` for a file uploaded to the Console. Pass the source's own `filename` and `mimeType` when you have them; a signed URL carries neither, and the file would otherwise be stored under the URL's hash. Identical imports are deduplicated.

## Change, move and delete

- `file_patch` edits a text file in place: replace one exact `oldText` with `newText`, or append. Target it by `locator` or `path` and pass `expectedRevision`. It refuses no match, more than one match, and non-text files.
- `file_move` renames a file (`locator` + `filename`) or moves one Wiki placement (`newPath`, or `parentPath` + `filename`); the file itself and its other placements stay. `kept: true` keeps a file in My Files; `kept: false` drops only the person's own keep.
- `file_delete` by `locator` or `path` deletes a file. A file that is still placed somewhere is refused with `FILE_IN_USE`; `details.placements` names every place. Wiki pages are deleted with `wiki_delete`.

## The Console's code workspace

`file_import_workspace` copies a stored file into the Console's code workspace so code running there can read it; wait for its result before using the path. `file_export_workspace` turns a file that code produced there into a durable file, optionally placed in a Wiki folder. Both take a code workspace `session_id`. An agent working on its own machine does not need them: read bytes with `file_read_bytes` and store them with `file_place`.
