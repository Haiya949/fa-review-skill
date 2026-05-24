# FA 脚本编写标准

这份标准用于之后所有 FA/TP 脚本。没有在这里列出的 API、事件参数、定位方式、兜底路径，不要自行脑补；先读本地 API，再问需求。

## 硬性规则

1. 不写兜底方法。
   - 不用反射找方法。
   - 不用 `Get(object, "...")`、`ReadPosition(object)` 这类多字段兼容读取。
   - 不用 `try/catch { }` 吃掉错误后改走另一套逻辑。
   - 不用 `?? string.Empty`、职业顺序、站位顺序、名字猜职业这类替代逻辑。
   - API 不存在、字段不确定、事件参数不够用时，先问。

2. 修改逻辑时删除原来的旧逻辑。
   - 新逻辑确认后，旧逻辑要从代码里移除。
   - 不把旧逻辑放进 `else`、备用方法、备用分支、失败重试分支。
   - 不写“新逻辑失败就走旧逻辑”的兼容路径。
   - 不保留旧逻辑当兜底、回退、备份、保险。
   - 如果旧逻辑里有仍然需要的常量、坐标或计算方式，先抽出来并明确说明用途；不要整段保留旧流程。

3. 脚本内部不要用 `static`。
   - 状态缓存、字典、列表、计时、标志位、辅助方法都写成实例成员。
   - 不写 `static readonly`、`static Dictionary`、`static List`。
   - `const` 也不要用于脚本状态或机制 ID；机制 ID 用实例 `readonly` 字段。
   - 调用库本身提供的静态 API 例外，例如 `RemoteControl.SetPos(...)`、`DebugPoint.Clear()`。

4. 每一次 TP 前都要先清 debug 点。
   - 每一批连续 TP 前调用一次 `DebugPoint.Clear();`。
   - 如果延迟/任务里执行 TP，必须在实际执行 `RemoteControl.SetPos(...)` 前清。
   - 最后脚本返回 `true` 前也要 `DebugPoint.Clear();`。
   - `RemoteControl.SetPos(...)` 本身会 `DebugPoint.Add(pos)`，所以必须先清旧点。

5. TP、移动、面向、发指令只用指定接口。
   - TP 用 `RemoteControl.SetPos(string role, Vector3 pos)`。
   - 绿玩移动/走路用 `RemoteControl.MoveTo(string role, Vector3 pos)`。
   - 停止移动用 `RemoteControl.MoveStop(string role)`。
   - 面向用 `RemoteControl.SetRot(string role, float rot)`。
   - 发游戏指令用 `RemoteControl.Cmd(string role, string command)`。
   - 不用 `XszRemote.SetPos`、`RemoteControlHelper.SetPos`、反射调用或自写移动兜底。

6. 事件检测只用指定 CondParams。
   - 读条：`EnemyCastSpellCondParams`。
   - Buff 添加：`AddStatusCondParams`。
   - 技能生效：`ReceviceAbilityEffectCondParams`。
   - 无目标技能生效：`ReceviceNoTargetAbilityEffectCondParams`。
   - 头顶标记：`TargetIconEffectTestCondParams`。
   - 连线/接线：`TetherCondParams`。
   - 单位创建：`UnitCreateCondParams`。
   - Buff 参数读取：`IBattleChara.StatusList`。

7. 脚本间共享依赖只用 `ScriptEnv.KV`。
   - 前置脚本负责收集并写入明确 key，例如 `"一麻"`、`"二麻"`、`"三麻"`、`"四麻"`。
   - 后置脚本只读取这些 key，不重新收集同一套条件，不用旧逻辑重算，不做兜底推断。
   - key 名称和 value 类型必须固定，写入和读取两边保持一致。
   - 读取不到 key、类型不对、人数不够时，只等待或返回 `false`；不要猜、不要补、不要回退旧逻辑。
   - 共享数据必须来自同一个 `ScriptEnv`，后置脚本依赖前置脚本已经完成写入。
   - 如果要新增共享 key 或改变 value 类型，先在 md/需求里说明，再改所有读写点。

8. 命名空间和类名必须跟文件位置一致。
   - `namespace` 必须和脚本所在的上级文件夹名一致。
   - 脚本类名必须和 `.cs` 文件名一致。
   - 例如文件是 `伊甸fa/伊甸p2光爆tp.cs`，就写 `namespace 伊甸fa; public class 伊甸p2光爆tp : ITriggerScript`。
   - 例如文件是 `绝欧fa/p1接线.cs`，就写 `namespace 绝欧fa; public class p1接线 : ITriggerScript`。
   - 不使用和文件夹不一致的旧 namespace，例如 `AEFA.TOP.P1`。
   - 改文件夹或改文件名时，同步改 namespace 和 class 名；不要保留旧命名当兼容。

9. 不使用小队列表做玩家/职能映射。
   - 不用 `Svc.Party`、`PartyHelper`、`party.Helper`、`party.helper` 这类队伍列表或队伍辅助封装。
   - 脚本拿到事件对象后直接用玩家名字，例如 `target.Name.TextValue`、`icon.Target.Name.TextValue`、`tether.Left.Name.TextValue`。
   - 需要把玩家名字转成职能位时，只用前端房间维护的名字映射：`RemoteControl.GetRoleByPlayerName(playerName)`。
   - 不为了拿 `EntityId`、职业位或站位顺序去遍历队伍，不建 `_idToRole`、`_roleToId` 这类小队缓存。
   - 名字映射不到职能位时只等待或返回 `false`；不要回退到队伍顺序、职业顺序、名字猜测或 Party 列表。

