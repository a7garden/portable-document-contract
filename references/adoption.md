# Adopting Portable Document Contract 1

Read the normative specification first. This guide explains how to apply it without broadening its scope.

## Decide whether the standard applies

It applies when an app persists durable text that a person understands as a note or document and another owner-created app may open, index, edit, link, embed, or migrate.

It does not automatically apply to:

- `README.md`, `AGENTS.md`, design docs, changelogs, and source-controlled prose;
- prompts, logs, telemetry, queues, immutable event ledgers, database-only records, and caches;
- generated previews, reports, HTML renderings, and temporary editor state;
- Markdown or HTML accepted only as an import or emitted only as an export.

If a document-like record must remain in a database for product reasons, expose a `pdc-document/1` import/export adapter only when cross-app document interchange is actually required. Do not force the database's internal schema to mimic a file.

## New applications

1. Choose the conformance role: Reader, Writer, or Mutator.
2. Discover lowercase `.djot` files according to the vault rules and surface every invalid or unsupported document as a visible diagnostic.
3. Parse the envelope with the frozen constrained grammar, then parse the body as `pdc-djot/1` rather than "whatever Markdown this library accepts."
4. Resolve document links by UUID and managed assets by digest. Never make a path, title, or app-local database ID the canonical identity.
5. Use a Djot-aware source editor or plain-text editor. A Markdown-only editor may be used only for legacy imports, not as the canonical writer.
6. Render through a sanitizer. Raw renderer output is untrusted even when the Djot implementation produced it.
7. Preserve unknown extension metadata and unsupported source constructs. If the editor cannot do so, downgrade that document to read-only.
8. Run the shared conformance fixtures in the app's native stack. Add round-trip, external-change, duplicate-ID, invalid-file visibility, and sanitizer tests.

## Existing applications

Use this order:

1. Inventory current transports, metadata, links, assets, semantic extensions, discovery rules, and editor round-trip behavior.
2. Add a PDC Reader without changing existing files.
3. Make invalid or unsupported `.djot` files visible instead of silently skipping them.
4. Write `pdc-document/1` for newly created documents while retaining legacy readers.
5. Add explicit importers from each legacy format. Preserve legacy IDs in the owning app's extension map when a UUID must be allocated.
6. Recalculate format-derived indexes such as task, tag, heading, and body hashes from the converted Djot body; never carry Markdown-derived projections forward as if they were still valid.
7. Offer per-document or user-authorized batch conversion with backup and a machine-readable loss report.
8. Retire legacy writing only after the fixture suite and representative real-vault tests pass in every participating app.

Never rewrite a vault merely because an app was upgraded. Opening and closing an untouched document must leave its bytes unchanged.

## Editor and preview integration

- CodeMirror, Atomic Editor, or another Markdown-specific editor is not automatically a Djot editor. Disable Markdown-only syntax transforms for canonical documents until they are implemented against the pinned Djot grammar.
- The safe first implementation is plain source editing plus a separate Djot preview. A richer editor is conforming only when unsupported nodes and attributes survive edits.
- Preview parity means the same document structure and core semantics, not identical CSS. Each app may style the rendered tree differently.
- Link clicks, embeds, tasks, query blocks, and asset URLs must go through PDC resolvers; do not hand custom URI schemes directly to a browser engine.

## Repository integration

For an in-scope app, add a short repository-local note only if it adds useful context beyond the global instruction. Prefer wording like:

> User-authored durable documents in this repository follow Portable Document Contract 1. Use the `portable-document-contract` skill before changing their storage, editor, renderer, discovery, linking, asset, or migration behavior. Repository documentation is out of scope.

Pin tests to the fixture corpus revision used by the app. When the standard changes, update the parser, writer, migrations, and fixture revision in one coherent change.

## Review checklist

- Does every participating app find the same valid files in the same vault?
- Does every invalid or unsupported file produce a visible, actionable diagnostic?
- Are document identity, links, and assets independent of filenames and app-local databases?
- Are metadata-only edits body-byte-preserving?
- Are unknown extensions preserved or the document made read-only?
- Are writes atomic and guarded against external changes?
- Is preview output sanitized and are unsafe schemes blocked?
- Can legacy import report every lossy transformation before replacing a source?
- Do Rust, TypeScript, Swift, and any other implementation consume the same fixtures?
