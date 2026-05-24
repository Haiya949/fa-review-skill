---
name: fa-review
description: Use when reviewing, creating, or modifying FFXIV FA/TP C# trigger scripts in ACR/HaiyaBox projects, especially tasks involving FA脚本编写标准.md, RemoteControl, DebugPoint, CondParams, ScriptEnv.KV, namespace/class naming, UTF-8 Chinese text, and no fallback/static/party-list logic.
---

# FA Review

## Overview

审查 FA/TP 脚本时，把随 skill 发布的标准当成硬约束，不把缺失 API、缺失字段、旧逻辑或队伍顺序猜测包装成“兼容”。需要精确 API、事件字段、命名例子或 EXDViewer 流程时，读取 `references/FA脚本编写标准.md`。`references/fa-script-standard.md` 是同内容兼容镜像。

读取中文标准、脚本说明、日志关键字时必须指定 UTF-8，例如 PowerShell 用 `Get-Content -Encoding UTF8`，避免乱码导致误判。

## Review Workflow

1. 先确认被审查文件的真实路径、上级目录、文件名、namespace、class 名。
2. 读取相关脚本和 `references/FA脚本编写标准.md` 中对应章节；不要只凭记忆审查 API。
3. 按“硬性红线”逐项查代码，再查机制逻辑、共享数据、坐标/角度、危险区计算。
4. 若发现标准未列出的 API、字段、定位方式或兜底路径，标为问题并建议先核对本地 API 或询问需求。
5. 输出 findings first：按严重度排序，给出文件和行号；没有问题时明确说没发现阻塞项，并说明未验证的残余风险。

## Hard Blocks

| Area | Reject when code uses |
| --- | --- |
| 兜底/猜测 | 反射找 API、多字段兼容读取、`try/catch` 吃错后改走另一套逻辑、`?? string.Empty` 替代、职业/站位/名字猜测 |
| 旧逻辑 | 新逻辑失败后回退旧逻辑、旧逻辑放进 `else`/备用方法/重试分支、整段保留旧流程当保险 |
| 静态状态 | 脚本内部 `static` 状态、`static readonly`、`static Dictionary/List`、机制 ID 用 `const` |
| TP/移动 | `SetPos` 前不清 `DebugPoint`、结束 `return true` 前不清点、使用非标准 TP/移动/面向接口 |
| 事件参数 | 不按指定 `CondParams` 读取事件，或把有目标/无目标技能事件混在兜底读取里猜 |
| 共享依赖 | 脚本间不通过 `ScriptEnv.KV`，后置脚本重新收集、重算、猜补或回退旧逻辑 |
| 命名 | namespace 不等于上级文件夹名，class 名不等于 `.cs` 文件名，保留旧 namespace/class 当兼容 |
| 玩家映射 | 用 `Svc.Party`、`PartyHelper`、队伍列表或缓存表做玩家/职能映射 |
| 编码 | 用 PowerShell 默认编码判断中文 md/txt/日志内容 |

## Required API Checks

优先按标准中的已核对 API 审查：

- TP：`RemoteControl.SetPos(string role, Vector3 pos)`，实际 TP 前必须 `DebugPoint.Clear();`。
- 移动：`RemoteControl.MoveTo(...)`，停止移动：`RemoteControl.MoveStop(...)`。
- 面向：`RemoteControl.SetRot(...)`。
- 指令：`RemoteControl.Cmd(...)`。
- 玩家名转职能位：只用 `RemoteControl.GetRoleByPlayerName(playerName)`，拿不到就等待或返回 `false`。
- Buff 参数、剩余时间、来源：从 `IBattleChara.StatusList` 读取。
- 敌人列表只在需要敌人对象表时用 `TargetMgr.Instance.Enemys`，不能用于玩家/职能映射。

如果代码用了标准未列出的成员或辅助封装，不要补“应该有”；要求核对本地 dll、源码或需求。

## Mechanics Checks

- 坐标只按 X/Z 平面计算，Y 通常固定为 0。
- 游戏对象朝向使用 FFXIV/Dalamud `Rotation` 约定：偏移用 `X + Sin(rotation) * distance`、`Z + Cos(rotation) * distance`。
- 自定义北向八方位与对象 `Rotation` 分开，北边通常是更小的 Z，常用 `Z - Cos`。
- 危险区必须记录形状、位置、朝向、大小、有效时间；矩形、圆、月环判断要带 `margin`。
- 找不到安全点时等待或保持现逻辑，不写随机点、职业顺序点、旧逻辑兜底。

## Output

审查结论用中文写。每个问题包含：

- 严重度：`P0` 会导致错误执行或违反硬性标准；`P1` 高风险/可见行为错误；`P2` 维护性或边界风险。
- 位置：文件路径和行号。
- 依据：引用本 skill 的规则名或 `references/FA脚本编写标准.md` 中的标准。
- 修正方向：给出最小改法，不扩展到未要求的重构。

没有发现问题时仍列出检查覆盖范围，例如“已检查 RemoteControl/DebugPoint/CondParams/ScriptEnv.KV/命名/Party 映射”，并说明没有运行的编译或实机验证。