10. 读取中文文本必须指定 UTF-8。
    - 查看 md、txt、脚本说明、中文日志关键字时，用 `Get-Content -Encoding UTF8` 或等效 UTF-8 读取。
    - 不用 PowerShell 默认编码输出结果来判断中文内容，避免乱码导致关键字写错。

## 软性规则

1. 变量名和方法名**优先用中文**。
   - 脚本是给说中文的人维护的，中文命名比英文更容易理解机制逻辑。
   - 例如 `private readonly uint _锁刃敲打击退ID = 43887;` 而不是 `private readonly uint _cleaveKnockbackId = 43887;`。
   - 例如 `private Vector3 计算分摊点(Vector3 boss位置, Vector3 场中) { ... }` 而不是 `private Vector3 CalculateStackPoint(Vector3 bossPos, Vector3 center) { ... }`。
   - 不强制，但默认优先。遇到无法用中文表达的技术术语（如 `hitbox`、`rotation`、`radius`）保持英文即可。

## AE 时间轴运行逻辑

AE 时间轴是一棵节点树。理解它的核心在于理解两类节点的分工，以及它们如何配合完成编排。

### 两类节点

| 类型 | 节点 | 职责 |
|------|------|------|
| **组合节点** | 序列 / 并行 / 选择 / 循环 | 编排结构——决定子节点"怎么执行" |
| **非组合节点** | 条件节点 / 脚本节点 / 动作节点 / 延迟节点 / 调试节点 / 清除等待 | 执行逻辑——每个节点都是一个检查，返回 true(通过) / false(不通过) |

组合节点 + 条件节点 = **时间轴的骨架**（什么时候做什么、走哪条路）
脚本节点 = **骨架上的肉**（具体检测、计算、TP）

### 组合节点

| 节点 | 语义 | 适用场景 |
|------|------|---------|
| 序列 | 依次执行子节点 | 固定流程的多步机制、单个机制的处理流程 |
| 并行 | 同时执行所有子节点 | 整场战斗的顶层框架、多人独立做不同事 |
| 选择 | 按顺序尝试子节点，第一个成功的返回 | 多分支、随机机制 |
| 循环 | 重复执行子节点 N 次 | 周期性重复机制、循环等待某机制出现 |

根节点通常是并行（整场战斗所有机制序列并行运行），也可以根据需求选择。

### 条件节点（编排守门员）

条件节点在时间轴编排中有一个专门的角色：**判断"当前是不是某个阶段/时机"**，作为组合节点的守卫条件，控制执行流的走向。

典型用途是和组合节点配合做阶段分流：

```
选择（或序列）
  ├─ 条件节点 [技能: 43919]    ← 等 Boss 读"护龙共振"（阶段一标志）
  │   └─ 脚本节点             ← 执行阶段一逻辑
  ├─ 条件节点 [技能: 43934]    ← 等 Boss 读"龙之咆哮"（阶段二标志）
  │   └─ 脚本节点             ← 执行阶段二逻辑
  └─ 条件节点 [技能: 43947]    ← 等 Boss 读"锁刃下挥"（阶段三标志）
      └─ 脚本节点             ← 执行阶段三逻辑
```

条件节点在这里的作用不是做具体计算，而是**决定走哪条分支**。如果条件节点的条件不匹配，选择就继续试下一条，直到找到正确的阶段。这种做法把"该不该做"的判断从脚本代码中抽出来，放到时间轴结构层。

条件节点配置的是内置条件（读条ID/BuffID/连线等），不需要写代码。它只回答"符合/不符合"，不涉及任何动作或计算。在编排树形结构时，条件节点要标注对应的 ID（如技能ID、BuffID），方便人工直接去编辑器中添加这些 ID。

### 脚本节点（实际工作节点）

脚本节点是做**实际工作**的地方：检测机制状态、计算坐标、执行 TP。脚本节点里写 C# 代码，返回 true 表示条件满足/工作完成，返回 false 表示还需要等。

脚本节点的两个模式：
- **仅检查 = false**（默认）→ 进入等待，每帧轮询+事件唤醒。适合"等机制出现"、"等判定完成"
- **仅检查 = true** → 立即检查一次，不等待。适合快照式判断（检测当前Buff状态、检查变量值）

### 脚本节点分工原则

一个脚本节点**只做一件事**。机制检测和位置计算通常放在同一个脚本节点里（因为计算依赖检测结果），但之后如果需要等一段时间再执行，拆成两个脚本节点 + 一个延迟节点。

低优先级写法（一个脚本做完所有事）：

```
脚本节点 {
  // 检测读条、算坐标、延迟、发TP——全部在一个文件里
}
```

高优先级写法（编排分节点）：

