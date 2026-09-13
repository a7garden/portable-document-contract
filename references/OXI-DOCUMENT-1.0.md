# OXI Document 1

Status: owner standard, draft 1

Date: 2026-09-13

Document identifier: `oxi-document/1`

Body identifier: `oxi-djot/1`

This document is normative. The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** describe conformance requirements.

## 1. Scope and compatibility promise

OXI Document 1 governs durable, user-authored notes and documents that owner-created applications may open, index, edit, link, embed, or migrate.

It does not govern repository documentation, source-code comments, prompts, logs, caches, generated previews, exported reports, immutable event ledgers, or database-internal records unless they are explicitly promoted to the shared document plane.

Two conforming applications given the same vault MUST:

1. discover the same conforming documents;
2. either open each document or show an explicit unsupported/invalid diagnostic;
3. agree on document identity, standard metadata, links, managed assets, task state, and block targets;
4. preserve data they do not understand;
5. never silently hide, discard, or reinterpret a conforming document during a write.

Compatibility does not require identical CSS, pixel-identical previews, identical search ranking, or execution of another app's optional feature. Unsupported features MUST retain a readable fallback and survive round trips.

## 2. Versioning

- `oxi-document/1` identifies the envelope, storage, identity, and semantic contract in this document.
- `oxi-djot/1` identifies the frozen body dialect.
- Compatible clarifications and new optional fields may be added without changing the major identifier.
- A change that makes a previously conforming document parse differently, changes required semantics, removes a field, or permits destructive down-conversion MUST use a new major identifier.
- Readers MUST compare identifiers as exact, case-sensitive strings. An unknown major version is unsupported, not malformed.
- Implementations MUST be tested against a named conformance-corpus revision. Depending only on a parser package version is insufficient.

