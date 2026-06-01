# OpenSpec 10 — 设计哲学总结

## 核心理念

OpenSpec 的设计文档明确列出了四个原则：

```
fluid not rigid         — 没有阶段锁，按合理顺序工作
iterative not waterfall — 边学边建，边建边完善
easy not complex        — 轻量设置，最少仪式
brownfield-first        — 适配既有代码库，不只是新项目
```

## 8 个核心设计原则

| 原则 | 体现 |
|------|------|
| **流体，不僵化** | Actions 不是 phases，可以乱序调用 |
| **迭代，不瀑布** | 实现中发现的 design 问题可以回溯修改 artifact |
| **简单，不复杂** | `openspec init` 几秒钟，目录结构直观 |
| **既有代码优先** | Delta spec 让修改成为一等公民 |
| **Spec 是活文档** | Archive 后 delta 合并，spec 随代码迭代增长 |
| **工具无关** | 25+ 适配器，同一个 workflow 跨平台可用 |
| **Schema 可定制** | YAML 声明 artifact 依赖图，支持 fork |
| **决策可追溯** | 每个 change 归档后保留完整上下文（proposal + design + tasks） |

## OpenSpec 的独特创新

### 1. Delta Spec

不做全量 spec，而是记录"什么在变"。这是 OpenSpec 区别于所有其他 SDD 方案的核心。

```
传统 SDD: 为每个 feature 写完整的新 spec
OpenSpec: 写 delta（ADDED/MODIFIED/REMOVED），archive 时自动合并
```

**好处：**
- 清晰（只看变化）
- 无冲突（两个 change 改不同 requirement 不冲突）
- 高效审查（不看不变内容）
- 天然 brownfield

### 2. Schema 系统

不是硬编码的 workflow，而是 YAML 定义的 artifact 依赖图。

```
可以定义:
  spec-driven: proposal → specs+design → tasks
  research-first: research → proposal → tasks
  minimal: proposal → tasks (跳过 specs/design)
```

### 3. CLI 作为 Agent 的"上下文服务器"

OpenSpec 的 CLI 不只是文件管理器——它是 agent 的运行时上下文提供者。

```
agent 不需要记住 workflow 规则
agent 调用 openspec instructions <artifact> --json
CLI 动态返回: 模板 + 规则 + 上下文 + 依赖
```

## 与其他方案的定位对比

| 方案 | 核心理念 | 适合人群 |
|------|---------|---------|
| **OpenSpec** | 流体 SDD + 决策追溯 | 需要 spec 管理和团队协作的开发者 |
| **Superpowers** | 纪律执行 + 高质量代码 | 追求 agent 输出质量的个人/团队 |
| **Spec Kit** | 前置 spec + 绿色项目 | 新项目从零开始 |
| **Cline / Aider** | 交互式 TDD | 偏好对话式开发的个人 |
| **Devin / Factory** | 全自主 | 愿意完全交给 agent 的团队 |

## 适合 OpenSpec 的场景

| 场景 | 适合？ |
|------|--------|
| 既有代码库的增量改动 | ✅ 核心场景（delta 机制） |
| 需要团队协作的大型项目 | ✅ spec 是共享的 source of truth |
| 需要决策追溯的企业项目 | ✅ archive 保留完整历史 |
| 使用多种 AI 工具的团队 | ✅ 25+ 适配器统一 workflow |
| 绿场新项目 | ⚠️ 可以但 Spec Kit 更合适 |
| 追求最高代码质量 | ⚠️ 不强制 TDD 和 review |

## 哲学名言

来自代码和文档中的话：

> "Actions, not phases. Commands are things you can do, not stages you're stuck in."

> "Dependencies are enablers, not gates. They show what's possible, not what's required next."

> "A spec is a behavior contract, not an implementation plan."

> "Most software work isn't building from scratch — it's modifying existing systems. OpenSpec's delta-based approach makes it easy to specify changes to existing behavior, not just describe new systems."

> "The virtuous cycle: specs describe current behavior → changes propose modifications → implementation makes changes real → archive merges deltas → specs now describe new behavior → next change builds on updated specs."
