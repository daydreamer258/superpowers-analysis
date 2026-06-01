# Superpowers & OpenSpec 源码深度分析

> 对 [obra/superpowers](https://github.com/obra/superpowers) v5.1.0 和 [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) 的完整源码分析与对比。

## 内容结构

### 📖 Superpowers 深度分析

- [01 - 整体架构概览](docs/01-architecture-overview.md)
- [02 - 启动与注入机制](docs/02-session-hook.md)
- [03 - Brainstorming 需求澄清流程](docs/03-brainstorming.md)
- [04 - Writing Plans 计划拆解](docs/04-writing-plans.md)
- [05 - Subagent-Driven Development 核心执行引擎](docs/05-subagent-driven-dev.md)
- [06 - Finishing & Git Worktree](docs/06-finishing-and-worktree.md)
- [07 - TDD 铁律与 Verification](docs/07-tdd-and-verification.md)
- [08 - Code Review 体系](docs/08-code-review.md)
- [09 - 并行 Agent 调度](docs/09-parallel-agents.md)
- [10 - 设计哲学总结](docs/10-design-philosophy.md)

### 🆚 OpenSpec vs Superpowers 对比分析

- [对比 01 - 核心流程对比](docs/compare/01-core-workflow-comparison.md)
- [对比 02 - 项目结构与技术实现差异](docs/compare/02-project-structure-diff.md)
- [对比 03 - 单点流程能力对比](docs/compare/03-single-point-capability-comparison.md)

## 一句话总结

- **Superpowers** = 用 prompt engineering 约束 AI agent 行为的纪律执行系统
- **OpenSpec** = 用文件系统结构管理 spec 生命周期的决策追溯系统
- **一句话区别**：OpenSpec 管理"改什么"，Superpowers 管理"怎么改得好"