The upstream Djot syntax baseline is commit [`d77f8a0cbea6785c42b3e2b03463195b5ca6f7c7`](https://github.com/jgm/djot/tree/d77f8a0cbea6785c42b3e2b03463195b5ca6f7c7). Upstream changes do not alter `oxi-djot/1` until incorporated here with fixtures.

## 3. Vault and discovery

A vault is a user-selected directory. A document path never defines document identity.

### 3.1 Vault manifest

A writable vault MUST contain `.oxi/vault.json`:

```json
{
  "format": "oxi-vault/1",
  "id": "018f47c6-1199-72b3-87de-344e5a493e27",
  "created": "2026-09-13T12:34:56.789Z"
}
```

- `format`, `id`, and `created` are required.
- The vault ID follows the document-ID rules in section 7.
- A Reader MAY open a manifest-free directory as an uninitialized vault.
- A Writer MUST create the manifest before its first canonical write, without moving or rewriting existing files.
- Unknown manifest properties MUST be preserved.

### 3.2 Document discovery

- Canonical documents use the lowercase `.djot` extension. `.dj`, `.md`, `.markdown`, and `.html` are not canonical aliases.
- Readers MUST recursively discover regular `.djot` files below the vault root.
- Readers MUST NOT scan `.oxi`, `.git`, or any path with a dot-prefixed component.
- Readers MUST NOT follow symbolic links during recursive discovery.
- File ordering MUST NOT affect identity or conflict resolution.
- Every discovered `.djot` file MUST appear as a document or as a visible diagnostic. Silent skipping is nonconforming.
- Duplicate document IDs are vault errors. An app MUST show every conflicting path and MUST NOT choose a winner silently.

Legacy formats may be discovered by separate adapters. They MUST NOT be classified as canonical OXI documents merely because their body resembles Djot.

## 4. File transport

- A canonical document is UTF-8 without a byte-order mark. A Reader MUST reject a byte-order mark as `invalid_transport`; it MUST NOT strip one silently.
- A canonical document is at most 4 MiB, including envelope and body.
- A canonical body has at most 256 simultaneously open block containers. Implementations MAY impose lower presentation limits only if they still preserve the source and show a visible limitation; they MUST NOT claim Reader conformance for a document they cannot parse to this limit.
- The internal media type is `application/vnd.oxi.document+djot;version=1`. It is an ecosystem identifier, not a claim of IANA registration.
- Writers create LF line endings. Readers MUST accept LF or CRLF and interpret them equivalently.
- Writers MUST NOT normalize an untouched document merely because it was opened.
- The first line is exactly `---`. The next exact `---` line closes the envelope. The remainder is the Djot body and may be empty.
- A missing, empty, duplicated, or unclosed envelope is malformed.
- Frontmatter-free `.djot` is valid upstream Djot but is an invalid OXI transport. A Reader reports `invalid_transport` and MAY offer an explicit foreign-Djot import; it never presents the file as a conforming document silently.

## 5. Envelope grammar

The envelope uses the frozen OXI constrained-YAML grammar, not general YAML.

- Keys are nonempty, case-sensitive strings at indentation level zero.
- A value is a Boolean, single-line string, flat string sequence, literal block string, or one nested map.
- Nested-map children use exactly two spaces. Deeper maps and sequences of maps are forbidden.
- Duplicate keys, tabs, comments, anchors, aliases, tags, complex keys, multi-document streams, and empty values are forbidden.
- `true` and `false` are Booleans. Other bare or quoted scalars are strings; numbers and timestamps are not implicit numeric or date types.
- Flow sequences are single-line flat string sequences. Block sequences are accepted by Readers, but Writers SHOULD emit flow sequences.
- CRLF is normalized only for envelope parsing. The original body slice remains distinct for preservation rules.
- A malformed opening envelope is a hard error, never body text.

General-purpose YAML libraries MAY be used only behind validation that rejects every feature forbidden above. They MUST use safe loading and MUST NOT instantiate tagged objects.

### 5.1 Required fields

| Field | Type | Requirement |
|---|---|---|
| `format` | string | Exact value `oxi-document/1` |
| `body` | string | Exact value `oxi-djot/1` |
| `id` | string | Canonical lowercase UUID |
| `created` | string | Canonical UTC timestamp |
| `updated` | string | Canonical UTC timestamp, not earlier than `created` |
| `title` | string | Display title; may be empty |

Canonical timestamps use `YYYY-MM-DDTHH:MM:SS.sssZ`, with exactly millisecond precision and a real Gregorian calendar date and time. Readers MAY accept other RFC 3339 forms only in a legacy importer; canonical Writers MUST emit the form above.

When `title` is empty, the display fallback is the plain text of the first level-one heading, then the filename stem. The stored title remains authoritative when nonempty.

### 5.2 Standard optional fields

| Field | Type | Default / semantics |
|---|---|---|
| `profile` | string | `note`; lower kebab-case token |
| `lang` | string | Unspecified; BCP 47 language tag |
| `tags` | string sequence | Empty; case-sensitive, duplicate-free labels |
| `aliases` | string sequence | Empty; alternate human-facing titles |
| `favorite` | Boolean | `false` |
| `deleted` | Boolean | `false`; excluded from default lists but available to trash/recovery views |
| `deleted_at` | string | Canonical UTC timestamp; MUST be present exactly when `deleted` is true |

Writers MUST preserve Unicode text. They MUST NOT silently case-fold or normalize titles, aliases, or body text. New tags SHOULD use Unicode NFC; tag comparison SHOULD compare NFC without changing stored spelling.

### 5.3 Extensions and unknown fields

- Unprefixed field names are reserved for future OXI Document versions.
- An application-specific extension MUST be a top-level map named `x_<namespace>`, where the namespace is lowercase ASCII letters and digits beginning with a letter.
- Owner-reserved namespaces are `x_oximemo`, `x_sawhorse`, `x_oxibrain`, `x_farm`, and `x_lexi`. Their corresponding app is the only Writer allowed to create or reinterpret values in that map.
- A new app claims a namespace by adding it to this registry before shipping writes. Renaming or transferring a claimed namespace is a standard change with a migration plan.
- A Writer MUST NOT create another application's namespace.
- Readers and Mutators MUST preserve unknown unprefixed fields and unknown extension maps.
- An unknown field is not permission to treat a document as malformed unless its serialized value violates the envelope grammar.

### 5.4 Canonical key order

New documents and full canonical rewrites emit known keys in this order:

`format`, `body`, `id`, `created`, `updated`, `title`, `profile`, `lang`, `tags`, `aliases`, `favorite`, `deleted`, `deleted_at`.

Unknown unprefixed fields retain observed order. Extension maps follow, sorted by namespace. Reordering alone MUST NOT trigger a rewrite of an existing file.

## 6. Body dialect: `oxi-djot/1`

The body uses the pinned Djot syntax with these constraints:

- Raw inline and raw block nodes in any output format are nonconforming. In particular, embedded raw HTML is forbidden.
- The only attributes with standard semantics are `id`, `class`, `lang`, `title`, and `data-oxi-*`. Unknown attributes MUST survive source-preserving edits but MUST NOT be copied blindly into rendered HTML.
- Class names beginning `oxi-` and attributes beginning `data-oxi-` are reserved by this standard.
- App-specific classes use `x-<namespace>-<name>`. App-specific attributes use `data-x-<namespace>-<name>`.
- A body parser MUST retain source ranges or another lossless representation sufficient for the write guarantees in section 10.
- Writers creating or fully regenerating a body use LF, place a blank line between block elements, omit trailing spaces, and end a nonempty body with one LF. Existing authored layout is preserved unless a deliberate format action is requested.
- An implementation that accepts syntax beyond this dialect does so only as an importer. It MUST NOT write the extra syntax while labeling the body `oxi-djot/1`.

Standard Djot structures—headings, paragraphs, emphasis, strong text, links, images, autolinks, verbatim text, highlight, superscript, subscript, insert/delete, math, footnotes, lists, task items, code blocks, divs, and pipe tables—must remain parseable and preservable. A viewer MAY use a readable textual fallback for math or another presentation feature it cannot render.

## 7. Identity and block targets

### 7.1 Document identity

- `id` is the sole canonical document identity.
- It is a hyphenated lowercase UUID string.
- Writers MUST generate UUIDv7 for new documents. Readers MUST accept any valid UUID version for migrated documents.
- Moving or renaming a file MUST NOT change its ID.
- App-local database IDs, filenames, paths, titles, and legacy human IDs MUST NOT replace it.

### 7.2 Block identity

- A block becomes a stable target when it has a Djot block attribute `#b-<uuid>` immediately before it.
- The UUID follows the document-ID rules; Writers generate UUIDv7.
- Once assigned, a block ID MUST survive edits, moves within the document, and round trips.
- An app MUST NOT assign IDs to every block merely by opening a document. It MAY assign one when a link, task identity, comment, or feature needs a stable target.
- Duplicate block IDs within one document are invalid. The app MUST surface the conflict before writing.

Example:

```text
{#b-018f47c6-7dbe-7a14-9f67-6f89a5e3cc32}
## Stable section
```

## 8. Links, embeds, and assets

### 8.1 Document links

Canonical internal links use:

```text
[Readable label](oxi://document/<document-uuid>)
[Readable label](oxi://document/<document-uuid>#b-<block-uuid>)
```

- Resolution is by UUID within the current vault.
- `oxi:` is a private ecosystem URI scheme, not an IANA registration and not an operating-system protocol-handler requirement.
- The label is fallback content and MUST remain readable when unresolved.
- An unresolved target is a visible broken-link state, not a reason to rewrite the URL.
- `[[wiki links]]`, app-specific custom schemes, and path-only note links are legacy or extension syntax, not canonical internal links.

A document embed is the same link with class `oxi-embed`:

```text
[Embedded note](oxi://document/018f47c6-4a77-7c52-9db8-0e5f9bcb17db){.oxi-embed}
```

A viewer that does not support inline embedding MUST render the ordinary link fallback.

### 8.2 Managed assets

- Managed asset bytes are addressed by lowercase SHA-256 digest.
- The canonical URI is `oxi://asset/sha256/<64-hex-digest>`.
- The canonical vault path is `.oxi/assets/sha256/<first-two-hex>/<64-hex-digest>`.
- A managed asset is at most 64 MiB. Larger resources remain external or require a future extension with explicit streaming semantics.
- Stored bytes MUST hash to the URI digest. A mismatch is a visible integrity error.
- Original filename and media type MAY be carried as `data-oxi-filename` and `data-oxi-media-type` attributes. They are hints, not identity.
- Writers MUST use an atomic create-if-absent operation and MUST NOT overwrite different bytes at an existing digest path.
- Apps MUST NOT garbage-collect unreferenced assets automatically. Garbage collection requires an explicit maintenance action, a complete reference scan, and a recoverable quarantine period.
- HTTP(S) resources are external links and may be rendered subject to policy. Relative file URLs, absolute filesystem paths, `file:`, `data:`, and app-specific asset schemes are not canonical managed assets.

Example:

```text
![Diagram](oxi://asset/sha256/0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef){data-oxi-filename="diagram.png" data-oxi-media-type="image/png"}
```

## 9. Standard semantic constructs

### 9.1 Tasks

Djot task items are canonical:

```text
- [ ] Open task
- [x] Completed task
```

`[ ]` means open and `[x]` or `[X]` means completed. A Mutator that exposes task toggling MUST change only the task marker and required metadata timestamps.

A task that needs stable identity wraps its complete leading label in an `oxi-task` span whose ID follows the block-ID grammar:

```text
- [ ] [Open task]{#b-018f47c6-c718-728c-9d91-b2bc700814bb .oxi-task}
```

Apps recognize the ID only when the span is the task item's first inline child. A plain task item remains valid but cannot be a stable link target. Once present, the task ID follows the stability and duplicate rules in section 7.2.

### 9.2 Query blocks

An executable query is a fenced code block whose language is `oxi-query`:

````text
``` oxi-query
tag = "project"
```
````

The query text is opaque in Document 1. Query execution belongs to a separately versioned query contract. Apps that do not implement it MUST show the code block. They MUST NOT execute, delete, or rewrite it.

### 9.3 Extension fallback

Every new semantic extension MUST have a fallback expressible as ordinary Djot text, link, code, or container content. Removing the extension class/attribute must leave understandable content. Opaque binary editor state is forbidden as the only representation of user content.

## 10. Read, write, and preservation rules

### 10.1 No-op and partial writes

- Opening, previewing, indexing, or closing a document without user-visible changes MUST leave every byte unchanged.
- A metadata-only mutation MUST preserve body bytes exactly.
- A body-only mutation SHOULD preserve untouched envelope spelling and unknown-field order; it MUST preserve all envelope values.
- Updating content or standard metadata MUST update `updated`. A no-op MUST NOT update it.
- Automatic ID assignment is a mutation and requires an actual feature need; it MUST NOT happen during read-only access.

### 10.2 Unknown and unsupported content

- Unknown envelope fields, extension maps, Djot attributes, and readable extension fallbacks MUST survive writes.
- If an editor's internal model cannot preserve a construct, the app MUST make the document read-only or require an explicit lossy conversion that lists the affected constructs.
- A lossy conversion MUST write to a new target until the user explicitly authorizes replacement of the source.

### 10.3 Concurrency and durability

- Writers MUST compare the current source bytes or digest with the snapshot that editing began from before replacement.
- If the source changed externally, the app MUST block overwrite, merge with conflict reporting, or create a conflict copy. Last-writer-wins without notice is forbidden.
- File replacement MUST be atomic on the target filesystem.
- A Writer MUST fsync the temporary file before rename when the platform exposes that primitive; durability of the containing directory SHOULD also be requested.
- Errors MUST retain the previous valid file and surface an actionable diagnostic.

## 11. Rendering and security

Documents and renderer output are untrusted input.

- Previewers MUST sanitize generated HTML or construct a safe native view tree.
- Raw HTML is forbidden even though upstream Djot can represent it.
- `javascript:`, `data:`, `file:`, and unknown schemes MUST NOT be navigated or loaded automatically.
- `oxi:` URIs MUST be handled by an internal resolver, never handed directly to a browser or operating system.
- Event-handler attributes, scripts, stylesheets, iframes, plugins, and executable embeds MUST be removed or inert.
- External HTTP(S) resources SHOULD require the app's normal privacy/network policy and MUST NOT be fetched merely for indexing.
- Attribute keys and values MUST be allowlisted when converted to HTML. Source preservation and render exposure are separate decisions.
- The shared render policy unwraps unknown generated elements while preserving their children, but removes the entire subtree of explicitly blocked executable elements. A conforming canonical body cannot rely on a blocked subtree for user content because raw nodes are already invalid.

## 12. Conformance roles

### Reader

A Reader MUST implement discovery, transport and envelope validation, the pinned body parse, standard field semantics, safe preview or source fallback, UUID link resolution, managed-asset verification, and visible diagnostics.

### Writer

A Writer is a Reader that can create canonical documents and vault manifests, generate UUIDv7 IDs, emit canonical timestamps, respect the size limit, and write atomically with external-change detection.

### Mutator

A Mutator is a Writer that can modify existing documents while satisfying byte preservation, unknown-data preservation, conflict detection, and read-only downgrade rules.

An app MUST declare its role. It MUST NOT claim a role until it passes every applicable shared fixture for that role in its shipping implementation. Corpus success is necessary but not sufficient: the shipping implementation also tests every normative requirement applicable to its role.

## 13. Diagnostics

At minimum, apps distinguish:

- `invalid_transport`
- `invalid_envelope`
- `unsupported_document_version`
- `unsupported_body_version`
- `invalid_document_id`
- `duplicate_document_id`
- `duplicate_block_id`
- `document_too_large`
- `document_too_complex`
- `missing_asset`
- `asset_digest_mismatch`
- `unsafe_content`
- `unsupported_lossless_edit`
- `external_change_conflict`

Diagnostics include the path and a human-readable reason. Parse diagnostics SHOULD include a line and column. Indexing one bad document MUST NOT hide or prevent access to unrelated valid documents.

## 14. Legacy migration and interchange

- Markdown, HTML, `shdoc/1`, and other formats are importer inputs or exporter outputs, not alternate canonical bodies.
- Existing apps SHOULD use dual-read and canonical-write-for-new-documents before offering conversion.
- Importers MUST map identity, timestamps, links, assets, task state, and app semantics explicitly.
- When a legacy ID parses as a UUID in a noncanonical spelling, preserve the UUID value and emit its canonical lowercase hyphenated spelling.
- When a legacy ID is not a UUID, allocate a UUIDv7 and preserve the original in the source app's extension map.
- Oximemo v4 uses `deleted` as a deletion timestamp string. Its importer maps that value to `deleted: true` plus the same instant normalized into `deleted_at`; absence maps to `deleted: false`.
- Importers MUST report dropped, approximated, externalized, or unsupported constructs before source replacement.
- Export does not change the canonical source and SHOULD state the target dialect, because “Markdown” alone is not a sufficient format identifier.

## 15. Change governance

The standard repository is the source of truth. A change is complete only when it updates, as applicable:

1. this specification;
2. the parsed-envelope schema;
3. positive and negative fixtures;
4. adoption or migration guidance;
5. participating implementations and their pinned corpus revision.

No single app implementation silently defines the standard. When the specification and an implementation differ, report the difference and change one deliberately; do not normalize the conflict away.

## Appendix A: minimal canonical document

```text
---
format: oxi-document/1
body: oxi-djot/1
id: 018f47c6-4a77-7c52-9db8-0e5f9bcb17db
created: 2026-09-13T12:34:56.789Z
updated: 2026-09-13T12:34:56.789Z
title: First document
profile: note
favorite: false
deleted: false
---
# First document

This body is Djot, not an unspecified Markdown dialect.
```

## Appendix B: the 100% guarantee

For this standard, “100% compatible” means that every conforming app can find, identify, parse, safely present, link, and preserve every conforming document without silent loss. An app may lack an optional execution feature, but the source remains readable and intact. Files outside the contract—legacy Markdown, legacy HTML, malformed documents, and newer major versions—must still be visible as legacy, invalid, or unsupported rather than disappearing.
