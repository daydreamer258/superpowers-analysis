# Superpowers 源码深度分析

> 对 [obra/superpowers](https://github.com/obra/superpowers) v5.1.0 的完整运行逻辑分析。

## 目录

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

## Superpowers 是什么？

Superpowers 是一套**用 prompt engineering 约束 AI agent 行为的纪律系统**。它不是框架库代码，而是一套层层相扣的提示词规则，通过 session hook 注入到 agent 的"潜意识"里，让 agent 在每个阶段自动触发正确的 workflow skill。

核心流程：**Brainstorming → Writing Plans → Subagent-Driven Development → Finishing**
