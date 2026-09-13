# OXI Document Standard

This repository is both the canonical OXI Document specification and an implicitly invokable Codex skill for applying it to owner-created apps.

Canonical home: [project-oxi/oxi-document-standard](https://github.com/project-oxi/oxi-document-standard). It is installed locally as `~/.codex/skills/oxi-document-standard` so Codex can discover it automatically.

- Normative contract: [`references/OXI-DOCUMENT-1.0.md`](references/OXI-DOCUMENT-1.0.md)
- Adoption and migration guide: [`references/adoption.md`](references/adoption.md)
- Parsed-envelope schema: [`references/envelope.schema.json`](references/envelope.schema.json)
- Shared render policy: [`references/render-policy.json`](references/render-policy.json)
- Cross-language fixtures: [`conformance/`](conformance/)

The contract governs durable user-authored documents, not repository documentation or every Markdown file in an app. Its compatibility target is reliable discovery, parsing, identity, core semantics, safe rendering, and lossless preservation across OXI apps.