```
并行（整场战斗框架）
  ├─ 序列（机制A）
  │   └─ 循环 ×N
  │       └─ 序列
  │           ├─ 条件节点 [技能: 43887] ← 等 Boss 读"锁刃敲打"
  │           ├─ 脚本节点    ← 检测机制 + 算坐标
  │           ├─ 延迟节点    ← 等动画/延迟
  │           └─ 脚本节点    ← 执行 TP
  ├─ 序列（机制B）...
  ├─ 序列（机制C）...
```

### 编排流程

```
Step 1: 人工描述机制详细怎么处理
          → AI 理解机制流程、并发关系、分支条件

Step 2: AI 编排时间轴结构（输出树形结构，类似编辑器视图）
          → 用组合节点 + 条件节点搭骨架（阶段分流、流程串接）
          → 确定每个脚本节点的职责边界
          → 输出可读的树形结构（供人工检查流程）

Step 3: AI 写每个脚本节点对应的 .cs 脚本文件
          → 每个文件只做 Step 2 中分配的单一职责
          → 脚本间通过 ScriptEnv.KV 共享数据

Step 4: 生成"地图"文档，描述编排结构和脚本对应关系
          → 树形结构 + 每个节点的职责说明
          → 每个脚本节点对应哪个 .cs 文件
          → 脚本间通过 ScriptEnv.KV 共享了哪些 key
          → 后续接手的 AI 或开发者读这份地图就能理解全貌

Step 5: 保存树形结构描述 + 地图文档 + 所有 .cs 文件
          → 树形结构和地图让后续接手的人不需要读完整代码就能理解编排
          → 单脚本模式做不到这点——所有逻辑都在代码里，必须读完整代码才能理解
```

### 常见编排模式

| 场景 | 结构 | 说明 |
|------|------|------|
| 整场战斗顶层框架 | 并行 + 多个序列 | 每个机制序列独立并行运行 |
| 单个机制的处理流程 | 序列 | 步骤严格先后（条件节点守卫→脚本节点执行） |
| 周期性重复机制 | 循环 | 把一组节点重复 N 次 |
| 等机制出现再处理 | 条件节点 → 脚本节点 | 条件节点等待特定读条，满足后触发脚本节点 |
| 循环等待机制序列 | 循环 → (条件节点 → 延迟 → 脚本节点) | 高频循环等待机制反复出现 |
| 多人并发处理 | 序列内嵌并行 | 固定流程中需要多人同时动作 |
| 检测→延迟→执行 | 脚本节点 → 延迟节点 → 脚本节点 | 中间延迟可调 |
| 多阶段/多分支 | 选择 + 条件节点 | 条件节点做守门员分流 |

### 跟 FA 脚本的关系

FA 脚本对应脚本节点。在编排好的时间轴里，一个脚本节点的职责是单一的：要么负责检测+计算，要么负责执行 TP。脚本节点通过检查返回值控制流程：false 就等，true 就往下走。组合节点 + 条件节点负责保证脚本节点在正确的时机被调用。

## 已核对 API

来源：本地 `bin/Debug/net10.0/HaiyaBox.dll`、`.nuget-local/aeassist.net/1.2.8/lib/net10.0/AEAssist.dll`、`.nuget-local/aeassist.net/1.2.8/lib/net10.0/Dalamud.dll`。

### RemoteControl

命名空间：`HaiyaBox.Utils`

```csharp
RemoteControl.SetPos(string role, Vector3 pos);
RemoteControl.SetRot(string role, float rot);
RemoteControl.MoveTo(string role, Vector3 pos);
RemoteControl.MoveStop(string role);
RemoteControl.Cmd(string role, string command);
RemoteControl.GetRoleByPlayerName(string playerName); // string?
```

要求：

- 玩家名字转职业位只能用 `RemoteControl.GetRoleByPlayerName(playerName)`；`playerName` 来自事件对象或已经明确传入的玩家名，不从队伍列表遍历获取。
- `GetRoleByPlayerName` 返回 `string?`；拿不到职业时不要猜，不要写空字符串替代，不要走别的职业识别方法。
- TP 前先 `DebugPoint.Clear();`，再 `RemoteControl.SetPos(...)`。
- 面向只调用 `RemoteControl.SetRot(...)`。
- 需要让玩家走到指定点时用 `RemoteControl.MoveTo(...)`，不是瞬移；移动结束或取消循环时调用 `RemoteControl.MoveStop(...)`。
- 发指令用 `RemoteControl.Cmd(...)`。
- 每一次需要接管移动/键盘的 FA 脚本开始前，先发 `RemoteControl.Cmd("", "/xsz-kb on")` 和 `RemoteControl.Cmd("", "/xsz-actionnomove on")`。
- 无敌开关只用 `RemoteControl.Cmd("", "/xsz-invuln on")` 和 `RemoteControl.Cmd("", "/xsz-invuln off")`。
- `MoveTo` 是持续移动目标点，不会自动清 debug 点；不要把它当 TP 用。需要直接改坐标仍然用 `SetPos`。

移动示例：

```csharp
private readonly string[] _roleOrder = { "MT", "ST", "H1", "H2", "D1", "D2", "D3", "D4" };
private readonly float _circleMoveRadius = 0.2f;

private void MoveAllInCircle(Vector3[] centers, int step)
{
    for (var i = 0; i < _roleOrder.Length; i++)
    {
        var rad = step * MathF.PI / 4f + i * MathF.PI / 4f;
        var target = new Vector3(
            centers[i].X + _circleMoveRadius * MathF.Sin(rad),
            centers[i].Y,
            centers[i].Z + _circleMoveRadius * MathF.Cos(rad));

        RemoteControl.MoveTo(_roleOrder[i], target);
    }
}

private void StopAllMovement()
{
    foreach (var role in _roleOrder)
    {
        RemoteControl.MoveStop(role);
    }
}
```

