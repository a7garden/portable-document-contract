---
name: oxi-document-standard
description: Apply or audit the OXI Document contract when an owner-created app stores, reads, edits, indexes, migrates, links, embeds, or renders durable user-authored notes or documents. Do not apply it to repository documentation, source-code Markdown, logs, caches, generated previews, or export-only files.
---

# OXI Document Standard

Use this skill to keep the document plane of the owner's apps interoperable.

Canonical repository: [project-oxi/oxi-document-standard](https://github.com/project-oxi/oxi-document-standard). The installed skill directory is a working copy of that source.

Before designing or changing a canonical user-document format, read [references/OXI-DOCUMENT-1.0.md](references/OXI-DOCUMENT-1.0.md) completely. For a new app, an existing-format migration, editor integration, or compatibility audit, also read [references/adoption.md](references/adoption.md).

## Working rules

- Determine whether the data is a durable user-authored document. Repository docs, prompts, logs, caches, generated HTML, database records, and import/export artifacts stay outside this standard unless the user explicitly promotes them to the canonical document plane.
- Treat `oxi-document/1` and `oxi-djot/1` as the default canonical write format. Do not introduce an app-specific canonical Markdown, HTML, JSON, or rich-text format for in-scope documents.
- Treat interoperability as discoverability, parsing, stable identity, core semantic behavior, safe rendering, and lossless preservation—not pixel-identical presentation or support for every optional app feature.
- If the standard does not define a needed semantic, use a namespaced extension with a readable fallback and preserve it in every other app. Do not silently invent new unprefixed fields or `oxi-*` syntax.
- An app that cannot preserve a construct must keep the document read-only or report a conversion conflict. It must never silently omit, hide, or destructively rewrite an in-scope document.
- New apps write only the canonical format. Existing apps migrate with dual-read and canonical-write-for-new-documents; bulk conversion requires explicit user authorization, backup, and a per-document loss report.
- Add the shared conformance fixtures to each implementation's tests. A parser library version or successful render alone is not evidence of conformance.
- Propose changes in the canonical standard repository first. A breaking syntax or semantic change requires a new major identifier and a migration path.

This skill guides format and implementation decisions; it does not itself authorize editing app repositories, converting user files, publishing, or deleting legacy data.
