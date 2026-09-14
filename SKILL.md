---
name: portable-document-contract
description: Apply or audit the Markdown-and-HTML Portable Document Contract when an app stores, reads, edits, indexes, migrates, links, embeds, or renders durable user-authored notes or documents, including readable legacy PDC 1 Djot and HTML files. Do not apply it to repository documentation, source-code Markdown, logs, caches, generated previews, or export-only files.
---

# Portable Document Contract

Use this skill to keep the document plane of participating apps interoperable.

Canonical repository: [a7garden/portable-document-contract](https://github.com/a7garden/portable-document-contract). The installed skill directory is a working copy of that source.

Before designing or changing a canonical user-document format, read [references/PDC-2.0.md](references/PDC-2.0.md) completely. For query blocks and `.base` files, read [references/PDC-QUERY-1.0.md](references/PDC-QUERY-1.0.md). For a new app, an existing-format migration, editor integration, or compatibility audit, also read [references/adoption.md](references/adoption.md).

## Working rules

- Determine whether the data is a durable user-authored document. Repository docs, prompts, logs, caches, generated HTML, database records, and import/export artifacts stay outside this standard unless the user explicitly promotes them to the canonical document plane.
- Treat `pdc-document/2` as the shared contract with two canonical body profiles: `pdc-markdown/1` (lowercase `.md`, Obsidian-compatible, the default for ordinary notes) and `pdc-html/1` (first-class for authored HTML structure, layout, or inline CSS). The optional `pdc-query/1` contract covers `.base` files and fenced `base` code blocks; do not invent a competing query surface.
- PDC 1 documents (`pdc-djot/1` and `pdc-html/1` under `pdc-document/1`) are mandatory readable legacy inputs. Never convert, rewrite, or re-emit them automatically. Conversion is explicit, user-authorized, new-target-first, with backup and a machine-readable loss report.
- The Markdown baseline is CommonMark 0.31.2 plus GFM tables, strikethrough, task lists, and autolinks. Preserve source extensions byte-for-byte. Recognize Obsidian-compatible wiki links/embeds, callouts, `==highlight==`, `%%comments%%`, math, footnotes, tags, and caret block IDs without inventing a different base grammar. Raw HTML is preserved in source and always sanitized or inert in preview.
- The envelope is safe general YAML 1.2 Core: unique string keys, JSON-compatible values, comments and nesting permitted; anchors, aliases, custom tags, complex keys, duplicate keys, and multi-document streams forbidden. Unknown top-level properties are user properties: preserve them losslessly and keep them queryable.
- Treat interoperability as discoverability, parsing, stable identity, core semantic behavior, safe rendering, and lossless preservation—not pixel-identical presentation or support for every optional app feature.
- If the standard does not define a needed semantic, use a namespaced user property or extension with a readable fallback and preserve it in every other app. Do not silently invent new unprefixed contract fields or `pdc-*` syntax.
- An app that cannot preserve a construct must keep the document read-only or report a conversion conflict. It must never silently omit, hide, or destructively rewrite an in-scope document.
- New apps write only canonical PDC 2 profiles. Existing apps migrate with dual-read (v1 and v2) and canonical-write-for-new-documents. Bulk conversion requires explicit user authorization, backup, and a per-document loss report.
- Add the shared conformance fixtures (`pdc-document-conformance/2`, revision 2, including legacy readability cases) to each implementation's tests. A parser library version or successful render alone is not evidence of conformance.
- Propose changes in the canonical standard repository first. A breaking syntax or semantic change requires a new major identifier and a migration path.

This skill guides format and implementation decisions; it does not itself authorize editing app repositories, converting user files, publishing, or deleting legacy data.