发指令示例：

```csharp
RemoteControl.Cmd("", "/xsz-kb on");
RemoteControl.Cmd("", "/xsz-actionnomove on");
RemoteControl.Cmd("", "/xsz-invuln on");
RemoteControl.Cmd("", "/xsz-invuln off");
```

原地绕圈指令：

```csharp
RemoteControl.Cmd("", "/xsz-walkcircle x y z 半径");
RemoteControl.Cmd("", "/xsz-walkcircle off");
```

说明：

- `/xsz-walkcircle x y z 半径` 表示围绕中心点 `x,y,z` 按指定半径持续走圈。
- `x y z` 是中心点坐标，`半径` 是围绕中心点移动的半径。
- `/xsz-walkcircle off` 用于停止绕圈。
- 例子：`RemoteControl.Cmd("", "/xsz-walkcircle 100 0 100 0.2");`
- 需要简单原地绕圈时优先用这个指令；需要每个角色不同中心点或不同路径时，再用 `RemoteControl.MoveTo(...)` 自己计算目标点。

### DebugPoint

命名空间：`HaiyaBox.Utils`

```csharp
DebugPoint.Clear();
DebugPoint.Add(Vector3 pos);
```

要求：

- 每次 TP 前必须 `DebugPoint.Clear();`。
- 最后返回 `true` 前必须 `DebugPoint.Clear();`。
- 通常不需要手动 `DebugPoint.Add(...)`，因为 `RemoteControl.SetPos(...)` 已经会添加目标点。

### EnemyCastSpellCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

```csharp
public IBattleChara Object;
public Vector3 CastPos;
public float TotalCastTimeInSec;
public float CastRot;
public uint SpellId;
public string SpellName;
public long StartCastTime;
```

用途：

- 只用它检测敌人读条。
- 判断技能用 `cast.SpellId`。
- 读条来源用 `cast.Object`。
- 读条位置用 `cast.CastPos`。
- 读条朝向用 `cast.CastRot`。
- 读条总时长用 `cast.TotalCastTimeInSec`。

### AddStatusCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

```csharp
public uint StatusId;
public string StatusName;
public IBattleChara? Source;
public IBattleChara? Target;
public uint count;
```

用途：

- 只用它检测 Buff 添加事件。
- 判断 Buff 用 `status.StatusId`。
- Buff 目标用 `status.Target`。
- Buff 来源用 `status.Source`。
- 层数/计数字段用 `status.count`。
- 如果要读 Buff 参数、剩余时间、来源 ID，使用 `status.Target.StatusList` 或对应 `IBattleChara.StatusList`，不要从别的路径猜。

### IBattleChara.StatusList

命名空间：

```csharp
using Dalamud.Game.ClientState.Objects.Types;
using Dalamud.Game.ClientState.Statuses;
```

已核对：

```csharp
IBattleChara.StatusList; // StatusList

IStatus.StatusId;
IStatus.Param;
IStatus.RemainingTime;
IStatus.SourceId;
IStatus.SourceObject;
```

用途：

- 读取 Buff 参数必须从 `IBattleChara.StatusList` 遍历。
- `Param` 用于 Buff 参数。
- `RemainingTime` 用于剩余时间。
- `SourceId` / `SourceObject` 用于来源。
- 不写“读不到就从 AddStatusCondParams 里猜”的兜底。

示例：

```csharp
foreach (var buff in actor.StatusList)
{
    if (buff.StatusId != targetStatusId) continue;

    var param = buff.Param;
    var remain = buff.RemainingTime;
    var sourceId = buff.SourceId;
}
```

### ReceviceAbilityEffectCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

```csharp
public IGameObject Source;
public uint ActionId;
public IGameObject? Target;
public string Name;
```

用途：

- 只用它检测技能生效事件。
- 判断技能用 `ability.ActionId`。
- 技能来源用 `ability.Source`。
- 技能目标用 `ability.Target`。

### ReceviceNoTargetAbilityEffectCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

已用字段：

```csharp
public uint ActionId;
```

用途：

- 只用它检测无目标技能生效事件。
- 判断技能用 `noTarget.ActionId`。
- 有目标技能继续用 `ReceviceAbilityEffectCondParams`，不要把两种事件混在同一个兜底读取里猜。

### TargetIconEffectTestCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

```csharp
public uint IconId;
public IGameObject Target;
```

用途：

- 只用它读取头顶标记。
- 判断标记用 `icon.IconId`。
- 标记目标用 `icon.Target`。

### TetherCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

```csharp
public IGameObject Left;
public IGameObject Right;
public uint Args0;
```

用途：

