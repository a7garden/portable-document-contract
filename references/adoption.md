# Adopting Portable Document Contract 1

Read the normative specification first. This guide explains how to apply the two canonical body profiles without broadening the shared document plane.

## Decide whether the standard applies

PDC applies when an app persists durable text that a person understands as a note or document and another participating app may open, index, edit, link, embed, or migrate.

It does not automatically apply to:

- `README.md`, `AGENTS.md`, design docs, changelogs, and source-controlled prose;
- prompts, logs, telemetry, queues, immutable event ledgers, database-only records, and caches;
- generated previews, reports, and temporary editor state;
- Markdown, HTML, or Djot accepted only as import or emitted only as export.

If a document-like record must remain in a database for product reasons, expose a PDC adapter only when cross-app interchange is required. Do not force the internal database schema to mimic files.

## Choose a body profile

- Use `pdc-djot/1` and `.djot` for ordinary notes, knowledge documents, task-bearing prose, and content whose structure should remain portable across non-browser renderers.
- Use `pdc-html/1` and `.html` when authored HTML structure, layout, or inline CSS is part of the source's value. HTML is not a generated preview and is not second-class.
- Do not invent app-specific canonical Markdown, HTML, JSON, or rich-text dialects. Extend PDC through registered namespaces and readable fallbacks.
- Conversion between Djot and HTML is an explicit migration/export, never an automatic save behavior.

A full PDC Reader supports both profiles. A Writer declares which profile or profiles it creates. A Mutator may edit one profile and keep the other read-only while preserving and safely presenting it.

## New applications

1. Choose Full Reader, Writer, and profile-specific Mutator capabilities.
2. Discover lowercase `.djot` and `.html` files according to the vault rules. Classify unmarked HTML visibly as legacy.
3. Parse the shared constrained envelope, then dispatch to exactly the declared `body` profile.
4. Resolve document links by UUID and managed assets by digest. Never use paths, titles, or app-local database IDs as canonical identity.
5. Use a Djot-aware or plain source editor for Djot. Use a source-preserving HTML editor for HTML. A structural editor must retain unsupported source constructs or downgrade to read-only.
6. Sanitize every preview. HTML uses an isolated view with scripts, event handlers, forms, popups, navigation, and unapproved subresources disabled; sanitization never rewrites stored source.
7. Preserve unknown extension metadata and unsupported constructs. If lossless preservation is impossible, require an explicit conversion to a new target.
8. Run the shared fixtures for both body profiles in the app's native stack.

## Existing applications

Use this order:

1. Inventory transports, metadata, links, assets, semantic extensions, discovery rules, security policy, and editor round-trip behavior.
2. Add a Full Reader for PDC Djot and HTML without changing existing files.
3. Make invalid `.djot`, canonical/legacy `.html`, and unsupported versions visible instead of silently skipping them.
4. Write PDC for newly created documents using the product-appropriate body profile while retaining legacy readers and writers during the compatibility window.
5. Add explicit importers from each legacy format. Legacy HTML that already meets the safe-authored profile should migrate to PDC HTML without body conversion.
6. Recalculate format-derived indexes such as tasks, tags, headings, links, and body hashes from the selected canonical body; never carry projections from another dialect forward as authoritative.
7. Offer per-document or user-authorized batch conversion with backup and a machine-readable loss report.
8. Retire a legacy writer only after shared fixtures and representative real-vault tests pass in every participating application.

Never rewrite a vault merely because an app was upgraded. Opening and closing an untouched document leaves every byte unchanged.

## Editor and preview integration

### Djot

- CodeMirror, Atomic Editor, or another Markdown-specific editor is not automatically a Djot editor. Disable Markdown-only transforms until implemented against the pinned Djot grammar.
- The safe first implementation is plain source editing plus separate Djot preview.

### HTML

- Preserve the raw body source; DOM serialization is not a harmless round trip.
- A source-only editor is conforming. A visual editor is conforming only when comments, unknown elements and attributes, whitespace, and ordering survive or the user explicitly accepts a new converted target.
- Render through sanitization and isolation. Inline CSS may remain useful inside the sandbox, but external CSS, fonts, imports, and CSS URLs require explicit app policy.

### Both profiles

- Preview parity means the same document structure and core semantics, not identical CSS.
- Link clicks, embeds, tasks, query blocks, and asset URLs go through PDC resolvers; custom URI schemes are never handed directly to a browser engine.
- Editing starts from a source-byte snapshot or digest and checks it before replacement.

## Existing product roles

- **Oximemo:** Full Reader; Djot and HTML Writer/Mutator. Keep HTML first-class. Migrate current Markdown to Djot selectively and current HTML to PDC HTML without body conversion when safe.
- **Sawhorse:** Full Reader. Apply PDC only to durable user-authored documents; keep workflow ledgers, runtime state, approvals, caches, and generated evidence outside. Prefer PDC HTML for compatible `shdoc/1` material.
- **Oxibrain:** Full Reader/indexer through connectors only. Never write or migrate the user's vault.
- **Farm:** no current document-plane migration. Re-run the scope gate before introducing a durable shared user document.
- **Lexi:** keep SQLite as product storage. Add PDC import/export only when dictionary entries are deliberately promoted for cross-app interchange.

## Repository integration

Each participating repository keeps a migration or adoption document that states its role, in-scope data, mappings, stages, safety gates, and completion criteria. Its root `AGENTS.md` marks that plan as the highest-priority document-plane initiative unless the user explicitly overrides it.

Use wording equivalent to:

> Portable Document Contract adoption is the repository's highest-priority document-plane initiative. Before changing durable user-document storage, discovery, parsing, editing, rendering, linking, assets, indexing, or migration, load the `portable-document-contract` skill and follow the repository migration plan. Do not convert or rewrite user files without the plan's explicit gates and user authorization.

Pin tests to the fixture corpus revision used by the app. When the standard changes, update parser, writer, migration, and corpus revision coherently.

## Review checklist

- Does every Full Reader find the same valid `.djot` and PDC `.html` files?
- Is unmarked HTML visible as legacy rather than silently skipped?
- Does every invalid or unsupported file produce an actionable diagnostic?
- Are identity, links, and assets independent of filenames and local databases?
- Are metadata-only edits body-byte-preserving for both profiles?
- Are HTML previews isolated without mutating source?
- Are unknown extensions and source constructs preserved or the document made read-only?
- Are writes atomic and guarded against external changes?
- Can legacy import report every lossy, unsafe, ambiguous, or externalized transformation before replacing a source?
- Do Rust, TypeScript, Swift, and other implementations consume the same fixture revision?
