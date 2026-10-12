---
name: optale-objects
description: Use when the user asks to create, add, record, sync, update, rename, link, unlink, archive, restore or delete Objects or Relationships in an Optale Console, such as "add this company", "set the deal to won", "link this person to that project" or "import these rows". Covers duplicate checks, create versus ensure, what an update replaces, and Relationship writes. Not for finding or reading data; use optale-find-and-read. Not for Wiki pages or files; use optale-wiki or optale-files.
license: MIT
---

# Write Objects and Relationships

## Before any write

1. Read the target type with `object_definition_list` and `objectSlug`. Note required Properties, option values for select Properties, and computed Properties (the server derives those; do not write them).
2. Actor Properties (people) take user ids from the Property's `choices`, not names.
3. File and Wiki-page Properties have their own value shapes. `object_read` on an Object of that type lists them per Property in `writeShapes`, and a validation error repeats them.

## Create without duplicates

1. Search first: `object_search` with `objectSlug` and the display name, then with its most distinctive word. Leave `fields` out; identity rows are enough to spot a duplicate. Remember that matching is contiguous text.
2. If a match is the same thing, update it instead of creating another.
3. Otherwise call `object_create` with `objectSlug`, `displayName` and `properties`.
4. The response lists `similarObjects` (same type, overlapping names, score 0 to 1). When it is not empty, tell the user which existing Objects look alike.

## Create or update from another system: `object_ensure`

Use `object_ensure` when the data comes from a system with its own ids (a CRM, a ticket tracker, a spreadsheet with a key column). It finds the Object by `source` + `externalId` and creates or updates it, with its Relationships, in one call.

- Required: `scope`, `operationKey`, `source`, `externalId`, `displayName`, and `object` with `objectSlug` and `properties`.
- `operationKey` identifies this one request. Repeating the identical request with the same key replays or resumes it safely. A different request needs a new key; reusing a key with a different body returns `blocked-by-conflict`.
- `relationships` takes up to 64 items, each with `relationshipDefinitionSlug` and exactly one of `targetObjectId` or `targetExternalRef` (`system`, `externalType`, `externalId`).
- The result lists each step and its status. `OPTALE_OBJECT_ENSURE_INCOMPLETE` means some steps did not finish; read the steps.

Run the same duplicate search first when the Console may already hold the Object without that external id.

## Update

Call `object_update` with `objectId` and only the Properties you are changing. Properties you leave out stay as they are.

A single-value Property holds one value, so writing it replaces the current value in the same call. The response then lists `replacedValues`: for each Property, the old value, the new value and `restoreFrom`, the Object history event (or Shared Value revision) that holds the old value. Tell the user what was replaced.

To attach a file to a file Property, use `file_place` (see `optale-files`). To give an Object a Wiki page, see `optale-wiki`.

## Relationships

1. Find the Relationship type with `relationship_definition_list`. Each definition names its source and target Object types; pick the direction that matches.
2. `relationship_create` with `sourceObjectId`, `targetObjectId` and `relationshipDefinitionSlug`. A missing or unknown slug is refused with the definitions those two Objects admit; choose from that list.
3. `metadata` holds only the scalar edge Properties the definition declares. Link to another Object with another Relationship, never inside metadata.
4. `relationship_update` replaces the whole metadata map. Send every key you want to keep.
5. `relationship_end` closes a Relationship but keeps its history; `relationship_reactivate` restores an ended one. `relationship_delete` removes it.

Use `relationship_list` on the source Object to check what already exists before you create a link twice.

## Archive and delete

- `object_archive` hides an Object from normal lists and keeps everything; `object_unarchive` brings it back.
- `object_delete` removes the Object permanently.

## After a write

- Quote the `link` from the response so the user can open the Object.
- When the result matters for your next step, read the target back with `object_read`.
- A write that timed out with `details.mayHaveCommitted: true` may have landed. Read the target before you try again.
- A validation error names the field and the allowed values. Correct that field and call once more.