- 只用它检测连线/接线事件。
- 判断连线类型用 `tether.Args0`。
- 连线左端实体 ID 用 `tether.Left.EntityId`。
- 连线右端实体 ID 用 `tether.Right.EntityId`。
- 不用 `tether.LeftId` / `tether.RightId`。
- 需要把连线两端转成职业位时，用 `RemoteControl.GetRoleByPlayerName(tether.Left.Name.TextValue)` 和 `RemoteControl.GetRoleByPlayerName(tether.Right.Name.TextValue)`。
- 不用对象名、DataId、队伍顺序、旧逻辑去猜线两端职业。

示例：

```csharp
private readonly uint _targetTetherId = 89;

private bool IsTargetTetherForRoles(TetherCondParams tether, IReadOnlyCollection<string> roles)
{
    if (tether.Args0 != _targetTetherId) return false;

    var leftRole = RemoteControl.GetRoleByPlayerName(tether.Left.Name.TextValue);
    var rightRole = RemoteControl.GetRoleByPlayerName(tether.Right.Name.TextValue);

    return (!string.IsNullOrWhiteSpace(leftRole) && roles.Contains(leftRole)) ||
           (!string.IsNullOrWhiteSpace(rightRole) && roles.Contains(rightRole));
}
```

### UnitCreateCondParams

命名空间：`AEAssist.CombatRoutine.Trigger`

已用字段：

```csharp
public IGameObject? BattleChara;
```

用途：

- 只用它检测单位创建/AddCombatant 事件。
- 单位对象用 `unitCreate.BattleChara`。
- `unitCreate.BattleChara` 为空时直接等待或返回。
- 判断单位/NPC 类型用 `obj.BaseId`。
- 读取单位位置用 `obj.Position`。
- 读取单位朝向用 `obj.Rotation`。
- 不用对象名、队伍顺序或旧逻辑去猜新创建单位。

### TargetMgr.Instance.Enemys

命名空间：`AEAssist.CombatRoutine.Module.Target`

已用路径：

```csharp
TargetMgr.Instance.Enemys;
```

用途：

- 只在需要读取敌人对象表时使用，例如单位创建事件漏收后重新找场上敌人。
- 对象表来源用 `TargetMgr.Instance.Enemys`。
- 敌人对象从 `TargetMgr.Instance.Enemys.Values.OfType<IBattleChara>()` 里筛。
- 判断敌人类型优先用已确认的 `actor.BaseId`。
- 不用 `TargetMgr.Instance.Enemys` 做玩家/职能映射，不用它替代 `RemoteControl.GetRoleByPlayerName(...)`。
- 读不到目标时等待或返回，不写多字段兼容兜底。

### 敌人死亡/无效判断

对象类型：`IBattleChara`

标准写法：

```csharp
actor.IsDead || actor.CurrentHp == 0
```

要求：

- 判断敌人死亡用 `actor.IsDead`。
- 判断敌人血量归零用 `actor.CurrentHp == 0`。
- 不写 `GetType()`、`GetProperty()`、`GetField()` 反射读取。
- 不写 `CurrentHp`、`CurrentHP`、`Hp`、`HP` 这类多字段兼容兜底。
- 如果本地 API 字段名和这里不一致，先重新核对本地类型，再改 md 和代码。

### LogHelper

命名空间：`AEAssist.Helper`

```csharp
LogHelper.Print(string message);
```

用途：

- 脚本调试日志用 `LogHelper.Print(...)`。
- 日志只用于观察流程，不作为机制判断条件。
- 建议日志前缀带脚本名，例如 `"[火车FA]"`、`"[永暗FA]"`。

## 坐标、角度和危险区

### 坐标和角度约定

游戏场地只按 X/Z 平面计算，Y 通常固定为 0。

游戏对象朝向使用 FFXIV/Dalamud 的 `Rotation` 约定：

```csharp
X偏移 = MathF.Sin(rotation) * distance;
Z偏移 = MathF.Cos(rotation) * distance;
rotation = MathF.Atan2(dx, dz);
```

要求：

- 读条朝向、单位朝向、怪物朝向用 `cast.CastRot`、`cast.Object.Rotation`、`obj.Rotation`。
- 按游戏对象朝向推出点位时，用 `X + Sin`、`Z + Cos`。
- 不要改成数学坐标常见的 `X + Cos`、`Z + Sin`。

场地八方位/时钟点位是脚本自定义角度，不等同于游戏对象 `Rotation`。如果定义 0 度为北，使用：

```csharp
X = center.X + MathF.Sin(rad) * radius;
Z = center.Z - MathF.Cos(rad) * radius;
```

要求：

- 北边是更小的 Z，所以八方位常用 `Z - Cos`。
- 游戏对象 `Rotation` 和自定义北向角不要混用。

### 危险区计算标准

危险区只记录明确的形状、位置、朝向、大小和有效时间。

支持的基础形状：

- 矩形：中心点、朝向、宽度、长度。
- 圆形：中心点、半径。
- 月环：中心点、内半径、外半径。

矩形判断用局部坐标：

```csharp
forward = (MathF.Sin(rotation), MathF.Cos(rotation));
right = (MathF.Cos(rotation), -MathF.Sin(rotation));
localX = Vector2.Dot(point - center, right);
localZ = Vector2.Dot(point - center, forward);
```

矩形命中条件：

```csharp
Abs(localX) <= width / 2 + margin;
Abs(localZ) <= length / 2 + margin;
```

圆形命中条件：

```csharp
Distance2D(point, center) <= radius + margin;
```

