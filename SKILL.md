---
name: portable-document-contract
description: Apply or audit the Djot-and-HTML Portable Document Contract when an app stores, reads, edits, indexes, migrates, links, embeds, or renders durable user-authored notes or documents. Do not apply it to repository documentation, source-code Markdown, logs, caches, generated previews, or export-only files.
---

# Portable Document Contract

Use this skill to keep the document plane of participating apps interoperable.

Canonical repository: [a7garden/portable-document-contract](https://github.com/a7garden/portable-document-contract). The installed skill directory is a working copy of that source.

Before designing or changing a canonical user-document format, read [references/PDC-1.0.md](references/PDC-1.0.md) completely. For a new app, an existing-format migration, editor integration, or compatibility audit, also read [references/adoption.md](references/adoption.md).

## Working rules

- Determine whether the data is a durable user-authored document. Repository docs, prompts, logs, caches, generated HTML, database records, and import/export artifacts stay outside this standard unless the user explicitly promotes them to the canonical document plane.
- Treat `pdc-document/1` as the shared contract with two canonical body profiles: `pdc-djot/1` for ordinary notes and `pdc-html/1` when authored HTML structure, layout, or inline CSS is part of the source's value. Do not demote conforming HTML to an import-only format or invent an app-specific canonical Markdown, HTML, JSON, or rich-text dialect.
- A fully compatible app discovers and safely presents both canonical profiles. Writers declare which profiles they create; a Mutator may keep an unsupported profile source-only or read-only while preserving it losslessly.
- Treat interoperability as discoverability, parsing, stable identity, core semantic behavior, safe rendering, and lossless preservation—not pixel-identical presentation or support for every optional app feature.
- If the standard does not define a needed semantic, use a namespaced extension with a readable fallback and preserve it in every other app. Do not silently invent new unprefixed fields or `pdc-*` syntax.
- An app that cannot preserve a construct must keep the document read-only or report a conversion conflict. It must never silently omit, hide, or destructively rewrite an in-scope document.
- New apps write only canonical PDC profiles. Existing apps migrate with dual-read and canonical-write-for-new-documents; legacy HTML that satisfies `pdc-html/1` should retain its body rather than be converted to Djot. Bulk conversion requires explicit user authorization, backup, and a per-document loss report.
- Add the shared conformance fixtures to each implementation's tests. A parser library version or successful render alone is not evidence of conformance.
- Propose changes in the canonical standard repository first. A breaking syntax or semantic change requires a new major identifier and a migration path.

This skill guides format and implementation decisions; it does not itself authorize editing app repositories, converting user files, publishing, or deleting legacy data.
