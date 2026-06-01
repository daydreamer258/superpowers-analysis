# OpenSpec 03 — Propose: 需求捕获与 Artifact 生成

## /opsx:propose 的完整流程

这是 OpenSpec 中最复杂的 command。它融合了需求捕获和 artifact 生成。

### 代码层面

`src/core/templates/workflows/propose.ts` 中的 `getOpsxProposeSkillTemplate()` 定义了完整的 agent 指导：

**步骤 1: 获取输入**

如果没有 change name → 用 AskUserQuestion 问用户"想做什么"

**步骤 2: 创建 change 脚手架**

```bash
openspec new change "add-dark-mode"
```

这会在 `openspec/changes/add-dark-mode/` 创建一个空的 change 文件夹（含 `.openspec.yaml`）。

**步骤 3: 获取 artifact 构建顺序**

```bash
openspec status --change "add-dark-mode" --json
```

返回 JSON:
```json
{
  "schemaName": "spec-driven",
  "applyRequires": ["tasks"],
  "artifacts": [
    { "id": "proposal", "status": "ready", "dependencies": [] },
    { "id": "specs", "status": "blocked", "dependencies": ["proposal"] },
    { "id": "design", "status": "blocked", "dependencies": ["proposal"] },
    { "id": "tasks", "status": "blocked", "dependencies": ["specs", "design"] }
  ]
}
```

**步骤 4: 按依赖顺序创建 artifact**

```
Loop until all applyRequires artifacts are done:

  For each artifact that is "ready" (dependencies satisfied):

    1. 获取指令:
       openspec instructions <artifact-id> --change "name" --json

    2. 返回内容:
       - context: 项目背景（约束，不放文件里）
       - rules: artifact 规则（约束，不放文件里）
       - template: 模板结构
       - instruction: schema 指导
       - resolvedOutputPath: 输出路径
       - dependencies: 已完成的 artifact 路径

    3. 读取依赖的 artifact 作为上下文

    4. 用 template 作为结构写文件到 resolvedOutputPath

    5. 重新运行 openspec status 检查进度
```

**步骤 5: 展示最终状态 + 提示 `/opsx:apply`**

## Artifact 文件模板

每个 artifact 有对应的模板文件（`schemas/spec-driven/templates/`）：

### proposal.md 模板

```markdown
## Why
<!-- 动机和问题 -->

## What Changes
<!-- 具体改变什么 -->

## Capabilities
### New Capabilities
- `<name>`: <描述>
### Modified Capabilities
- `<existing-name>`: <什么需求在变>

## Impact
<!-- 影响范围 -->
```

### design.md 模板

```markdown
## Context
<!-- 背景和当前状态 -->

## Goals / Non-Goals
**Goals:** ...
**Non-Goals:** ...

## Decisions
<!-- 关键设计决策和理由 -->

## Risks / Trade-offs
<!-- 已知风险和权衡 -->
```

### tasks.md 模板

```markdown
## 1. <!-- 任务组名 -->
- [ ] 1.1 <!-- 任务描述 -->
- [ ] 1.2 <!-- 任务描述 -->

## 2. <!-- 任务组名 -->
- [ ] 2.1 <!-- 任务描述 -->
```

### spec.md (delta) 模板

```markdown
## ADDED Requirements
### Requirement: <!-- 需求名 -->
#### Scenario: <!-- 场景名 -->
- **WHEN** <!-- 条件 -->
- **THEN** <!-- 预期结果 -->
```

## Propose 的设计理念

1. **一个命令生成所有 artifact** — 不像 Superpowers 那样分 brainstorming → plans 两阶段
2. **模板驱动** — 每个 artifact 有固定结构，agent 填空
3. **CLI 动态提供上下文** — `openspec instructions` 按需返回规则和模板
4. **依赖图保证顺序** — schema 定义的 requires 关系决定了创建顺序
5. **用户在 loop 中** — 不需要像 Superpowers 那样"逐个问题"交互

## 两种速度

| 模式 | 命令 | 控制粒度 |
|------|------|---------|
| 快速 | `/opsx:propose` → `/opsx:ff` | 一键全生成 |
| 精细 | `/opsx:new` → `/opsx:continue` (×N) | 每个 artifact 都过目 |

## 与 Superpowers Brainstorming 的对比

| | OpenSpec Propose | Superpowers Brainstorming |
|--|-----------------|--------------------------|
| 交互模式 | 模板填空（agent 自主） | 9 步对话（逐个问题） |
| 产出 | proposal + design + delta specs + tasks | 单一 design doc |
| 是否需要用户逐节批准 | 否 | 是 |
| 是否有 spec delta | 是（核心机制） | 否 |
| 后续步骤 | /opsx:apply | writing-plans → subagent-driven-dev |