月环命中条件：

```csharp
distance >= innerRadius - margin && distance <= outerRadius + margin;
```

要求：

- `margin` 是安全边距，危险区判断必须带边距。
- 危险区需要 `StartAt` / `ExpireAt`，过期后移除。
- 需要提前躲未来危险区时，只纳入明确时间窗口内会生效的危险区。
- 找安全点优先从场中向外采样，候选点必须不在任何危险区内，也不能出场。
- 场地边界按机制需要固定，例如菱形场可用 `Abs(x - center.X) + Abs(z - center.Z)` 判断。
- 没找到安全点时返回等待或保持原逻辑，不写随机点、职业顺序点或旧逻辑兜底。

## 命名规范

脚本命名必须跟目录结构一致，方便按文件定位类，也避免同一阶段脚本分散到旧 namespace 里。

标准格式：

```csharp
namespace 上级文件夹名;

public class 文件名不带扩展名 : ITriggerScript
{
}
```

例子：

```csharp
namespace 伊甸fa;

public class 伊甸p2光爆tp : ITriggerScript
{
}
```

```csharp
namespace 绝欧fa;

public class p1接线 : ITriggerScript
{
}
```

要求：

- 文件在 `伊甸fa` 文件夹下，namespace 就是 `伊甸fa`。
- 文件在 `绝欧fa` 文件夹下，namespace 就是 `绝欧fa`。
- 类名就是文件名去掉 `.cs`。
- 不写 `TriggerScript_...` 这种和文件名不一致的旧类名。
- 不写 `AEFA.TOP.P1` 这种和上级文件夹不一致的旧 namespace。
- 修改命名时直接替换旧命名，不保留旧 class 或旧 namespace 作为兜底。

### 玩家名字和职业位

脚本不使用小队列表作为玩家来源。前端房间已经维护“职能位 -> 玩家”的对应关系，脚本只需要使用事件对象里的玩家名字，必要时再把名字交给 `RemoteControl.GetRoleByPlayerName(...)` 转成职能位。

标准写法：

```csharp
var targetRole = RemoteControl.GetRoleByPlayerName(icon.Target.Name.TextValue);
if (string.IsNullOrWhiteSpace(targetRole))
{
    return false;
}

DebugPoint.Clear();
RemoteControl.SetPos(targetRole, new Vector3(100f, 0f, 95f));
```

要求：

- 需要玩家名时，直接读事件对象的 `Name.TextValue`，例如 `status.Target.Name.TextValue`、`icon.Target.Name.TextValue`、`tether.Left.Name.TextValue`。
- 需要职能位时，只用 `RemoteControl.GetRoleByPlayerName(playerName)` 查询前端房间映射。
- 不使用 `Svc.Party`、`PartyHelper`、`party.Helper`、`party.helper` 或同类小队列表封装。
- 不缓存队伍 `EntityId` 和职能位映射，不写 `_idToRole`、`_roleToId`、`SyncPartyRoleCache()`。
- 读不到名字或映射不到职能位时，只等待或返回 `false`；不要按队伍顺序、职业顺序、对象 ID、名字猜测去补。

## 脚本间共享依赖

标准共享方式是 `scriptEnv.KV`。前置脚本把已经确认的机制结果写入 `scriptEnv.KV`，后置脚本按相同 key 读取，不直接调用前置脚本实例，不复制前置脚本的旧判断逻辑。

### 前置脚本写入

前置脚本必须等数据完整后再写入共享结果。以 P1 麻将点名为例：收齐 8 人，并确认 `"一麻"`、`"二麻"`、`"三麻"`、`"四麻"` 每组 2 人后，再写入：

```csharp
private readonly string[] _roleOrder = ["MT", "ST", "H1", "H2", "D1", "D2", "D3", "D4"];
private readonly Dictionary<string, HashSet<string>> _pendingRoles = new()
{
    { "一麻", [] },
    { "二麻", [] },
    { "三麻", [] },
    { "四麻", [] }
};

private void Commit(ScriptEnv scriptEnv)
{
    foreach (var (key, roles) in _pendingRoles)
    {
        var orderedRoles = roles.OrderBy(role => Array.IndexOf(_roleOrder, role)).ToList();
        scriptEnv.KV[key] = orderedRoles;
    }
}
```

要求：

- 共享 key 用机制语义命名，不用临时名、魔法缩写、可变字符串。
- value 类型固定；职业组统一用 `List<string>`。
- 写入前要确认数据完整；不要把半成品写进去让后置脚本猜。
- 写入后可以返回 `true` 结束前置脚本。

### 后置脚本读取

后置脚本读取前置脚本写入的 key。读取不到、类型不匹配、人数不足时返回 `false` 等下一次触发。

```csharp
private bool TryGetSharedRoleList(ScriptEnv scriptEnv, string key, out List<string> roles)
{
    roles = [];

    if (!scriptEnv.KV.TryGetValue(key, out var value)) return false;
    if (value is not List<string> list) return false;

    roles = list.Where(role => !string.IsNullOrWhiteSpace(role)).ToList();
    return roles.Count > 0;
}
```

使用示例：

```csharp
if (!TryGetSharedRoleList(scriptEnv, "三麻", out var lineRoles) || lineRoles.Count < 2)
{
    return;
}
```

