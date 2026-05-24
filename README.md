# fa-review

Codex skill for reviewing FFXIV FA/TP C# trigger scripts against the bundled FA writing standard.

## Contents

- `SKILL.md`: skill entrypoint and review workflow.
- `references/FA脚本编写标准.md`: bundled Chinese FA/TP script standard.
- `references/fa-script-standard.md`: compatibility mirror of the same standard.
- `agents/openai.yaml`: optional Codex UI metadata.

## Install

Clone this repository directly into your Codex skills directory:

```powershell
git clone <repo-url> "$env:USERPROFILE\.codex\skills\fa-review"
```

Then ask Codex to use `$fa-review` when reviewing FA/TP scripts.

## Notes

The skill expects scripts to follow the bundled standard for `RemoteControl`, `DebugPoint`, `CondParams`, `ScriptEnv.KV`, namespace/class naming, UTF-8 reading, and no fallback/static/party-list logic.
