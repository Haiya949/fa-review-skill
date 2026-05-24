# fa-review

用于审查 FFXIV FA/TP C# 触发器脚本的 Agent Skill，按仓库内置的 FA 脚本编写标准检查 `RemoteControl`、`DebugPoint`、`CondParams`、`ScriptEnv.KV` 等规则。

Agent Skill for reviewing FFXIV FA/TP C# trigger scripts against the bundled FA writing standard.

## Contents

- `SKILL.md`: skill entrypoint and review workflow.
- `references/FA脚本编写标准.md`: bundled Chinese FA/TP script standard.
- `references/fa-script-standard.md`: compatibility mirror of the same standard.
- `agents/openai.yaml`: optional Codex UI metadata.

## 安装 / Install

### Codex

```powershell
git clone https://github.com/Haiya949/fa-review-skill.git "$env:USERPROFILE\.codex\skills\fa-review"
```

### Claude Code

```powershell
git clone https://github.com/Haiya949/fa-review-skill.git "$env:USERPROFILE\.claude\skills\fa-review"
```