要求：

- 后置脚本只读共享结果，不重新收集同一套点名。
- 不把“重新判断点名”“按职业顺序补位”“旧脚本分支”作为读取失败后的兜底。
- 后置脚本如果依赖固定人数，必须明确检查人数，例如 `lineRoles.Count < 2` 就等待。
- 共享 key 改名时，前置写入和后置读取必须一起改，不能同时保留新旧 key。

### P1 点名到接线的标准依赖

P1 点名脚本写入：

```csharp
scriptEnv.KV["一麻"] = List<string>;
scriptEnv.KV["二麻"] = List<string>;
scriptEnv.KV["三麻"] = List<string>;
scriptEnv.KV["四麻"] = List<string>;
```

P1 接线脚本读取：

```csharp
TryGetSharedRoleList(scriptEnv, "三麻", out var lineRoles); // 拉线
TryGetSharedRoleList(scriptEnv, "一麻", out var towerRoles); // 踩塔
TryGetSharedRoleList(scriptEnv, "二麻", out var crowdA);
TryGetSharedRoleList(scriptEnv, "四麻", out var crowdB);
```

要求：

- `"三麻"` 用作接线组来源。
- `"一麻"` 用作踩塔组来源。
- `"二麻"` 和 `"四麻"` 用作人群组来源。
- 如果 P1 点名脚本没有完成写入，P1 接线脚本等待，不做任何旧逻辑兜底。

## 标准脚本结构

```csharp
using System;
using System.Numerics;
using AEAssist.CombatRoutine.Trigger;
using AEAssist.CombatRoutine.Trigger.Node;
using Dalamud.Game.ClientState.Objects.Types;
using HaiyaBox.Utils;

public class ExampleTpScript : ITriggerScript
{
    // 机制 ID 写成实例字段，不用 static/const。
    private readonly uint _targetCastId = 0;
    private readonly uint _targetStatusId = 0;

    // 脚本完成标记；完成后在 Check 里清 debug 点并返回 true。
    private bool _done;

    public bool Check(ScriptEnv scriptEnv, ITriggerCondParams condParams)
    {
        // Check 会被不同事件反复触发，只按明确的 CondParams 类型分发。
        if (condParams is EnemyCastSpellCondParams cast)
        {
            HandleCast(cast);
        }

        if (condParams is AddStatusCondParams status)
        {
            HandleAddStatus(status);
        }

        if (condParams is ReceviceAbilityEffectCondParams ability)
        {
            HandleAbilityEffect(ability);
        }

        if (condParams is TargetIconEffectTestCondParams icon)
        {
            HandleTargetIcon(icon);
        }

        if (condParams is TetherCondParams tether)
        {
            HandleTether(tether);
        }

        // 结束前必须清旧 debug 点，避免残留上一次 TP 的点位。
        if (_done)
        {
            DebugPoint.Clear();
            return true;
        }

        return false;
    }

    private void HandleCast(EnemyCastSpellCondParams cast)
    {
        // 读条事件用 SpellId 判断机制。
        if (cast.SpellId != _targetCastId) return;

        // 每一批 TP 前先清 debug 点，再调用 SetPos。
        DebugPoint.Clear();
        RemoteControl.SetPos("MT", new Vector3(100f, 0f, 95f));

        // 面向只用 SetRot，不自写面向控制。
        RemoteControl.SetRot("MT", cast.CastRot);
    }

    private void HandleAddStatus(AddStatusCondParams status)
    {
        // Buff 添加事件用 StatusId 判断机制。
        if (status.StatusId != _targetStatusId) return;
        if (status.Target is not IBattleChara target) return;

        // 需要 Buff 参数、剩余时间、来源时，从目标的 StatusList 读取。
        foreach (var buff in target.StatusList)
        {
            if (buff.StatusId != status.StatusId) continue;

            var param = buff.Param;
            var remain = buff.RemainingTime;
            var sourceId = buff.SourceId;
        }
    }

    private void HandleAbilityEffect(ReceviceAbilityEffectCondParams ability)
    {
        // 有目标技能生效事件用 ActionId 判断机制。
        var actionId = ability.ActionId;
        var source = ability.Source;
        var target = ability.Target;
    }

    private void HandleTargetIcon(TargetIconEffectTestCondParams icon)
    {
        // 头顶标记事件用 IconId 判断机制；目标对象从 Target 读取。
        var iconId = icon.IconId;
        var target = icon.Target;
    }

    private void HandleTether(TetherCondParams tether)
    {
        // 连线事件用 Args0 判断类型；实体 ID 从左右端对象的 EntityId 读取。
        var tetherType = tether.Args0;
        var leftId = tether.Left.EntityId;
        var rightId = tether.Right.EntityId;
    }
}
```

## 禁止写法

