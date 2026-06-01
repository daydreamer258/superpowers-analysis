# 08 - Code Review 体系

Superpowers 建立了完整的两侧 code review 规程：请求侧和接收侧。

---

## 第一部分：Requesting Code Review

**文件:** `skills/requesting-code-review/SKILL.md`

### 核心原则

> Review early, review often.

### 什么时候必须 review

- subagent-driven development 中每完成一个 task
- 每完成一个 major feature
- merge 到 main 之前

### 什么时候可选 review

- 卡住时（新视角）
- 重构前（基线检查）
- 修完复杂 bug 后

### 发起 review 的标准流程

1. **获取 git SHA:**
   ```bash
   BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
   HEAD_SHA=$(git rev-parse HEAD)
   ```

2. **Dispatch code reviewer subagent**，填充 `code-reviewer.md` 模板：
   - `{DESCRIPTION}` — 简短总结做了什么
   - `{PLAN_OR_REQUIREMENTS}` — 应该是什么
   - `{BASE_SHA}` — 起始 commit
   - `{HEAD_SHA}` — 结束 commit

3. **根据反馈行动:**
   - Critical → 立即修复
   - Important → 继续之前修复
   - Minor → 稍后注意
   - Reviewer 如果错了 → 用技术推理推回去

### Red Flags

```
❌ 因为"简单"跳过 review
❌ 忽略 Critical issues
❌ 在未修复 Important issues 的情况下继续
❌ 和有效的技术反馈争论
```

---

## 第二部分：Receiving Code Review

**文件:** `skills/receiving-code-review/SKILL.md`

### 核心原则

> Code review requires technical evaluation, not emotional performance.
> Verify before implementing. Ask before assuming. Technical correctness over social comfort.

### 接收反馈的标准模式

```
1. READ → 完整阅读反馈，不作出反应
2. UNDERSTAND → 用自己的话重述需求（或提问）
3. VERIFY → 对照代码库实际检查
4. EVALUATE → 对这个代码库技术上是正确的吗？
5. RESPOND → 技术确认 或 合理的 push back
6. IMPLEMENT → 一条一条修复，每一项都单独测试
```

### 禁止的表达

```
❌ "You're absolutely right!"
❌ "Great point!"
❌ "Excellent feedback!"
❌ "Let me implement that now"（验证前不能行动）
❌ "Thanks for [anything]"
❌ 任何感恩表达
```

**正确的做法：** 重述技术需求、问澄清问题、技术推理推回去、或者直接开干（行动 > 话）。

### 处理不清晰的反馈

如果有任何一条不清楚：
→ **STOP**，不要开始实现任何东西
→ 请求澄清

**为什么：** 几条反馈可能是相关的。部分理解 = 错误的实现。

### 对不同来源的处理策略

#### 来自 Human Partner
- 信任 — 理解后实现
- 如果范围不清楚仍然要问
- 不做表演性同意
- 直接跳到行动或技术确认

#### 来自外部 Reviewer
- 保持怀疑，逐条检查：
  1. 对这个代码库技术正确吗？
  2. 会破坏现有功能吗？
  3. 当前实现的原因是什么？
  4. 在所有平台/版本上有效吗？
  5. Reviewer 拥有完整上下文吗？

如果建议看起来不对 → 用技术推理 push back
如果无法轻松验证 → 说出来："I can't verify this without [X]. Should I proceed?"
如果与 human partner 之前的决策冲突 → 停下来和 human partner 讨论

### YAGNI 检查

如果 reviewer 建议"正确实现"某功能：
→ `grep codebase for actual usage`
→ 没人用？→ "This endpoint isn't called. Remove it (YAGNI)?"
→ 有人用？→ 那就正确实现

### Push Back 的时机

**什么时候 push back:**
- 建议会破坏现有功能
- Reviewer 缺少完整上下文
- 违反 YAGNI（没用的 feature）
- 对这个技术栈技术上是错误的
- 有遗留/兼容性原因
- 与 human partner 的架构决策冲突

**怎么 push back:**
- 用技术推理，不用防御态度
- 问具体问题
- 引用工作中的测试/代码
- 如果涉及架构决策，带上 human partner

### 发现自己 push back 错了怎么办

```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ 长道歉
❌ 解释为什么 push back
❌ 过度解释
```

---

## Reviewer Subagent 的两类模板

### Spec Compliance Reviewer

`subagent-driven-development/spec-reviewer-prompt.md`

目的: **验证 implementer 构建了什么被要求的，不多不少**

```
核心指令:
> CRITICAL: Do Not Trust the Report
> The implementer finished suspiciously quickly.
> You MUST verify everything independently.

检查:
1. 遗漏: 要求的东西都实现了吗？
2. 冗余: 多做了没要求的？
3. 误解: 解释错了需求？
```

### Code Quality Reviewer

`subagent-driven-development/code-quality-reviewer-prompt.md`

目的: **验证实现质量——干净、有测试、可维护**

**前提: spec compliance review 已通过**

额外检查：
- 每个文件有清晰的职责吗？
- 单元拆得可以被独立理解和测试吗？
- 实现遵循 plan 的文件结构吗？
- 新文件是否已经太大？是否显著增大了已有文件？

返回格式: Strengths + Issues (Critical/Important/Minor) + Assessment

---

## 与 OpenSpec 审查机制的对比

| 维度 | Superpowers Code Review | OpenSpec |
|------|------------------------|----------|
| 是否强制 | ✅ 每个 task 后强制执行 | ❌ 无内置 review |
| 谁执行 review | 独立 reviewer subagent | 无独立 reviewer |
| review 层级 | 两阶段（spec compliance → code quality） | 无 |
| 接收反馈规则 | 严格行为准则（禁止"Thanks"） | 无 |
| 是否信任实现者 | ❌ "Do Not Trust the Report" | 无此机制 |
| review 失败处理 | 修复 → re-review → 直到通过 | 无 |
