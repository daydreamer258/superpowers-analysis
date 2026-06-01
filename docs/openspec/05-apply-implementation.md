# OpenSpec 05 — Apply: 代码实现流程

## /opsx:apply 的完整流程

`src/core/templates/workflows/apply-change.ts` 中的 `getApplyChangeSkillTemplate()` 定义了实现流程。

### 步骤 1: 选择 change

```
如果指定了 change name → 直接用
否则:
  - 从对话上下文推断
  - 只有一个活跃 change → 自动选择
  - 多个 change → openspec list --json → 让用户选择
```

### 步骤 2: 检查状态

```bash
openspec status --change "add-dark-mode" --json
```

获取：
- `schemaName` — 使用的 workflow（如 "spec-driven"）
- `planningHome` / `changeRoot` / `actionContext` — 路径和编辑范围
- 哪个 artifact 包含 tasks（spec-driven 中是 "tasks"）

### 步骤 3: 获取 apply 指令

```bash
openspec instructions apply --change "add-dark-mode" --json
```

返回：
- `contextFiles` — 需要读取的文件列表（proposal + specs + design + tasks）
- Progress（total / complete / remaining）
- Task list with status
- 动态指令（基于当前状态）

**处理状态：**
- `state: "blocked"`（缺 artifact） → 提示用 openspec-continue-change
- `state: "all_done"` → 祝贺，建议 archive
- 其他 → 进入实现

**Workspace guard：** 如果在 workspace-planning 模式下且 `allowedEditRoots` 为空 → 停止，不编辑文件。

### 步骤 4: 读取上下文文件

读取 `contextFiles` 中列出的每一个文件。取决于使用的 schema：
- spec-driven: proposal + specs + design + tasks
- 其他 schema: 跟随 CLI 输出

### 步骤 5: 展示进度

```
## Implementing: add-dark-mode (schema: spec-driven)
Progress: 0/7 tasks complete
```

### 步骤 6: 循环实现

```text
for each pending task:
    ├── 展示正在处理的 task
    ├── 做代码改动（最小化、聚焦）
    ├── 标记 tasks.md 中 [ ] → [x]
    └── 继续下一个

暂停条件:
    ├── task 不清楚 → 问用户
    ├── 实现揭示设计问题 → 建议更新 artifact
    ├── 遇到错误/阻塞 → 报告并等待指导
    └── 用户中断
```

### 步骤 7: 完成或暂停

**完成时：**
```
## Implementation Complete
**Change:** add-dark-mode
**Progress:** 7/7 tasks complete ✓
All tasks complete! Ready to archive.
```

**暂停时：**
```
## Implementation Paused
**Progress:** 4/7 tasks complete
### Issue Encountered
<问题描述>
**Options:** 1. ... 2. ... 3. ...
```

## Apply 的关键设计特征

### 1. 单 Agent 执行

没有 subagent。当前 agent 在当前 session 中顺序执行所有 task。

### 2. 没有内置 TDD

OpenSpec 不强制 TDD。测试策略由用户在 spec 中定义，由正常开发流程执行。

### 3. 用户可以随时介入

apply 流程明确设计多个暂停点：
- task 不清楚 → 问
- 设计问题 → 建议更新 artifact
- 错误 → 报告

### 4. Task 粒度灵活

不像 Superpowers 强制 2-5 分钟，OpenSpec 的 task 粒度完全由用户决定。

## "Fluid" 特性在 Apply 中的体现

apply skill 明确说明：

> "Can be invoked anytime: before all artifacts are done (if tasks exist), after partial implementation, interleaved with other actions."
> "Allows artifact updates: if implementation reveals design issues, suggest updating artifacts — not phase-locked, work fluidly."

这意味着：
- 你可以在 proposal 写好但 design 没写之前就开始 apply（只要 tasks 存在）
- 你可以在 apply 过程中停下来更新 design
- 你可以在多个 change 之间切换

## 与 Superpowers Subagent-Driven Dev 的对比

| | OpenSpec Apply | Superpowers Subagent-Driven |
|--|---------------|---------------------------|
| 执行者 | 当前 agent | 每个 task 一个 fresh subagent |
| 上下文隔离 | 否 | 是 |
| TDD | 无内置规则 | 强制 |
| 独立 review | 否 | 是（两阶段 gate） |
| 用户参与 | task 级参与 | 只在 BLOCKED 时 |
| 失败处理 | 暂停 → 等用户 | BLOCKED → 更强模型/更小 task/human |
| Task 粒度 | 用户定义 | 强制 2-5 分钟 |
| 成本 | 低 | 高（每个 task: 3 个 agent） |
