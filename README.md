# Design Draft Codex Skill

Codex skill for restoring Mockplus, CoDesign, and similar browser-based design drafts into project files through Chrome DevTools MCP.

This repository is structured for two uses:

1. Human review on GitHub
2. Direct skill installation from the `skills/design-draft` folder

## What It Does

`design-draft` guides a coding agent through a pixel-focused design restoration workflow:

- Resolve the target design tab from the user's request
- Inspect Mockplus or CoDesign pages through Chrome DevTools MCP
- Confirm the business content area before editing local files
- Restore the design by units instead of guessing a whole page at once
- Extract style values from design-tool panels or metadata where available
- Preserve existing app logic and focus on visual restoration
- Report restored units, evidence sources, assets, estimated fidelity, and remaining gaps

## Repository Layout

```text
.
+-- README.md
+-- LICENSE
+-- commands/
|   +-- DesignDraft.md
+-- skills/
    +-- design-draft/
        +-- SKILL.md
        +-- agents/
        |   +-- openai.yaml
        +-- references/
            +-- asset-rules.md
            +-- codesign.md
            +-- content-region.md
            +-- mockplus.md
            +-- pixel-restore-rules.md
            +-- platform-detection.md
            +-- reporting.md
            +-- restoration-workflow.md
            +-- style-conversion.md
            +-- unit-splitting.md
```

## Install In Codex

Ask Codex to install the skill from this GitHub path:

```text
Install this Codex skill:
https://github.com/Bohaohao/design-draft-codex-skill/tree/main/skills/design-draft
```

After installation, restart Codex so it can discover the new skill.

## Manual Install

If automatic installation is not available, copy the skill folder to your Codex skills directory:

```text
skills/design-draft -> $CODEX_HOME/skills/design-draft
```

If `CODEX_HOME` is not set, Codex commonly uses:

```text
~/.codex/skills/design-draft
```

Restart Codex after copying the folder.

## Optional Slash Command

The `commands/DesignDraft.md` file is included for hosts that support slash-command style entrypoints. It is not required for Codex skill installation.

## Usage

Open the target design draft in Chrome, then ask Codex something like:

```text
Use the design-draft skill to restore the current CoDesign page into src/views/example/index.vue.
```

Or:

```text
/DesignDraft src/views/example/index.vue
```

The skill expects Chrome DevTools MCP to be available. It will ask for confirmation before writing implementation code when the restoration boundary is ambiguous or when the target Vue file is missing.

## Notes For Maintainers

- Keep the installable skill package under `skills/design-draft`.
- Keep long procedural details in `skills/design-draft/references` and keep `SKILL.md` focused on orchestration.
- Do not put repository-level docs inside the skill package unless the agent needs them at runtime.
- Validate changes with a fresh Codex session after modifying `SKILL.md` or the references.

## License

MIT
