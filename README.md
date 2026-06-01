# Superpowers & OpenSpec 源码深度分析

> 对 [obra/superpowers](https://github.com/obra/superpowers) v5.1.0 和 [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) 的完整源码分析与对比。

---

## 📂 文档结构

### 🔧 Superpowers — 结构与流程

→ **[完整文档](docs/superpowers/structure-and-workflow.md)**

内容：
- 项目代码仓库结构（纯 prompt 工程，0 业务代码）
- 三层架构（Hook → 编排层 → 纪律约束层）
- 完整 5 阶段工作流（注入 → Brainstorming → Writing Plans → Subagent-Driven Dev → Finishing）
- 4 个 Subagent prompt 模板详解
- TDD + Verification 双铁律机制
- 15 个 Skill 分类与职责

### 📋 OpenSpec — 结构与流程

→ **[完整文档](docs/openspec/structure-and-workflow.md)**

内容：
- 项目代码仓库结构（CLI + 模板引擎 + 25 平台适配器）
- 完整 4 action 工作流（Propose → Apply → Verify → Archive）
- Schema 系统与 Artifact 依赖图
- Delta Spec 机制（ADDED/MODIFIED/REMOVED）
- Core vs Expanded 两种模式
- Workspace 多 repo 协调（beta）

### 🆚 对比分析

- [核心流程对比](docs/compare/01-core-workflow-comparison.md)
- [项目结构与技术实现差异](docs/compare/02-project-structure-diff.md)
- [单点流程能力对比](docs/compare/03-single-point-capability-comparison.md)

### 📖 Superpowers 逐 Skill 深度分析

- [01 - 整体架构概览](docs/01-architecture-overview.md)
- [02 - 启动与注入机制](docs/02-session-hook.md)
- [03 - Brainstorming 流程](docs/03-brainstorming.md)
- [04 - Writing Plans 计划拆解](docs/04-writing-plans.md)
- [05 - Subagent-Driven Development 核心引擎](docs/05-subagent-driven-dev.md)
- [06 - Finishing & Git Worktree](docs/06-finishing-and-worktree.md)
- [07 - TDD 铁律与 Verification](docs/07-tdd-and-verification.md)
- [08 - Code Review 体系](docs/08-code-review.md)
- [09 - 并行 Agent 调度](docs/09-parallel-agents.md)
- [10 - 设计哲学总结](docs/10-design-philosophy.md)

---

## 🎯 一句话总结

| | Superpowers | OpenSpec |
|---|---|---|
| **定位** | 纪律执行系统 | 决策追溯系统 |
| **核心** | prompt engineering 约束 agent 行为 | CLI + 文件系统管理 spec 生命周期 |
| **一句话** | 管理"怎么改得好" | 管理"改了什么" |
