# OXI Document Standard

OXI Document is a portable document contract for apps that need to read, edit, index, link, and preserve the same user-authored notes without silently losing data. The canonical body syntax is a frozen Djot profile rather than an unspecified Markdown dialect.

This repository is both the canonical specification and an implicitly invokable Codex skill. Install it once and Codex can apply the contract automatically when a task concerns a durable user-document format.

> Status: `v1.0.0-draft.2`. The identifiers `oxi-document/1` and `oxi-djot/1` are already treated as compatibility boundaries; breaking changes require a new major identifier.

## Contents

- Normative contract: [`references/OXI-DOCUMENT-1.0.md`](references/OXI-DOCUMENT-1.0.md)
- Adoption and migration guide: [`references/adoption.md`](references/adoption.md)
- Parsed-envelope schema: [`references/envelope.schema.json`](references/envelope.schema.json)
- Shared render policy: [`references/render-policy.json`](references/render-policy.json)
- Cross-language fixtures: [`conformance/`](conformance/)

The contract governs durable user-authored documents, not repository documentation or every Markdown file in an app. Its compatibility target is reliable discovery, parsing, identity, core semantics, safe rendering, and lossless preservation across OXI apps.

## Install as a Codex skill

### Ask Codex

The simplest cross-platform method is to give Codex this request:

```text
Install the oxi-document-standard skill from https://github.com/project-oxi/oxi-document-standard.
```

Codex can use its bundled skill installer to place the repository in the personal skills directory. The skill becomes available on the next turn; restart the client if an older Codex build does not refresh the skill list.

### Use the bundled installer directly

macOS or Linux:

```sh
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo project-oxi/oxi-document-standard \
  --path . \
  --name oxi-document-standard \
  --ref v1.0.0-draft.2
```

PowerShell:

```powershell
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" --repo project-oxi/oxi-document-standard --path . --name oxi-document-standard --ref v1.0.0-draft.2
```

The explicit `--name` is required because the skill lives at the repository root. Omit `--ref v1.0.0-draft.2` to follow the latest `main` revision instead of pinning the current draft.

### Direct Git installation

If the bundled installer is unavailable, clone the repository into the personal Codex skills directory:

```sh
git clone --depth 1 --branch v1.0.0-draft.2 \
  https://github.com/project-oxi/oxi-document-standard.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/oxi-document-standard"
```

Do not nest the repository one directory deeper: `SKILL.md` must be directly inside the `oxi-document-standard` directory.

The command above pins the draft tag. Remove `--branch v1.0.0-draft.2` if you intentionally want a Git-managed installation that follows `main` and can be updated with `git pull`.

## Use

Codex may load the skill implicitly when a task changes an app's durable user-document storage, editor, renderer, discovery, links, assets, or migration behavior. Invoke it explicitly when you want to force a standards pass:

```text
$oxi-document-standard audit this note editor for cross-app compatibility.
```

For stronger project-wide routing, add this line to the applicable global or repository `AGENTS.md`:

```markdown
For durable user-authored documents, use the `oxi-document-standard` skill before changing storage, parsing, editing, rendering, linking, assets, or migrations. Repository documentation is out of scope.
```

User instructions and repository-specific architecture remain authoritative. Installing this skill does not authorize bulk conversion, deletion, publication, or modification of user documents.

## Update or remove

The bundled installer intentionally refuses to overwrite an existing skill directory. To update an installer-managed copy, move the current `oxi-document-standard` directory to a backup location, run the install command again, verify the new copy, and then remove the backup when satisfied.

For a Git-managed installation that follows `main`:

```sh
git -C "${CODEX_HOME:-$HOME/.codex}/skills/oxi-document-standard" pull --ff-only
```

To disable the skill without deleting it, move its directory outside the personal `skills` directory. Start a new Codex turn after updating, replacing, or disabling a skill.

## Contributing

Issues and pull requests are welcome. Changes to document behavior should update the normative specification, schema or render policy where applicable, and the shared conformance fixtures in the same pull request. Breaking changes require a new major format identifier and a documented migration path.

Before installing any agent skill, review its `SKILL.md` and referenced instructions. This skill contains instructions, specifications, schemas, and fixtures; it does not require credentials or execute application code.

## License

[MIT](LICENSE)
