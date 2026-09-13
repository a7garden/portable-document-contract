# Portable Document Contract

Portable Document Contract (PDC) is a neutral interoperability contract for apps that need to read, edit, index, link, and preserve the same user-authored notes without silently losing data. It defines a frozen Djot profile for ordinary notes and a source-preserving HTML profile for documents whose native structure, layout, or inline CSS matters.

This repository is both the canonical specification and an implicitly invokable Codex skill. Install it once and Codex can apply the contract automatically when a task concerns a durable user-document format.

> Status: `v1.0.0-draft.4` working draft. `pdc-document/1` is the shared contract; `pdc-djot/1` and `pdc-html/1` are canonical body-profile boundaries. Earlier draft tags used the provisional OXI name and were superseded before application adoption.

## Contents

- Normative contract: [`references/PDC-1.0.md`](references/PDC-1.0.md)
- Adoption and migration guide: [`references/adoption.md`](references/adoption.md)
- Parsed-envelope schema: [`references/envelope.schema.json`](references/envelope.schema.json)
- Shared render policy: [`references/render-policy.json`](references/render-policy.json)
- Cross-language fixtures: [`conformance/`](conformance/)

The contract governs durable user-authored documents, not repository documentation or every Markdown/HTML file in an app. Its compatibility target is reliable discovery, parsing, identity, core semantics, safe rendering, and lossless preservation across independently branded apps. Djot and marked PDC HTML are equally canonical; ordinary unmarked HTML remains visible legacy input.

## Install as a Codex skill

### Ask Codex

The simplest cross-platform method is to give Codex this request:

```text
Install the portable-document-contract skill from https://github.com/a7garden/portable-document-contract.
```

Codex can use its bundled skill installer to place the repository in the personal skills directory. The skill becomes available on the next turn; restart the client if an older Codex build does not refresh the skill list.

### Use the bundled installer directly

macOS or Linux:

```sh
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo a7garden/portable-document-contract \
  --path . \
  --name portable-document-contract
```

PowerShell:

```powershell
python "$HOME/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py" --repo a7garden/portable-document-contract --path . --name portable-document-contract
```

The explicit `--name` is required because the skill lives at the repository root. Add `--ref <released-tag-or-commit>` when you intentionally want to pin an audited revision instead of following `main`.

### Direct Git installation

If the bundled installer is unavailable, clone the repository into the personal Codex skills directory:

```sh
git clone --depth 1 \
  https://github.com/a7garden/portable-document-contract.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/portable-document-contract"
```

Do not nest the repository one directory deeper: `SKILL.md` must be directly inside the `portable-document-contract` directory.

The command above follows `main` and can be updated with `git pull`. Add `--branch <released-tag>` when you intentionally want a fixed release.

## Use

Codex may load the skill implicitly when a task changes an app's durable user-document storage, editor, renderer, discovery, links, assets, or migration behavior. Invoke it explicitly when you want to force a standards pass:

```text
$portable-document-contract audit this note editor for cross-app compatibility.
```

For stronger project-wide routing, add this line to the applicable global or repository `AGENTS.md`:

```markdown
For durable user-authored documents, use the `portable-document-contract` skill before changing storage, parsing, editing, rendering, linking, assets, or migrations. Repository documentation is out of scope.
```

User instructions and repository-specific architecture remain authoritative. Installing this skill does not authorize bulk conversion, deletion, publication, or modification of user documents.

## Update or remove

The bundled installer intentionally refuses to overwrite an existing skill directory. To update an installer-managed copy, move the current `portable-document-contract` directory to a backup location, run the install command again, verify the new copy, and then remove the backup when satisfied.

For a Git-managed installation that follows `main`:

```sh
git -C "${CODEX_HOME:-$HOME/.codex}/skills/portable-document-contract" pull --ff-only
```

To disable the skill without deleting it, move its directory outside the personal `skills` directory. Start a new Codex turn after updating, replacing, or disabling a skill.

## Contributing

Issues and pull requests are welcome. Changes to document behavior should update the normative specification, schema or render policy where applicable, and the shared conformance fixtures in the same pull request. Breaking changes require a new major format identifier and a documented migration path.

Before installing any agent skill, review its `SKILL.md` and referenced instructions. This skill contains instructions, specifications, schemas, and fixtures; it does not require credentials or execute application code.

## License

[MIT](LICENSE)
