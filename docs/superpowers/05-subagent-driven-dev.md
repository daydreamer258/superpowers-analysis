# 05 - Subagent-Driven Development 核心执行引擎

**文件:** `skills/subagent-driven-development/SKILL.md`

这是 Superpowers 的 **核心精华部分**，也是最复杂的 orchestration。

## 核心原则

> Fresh subagent per task + two-stage review (spec then quality) = high quality, fast iteration

**为什么用 subagent:**
- 每个任务委托给专门的 agent，上下文隔离
- 精确构造 instructions 和 context，确保专注和成功
- 永远不会继承父 session 的上下文或历史
- Controller 也能保持自己上下文干净，专注编排

**持续性执行:**
> Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping.
>
> "Should I continue?" prompts and progress summaries waste their time.

## 每个 Task 的完整生命周期

```
     ┌──────────────────────────────┐
     │ D❶ispatch implementer subagent │
     │ (提供: 完整 task 文本 + 场景上下 │
     │ 文 + 工作目录，不传 session 历史)  │
     └──────────────┬───────────────┘
                    │
           ┌────────▼────────┐
           │ Implementer 状态  │
           └───┬───┬───┬─────┘
               │   │   │
         DONE │ BLOCKED │ NEEDS_CONTEXT
              │   │   │
    ┌─────────▼───┼───┼───> 提供更多上下文 / 更强模型 / 拆分任务
    │ DONE_WITH_CONCERNS
    │ (impl 完成但有疑虑，
    │  controller 先读 concerns)
    │
    └─────────> 进入审核流程
```

### Implementer 的 4 种状态

| 状态 | 含义 | Controller 行为 |
|------|------|-----------------|
| **DONE** | 完成 | 进入 spec compliance review |
| **DONE_WITH_CONCERNS** | 完成但有疑虑 | 先读 concerns，如果是正确性/范围问题→修复；如果只是观察→记录→进入 review |
| **NEEDS_CONTEXT** | 需要更多信息 | 补充上下文 → re-dispatch |
| **BLOCKED** | 无法完成 | 4 级处理: 补上下文 → 更强模型 → 更小 task → 上报 human |

## 两级审核 Gate

### Gate 1: Spec Compliance Review

发一个 spec reviewer subagent，核心指令：

> **CRITICAL: Do Not Trust the Report**
> The implementer finished suspiciously quickly. Their report may be incomplete, inaccurate, or optimistic.

检查三项：

| 检查 | 含义 |
|------|------|
| **遗漏** | 有什么要求没被实现？ |
| **冗余** | 多做了什么没被要求的？YAGNI 违反？ |
| **误解** | 解释错了需求？实现了对的 feature 但是错的方式？ |

**必须逐行读代码验证，不能信任 implementer 的报告。**

```
❌ 不通过 → implementer 修复 → 重新 review → 直到 ✅
✅ 通过 → 进入 Gate 2
```

### Gate 2: Code Quality Review

**前提条件：Spec Compliance Review 必须已经 ✅**

使用 `requesting-code-review/code-reviewer.md` 模板，以 BASE_SHA..HEAD_SHA 的 git diff 为范围。

返回格式：
- **Strengths:** 这波实现哪里做得好
- **Issues:** Critical / Important / Minor 三级
- **Assessment:** 总体评价

```
❌ 不通过 → implementer 修复 → 重新 review → 直到 ✅
✅ 通过 → Mark task complete → 下一个 task
```

## 模型选择策略（成本优化）

| 任务类型 | 推荐模型 | 信号 |
|---------|---------|------|
| 机械实现 | 最快最便宜 | 1-2 文件、完整 spec |
| 集成判断 | 标准模型 | 多文件协调、模式匹配、调试 |
| 架构/设计/审核 | 最强模型 | 需要设计判断或广泛的代码库理解 |

## Implementer Prompt 的结构

`subagent-driven-development/implementer-prompt.md` 定义了模板：

1. **Task Description** — 完整 paste task 文本，不让 subagent 自己读文件
2. **Context** — 场景描述：这个 task 在整个系统中的位置、依赖、架构上下文
3. **Before You Begin** — 有疑问现在问，不要开始后猜
4. **Your Job** — 5 步: 实现→测试→验证→commit→自审→汇报
5. **Code Organization** — 遵循 plan 的文件结构，遇到文件过大标记 DONE_WITH_CONCERNS
6. **When You're in Over Your Head** — 允许 escalation，坏工作 > 没工作
7. **Self-Review** — 4 维度: 完整性/质量/纪律/测试
8. **Report Format** — 标准汇报格式

## Controller（父 session）的职责

```
Controller:
├── 读取完整 plan，提取所有 task 的完整文本
├── 为每个 task 构建子 agent prompt
├── 按顺序 dispatch implementer
├── 处理 implementer 状态:
│   ├── DONE → 进入 review pipeline
│   ├── DONE_WITH_CONCERNS → 读 concerns，判断要不要先修复
│   ├── NEEDS_CONTEXT → 补充信息 → re-dispatch
│   └── BLOCKED → 更强模型 / 更小 task / 报 human
├── 按 Gate 1 → Gate 2 顺序执行 review
├── 处理 review 反馈 (implementer 修复 → 重新 review → 直到 ✅)
└── 所有 task 完成后 → 最终 full code review → finishing
```

## Red Flags（绝对不能做的事）

```
❌ 在 main/master 分支上启动实现
❌ 跳过任何一个 review gate
❌ spec review 没通过就进入 code quality review（顺序！）
❌ 同时 dispatch 多个 implementer（会冲突）
❌ 让 subagent 自己去读 plan 文件（controller 必须提供完整文本）
❌ 忽略 subagent 的问题（先回答问题再让它继续）
❌ 接受"差不多"的 spec compliance（reviewer 发现问题 = 没完成）
❌ 跳过 review 循环（reviewer 发现问题 → implementer 修复 → 必须重新 review）
❌ 用 implementer 的自审代替正式 review（两者都需要）
❌ 在任一 review 还有未解决问题时进入下一个 task
```

## 与 parallel agents 的区别

| | subagent-driven-dev | dispatching-parallel-agents |
|--|---------------------|----------------------------|
| 用途 | 顺序执行 plan 的每个 task | 同时调试多个独立问题 |
| agent 数量 | 每次 1 个 (顺序) | 多个同时 (并行) |
| 审核 | 两个 formal gate | 人工 review 结果 |
| 适用 | 实现了 plan 的有序任务 | 3+ 独立测试文件出错 |
