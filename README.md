# Superpowers & OpenSpec 源码深度分析

> 对 [obra/superpowers](https://github.com/obra/superpowers) v5.1.0 和 [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) 的完整源码分析与对比。

---

## 📂 文档结构

### 🔧 Superpowers — 完整分析

| 文档 | 内容 |
|------|------|
| [结构与流程总览](docs/superpowers/structure-and-workflow.md) | 项目结构、三层架构、5 阶段完整工作流 |
| [01 - 整体架构概览](docs/superpowers/01-architecture-overview.md) | 三层结构 + 完整状态机 |
| [02 - 启动与注入机制](docs/superpowers/02-session-hook.md) | SessionStart Hook 注入原理 |
| [03 - Brainstorming 需求澄清](docs/superpowers/03-brainstorming.md) | 9 步流程 + 硬 Gate |
| [04 - Writing Plans 计划拆解](docs/superpowers/04-writing-plans.md) | Bite-sized 任务，零 placeholder |
| [05 - Subagent-Driven Development](docs/superpowers/05-subagent-driven-dev.md) | 核心执行引擎 + 双 review gate |
| [06 - Finishing & Git Worktree](docs/superpowers/06-finishing-and-worktree.md) | 完成流程 + 隔离机制 |
| [07 - TDD 铁律与 Verification](docs/superpowers/07-tdd-and-verification.md) | 双铁律 + 借口表防御 |
| [08 - Code Review 体系](docs/superpowers/08-code-review.md) | 请求 + 接收两侧 |
| [09 - 并行 Agent 调度](docs/superpowers/09-parallel-agents.md) | 多问题并行调试 |
| [10 - 设计哲学总结](docs/superpowers/10-design-philosophy.md) | 核心原则 + 对比 + 名言 |

### 📋 OpenSpec — 完整分析

| 文档 | 内容 |
|------|------|
| [结构与流程总览](docs/openspec/structure-and-workflow.md) | 项目结构、4 action 工作流、Schema 系统、Delta 机制 |
| [01 - 整体架构概览](docs/openspec/01-architecture-overview.md) | 双层架构 + 25 适配器 + 状态机 |
| [02 - 初始化与配置注入](docs/openspec/02-init-and-config.md) | init/update 原理 + Profile 系统 |
| [03 - Propose & Artifact 生成](docs/openspec/03-propose-artifacts.md) | 需求捕获 + 4 种 artifact 模板 |
| [04 - Spec 系统与 Delta 机制](docs/openspec/04-spec-and-delta.md) | Delta spec 核心创新 + merge 原理 |
| [05 - Apply 代码执行](docs/openspec/05-apply-implementation.md) | 单 agent 实现流程 |
| [06 - Verify 三维度验证](docs/openspec/06-verify.md) | Completeness/Correctness/Coherence |
| [07 - Archive 归档管理](docs/openspec/07-archive.md) | Delta merge + 历史保留 |
| [08 - 多平台 25+ 适配器](docs/openspec/08-multi-platform-adapters.md) | 适配器架构 + 各平台差异 |
| [09 - Schema 系统与自定义](docs/openspec/09-schema-customization.md) | Artifact 依赖图 + fork 机制 |
| [10 - 设计哲学总结](docs/openspec/10-design-philosophy.md) | 核心原则 + 定位对比 + 名言 |

### 🆚 对比分析

| 文档 | 内容 |
|------|------|
| [核心流程对比](docs/compare/01-core-workflow-comparison.md) | 两套流程并排对比、哲学差异、组合策略 |
| [项目结构与技术实现差异](docs/compare/02-project-structure-diff.md) | 逐目录代码量对比、在用户项目中的布局 |
| [单点流程能力对比](docs/compare/03-single-point-capability-comparison.md) | 需要澄清/计划/实现/验证/归档 逐节点对比 |

---

## 🎯 一句话总结

| | Superpowers | OpenSpec |
|---|---|---|
| **定位** | 纪律执行系统 | 决策追溯系统 |
| **核心** | prompt engineering 约束 agent 行为 | CLI + 文件系统管理 spec 生命周期 |
| **一句话** | 管理"怎么改得好" | 管理"改了什么" |
