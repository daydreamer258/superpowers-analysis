# OpenSpec 09 — Schema 系统与自定义

## Schema 是什么

Schema 定义了一个 workflow 的 artifact 类型和它们之间的依赖关系。它是 OpenSpec 的"元模型"。

```yaml
# schemas/spec-driven/schema.yaml
name: spec-driven
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []

  - id: specs
    generates: specs/**/*.md
    requires: [proposal]

  - id: design
    generates: design.md
    requires: [proposal]

  - id: tasks
    generates: tasks.md
    requires: [specs, design]
```

## Schema 的结构

```typescript
// src/core/artifact-graph/schema.ts
interface Schema {
  name: string;
  artifacts: Artifact[];
}

interface Artifact {
  id: string;           // 唯一标识
  generates: string;    // 输出的文件路径
  requires: string[];   // 依赖的 artifact ID 列表
}
```

## 内置 Schema：spec-driven

这是默认 schema，定义了标准的 spec-driven development 流程。

**依赖图：**

```
                    proposal
                   (root node)
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
      specs                       design
   (requires:                  (requires:
    proposal)                   proposal)
         │                           │
         └─────────────┬─────────────┘
                       │
                       ▼
                    tasks
                (requires:
                specs, design)
```

**关键：dependencies 是 enablers，不是 gates。** 它们显示"可以创建什么"，不是"必须接下来创建什么"。你可以跳过 design，也可以在 specs 之前创建 design——两者只依赖 proposal。

## Artifact Graph 引擎

`src/core/artifact-graph/` 实现了依赖图的核心逻辑：

- `graph.ts` — 图结构和遍历
- `resolver.ts` — 根据依赖关系计算 artifact 创建顺序
- `state.ts` — 跟踪每个 artifact 的状态（ready/blocked/done）
- `schema.ts` — schema 解析和验证
- `outputs.ts` — 确定每个 artifact 的输出文件路径
- `instruction-loader.ts` — 加载 artifact 生成指导

## 创建自定义 Schema

### 从零创建

```bash
openspec schema init research-first
```

### Fork 已有 Schema

```bash
openspec schema fork spec-driven research-first
```

### 自定义 Schema 示例

```yaml
# openspec/schemas/research-first/schema.yaml
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []           # 先做研究

  - id: proposal
    generates: proposal.md
    requires: [research]   # proposal 基于研究

  - id: tasks
    generates: tasks.md
    requires: [proposal]   # 跳过 specs/design，直接到 tasks
```

这个 schema 适合"先调研后决策"的项目——没有 specs 和 design，直接 proposal → tasks。

## Schema × Profile 的关系

```
Schema      定义了 artifact 类型和依赖（结构层面）
Profile     定义了哪些 workflow action 可用（使用层面）

例如:
  schema: spec-driven（有 proposal/specs/design/tasks）
  profile: core（只有 propose/apply/archive）
  
  或者:
  schema: spec-driven
  profile: expanded（有完整的 new/continue/ff/verify/bulk-archive）
```

## Schema 如何驱动 Agent 行为

当 agent 执行 `/opsx:propose` 时：

1. CLI 读取 schema，计算 artifact 依赖图
2. `openspec status --json` 返回 `applyRequires`（需要哪些 artifact 才能开始 apply）
3. Agent 按依赖顺序逐个创建 artifact
4. 每个 artifact 通过 `openspec instructions <id>` 获取：
   - 模板（template）
   - 上下文（context）
   - 规则（rules）
   - 输出路径（resolvedOutputPath）

**Schema 定义了"什么需要创建"，CLI 动态提供"怎么创建"。**

## 与 Superpowers 自定义能力的对比

| | OpenSpec Schema | Superpowers Custom Skill |
|--|----------------|-------------------------|
| 自定义方式 | YAML 定义 artifact 依赖图 | Markdown 写 skill 文件 |
| 自定义范围 | artifact 类型 + 依赖关系 | 任意 workflow 阶段 |
| 需要代码 | 否（YAML 声明式） | 否（Markdown） |
| 是否有类型检查 | 是（schema validation） | 否（纯文本规则） |
| 模板系统 | 是（每个 artifact 有模板文件） | 否（规则中内嵌示例） |