```csharp
// 禁止：反射找 API
typeof(RemoteControl).GetMethod("GetRoleByPlayerName", ...);

// 禁止：吞异常后改走空字符串/猜测逻辑
try { role = RemoteControl.GetRoleByPlayerName(name) ?? string.Empty; } catch { role = string.Empty; }

// 禁止：自写多字段兜底读取
Get(cond, "Source") ?? Get(cond, "Caster") ?? Get(cond, "Actor");
GetMemberValue(actor, "IsDead");
TryGetUIntProp(actor, out var hp, "CurrentHp", "CurrentHP", "Hp", "HP");

// 禁止：新逻辑失败后回退旧逻辑
if (!TryRunNewLogic())
{
    RunOldLogic();
}

// 禁止：旧逻辑整段保留成备用分支
if (useNewLogic)
{
    RunNewLogic();
}
else
{
    RunOldLogic();
}

// 禁止：TP 前不清旧 debug 点
RemoteControl.SetPos("MT", pos);

// 禁止：最后完成时不清 debug 点
return true;

// 禁止：脚本内部 static 状态
private static readonly Dictionary<uint, string> IdToRole = [];

// 禁止：遍历小队列表做玩家/职能映射
foreach (var member in Svc.Party) { ... }
PartyHelper.GetPartyList();
party.helper.Members;
```

## 新需求处理

之后如果需要这里没列出的能力，例如读地图特效、按 ContentId 查人、延迟调度、时间轴管理、滑步、技能释放等，必须先确认本地 API 类型和字段，再写代码。确认前不要自行添加兜底方法或替代方案。

## MCP 工具

### EXDViewer：FFXIV Excel 数据查询

`EXDViewerCN.exe` 放在各自的仓库根目录。不要把别人的本机绝对路径写进自己的配置。

本机示例路径：

```text
E:\acr备份\fa\m12s\EXDViewerCN.exe
```

启动后内置 MCP 服务器监听：

```text
http://127.0.0.1:3001/mcp
```

当前 Codex MCP 配置已写入：

```text
C:\Users\heizao\.codex\config.toml
```

其他人使用时，配置文件位置是自己用户目录下的 `.codex\config.toml`。

配置项：

```toml
[mcp_servers.exdviewer]
url = "http://127.0.0.1:3001/mcp"
```

使用前必须先在自己的仓库根目录打开 EXDViewer。

通用命令：

```powershell
.\EXDViewerCN.exe
```

本机示例命令：

```powershell
& "E:\acr备份\fa\m12s\EXDViewerCN.exe"
```

典型查询流程：

```text
search_sheets 找表
get_sheet_info 看列数、行数、语言
get_schema_raw 取字段名
query_rows 行级筛选
get_row 精确取行
```

常用工具：

| 工具 | 用途 |
|------|------|
| `search_sheets` | 按名称模糊搜索数据表 |
| `get_sheet_info` | 获取表元数据，例如列数、子行、语言列表 |
| `get_schema_raw` | 获取原始 schema YAML，优先用这个 |
| `get_sheet_schema` | 获取结构化 schema |
| `get_game_version` | 查看数据源和 schema 源版本 |
| `query_rows` | 行级分页查询，支持 filter DSL |
| `get_row` | 按 row_id 精确取行，也可按显示字段简单搜索 |
| `search_cells` | 在指定表中搜索包含关键词的字符串单元格 |
| `resolve_display_field` | 解析行的 GUI 显示文本 |
| `get_sheet_relations` | 获取表的关系映射 |
| `get_referencing_sheets` | 查找哪些表声明了指向目标表的关系 |
| `follow_link` | 沿链接字段解析目标行 |
| `get_icon_url` | 图标 ID 转纹理路径 |
| `decompose_model_id` | 拆分 ModelId |
| `decode_se_string` | 解码 SeString 单元格 |
| `validate_filter` | 校验 filter DSL，构造 `query_rows` 前先用 |
| `validate_schema` | 校验 schema YAML |
| `save_schema` | 保存 schema YAML |

筛选示例：

```text
Level >= 50 AND Name *= "Potion"
```

Schema 注意事项：

- `get_schema_raw` 和 `get_sheet_schema` 会从 GitHub 拉取 `xivdev/EXDSchema` 的 YAML。
- 国内网络可能超时；超时后列名可能显示为 `Unknown0..N`。
- 直连 GitHub 不可用时，优先用镜像：

```text
https://gh.atmoomen.top/raw.githubusercontent.com/xivdev/EXDSchema/refs/heads/latest/{TableName}.yml
```

- 镜像拉到 YAML 后，解析 `- name:` 行可以拿字段名。
- `query_rows` 返回行数据时，每列的 `name` 字段也会带 schema 原始列名，可作为兜底参考。

大表导出注意事项：

- MCP 工具输出有大小限制，大表不要直接靠 MCP 分页刷完整 JSON。
- 大表导出用脚本直连 HTTP MCP：

```text
MCP_URL = "http://127.0.0.1:3001/mcp"
Accept = "application/json, text/event-stream"
```

- 先发 `initialize` 获取 `mcp-session-id`。
- 后续请求带 `mcp-session-id` 复用会话。
- 服务端实际大约限制 200 行/次，`limit` 设置更大也会被截断。
- HTTP 桥接返回 SSE，结果在 `data: {...}` 里。
- HTTP 桥接结果包在 `result.content[0].text` 字符串里，需要再解析一次 JSON。
- 每次 POST 都按新 TCP 连接处理，不要依赖 HTTP pipelining。
- 行数据里除 `f_N` 列外还有 `row_id`、`subrow_id`、`row_index`，按列索引排序时要过滤这些键。
- Link 字段的值是目标表 row_id，不是显示名；要中文名时继续查询目标表做映射。
