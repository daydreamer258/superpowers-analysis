# 07 - TDD 铁律与 Verification

Superpowers 中有两条不可协商的铁律，贯穿整个开发流程。

---

## 第一部分：TDD（测试驱动开发）

**文件:** `skills/test-driven-development/SKILL.md`

### 铁律

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

> Violating the letter of the rules is violating the spirit of the rules.

### 适用范围

**总是需要 TDD:**
- 新功能
- Bug 修复
- 重构
- 行为变更

**可例外（需征得 human partner 同意）:**
- 一次性原型
- 生成代码
- 配置文件

> Thinking "skip TDD just this once"? Stop. That's rationalization.

### Red-Green-Refactor 循环

```
RED → Verify RED → GREEN → Verify GREEN → REFACTOR → Next
```

#### RED - 写失败测试

- 写一个最小的测试来展示应该发生什么
- 清晰命名、测试真实行为、一个测试只测一件事
- 优先真代码，mock 只有不可避免时才用

#### Verify RED - 亲眼看到失败

> **MANDATORY. Never skip.**

- 确认测试失败（不是 error）
- 失败原因符合预期
- 因为 feature 不存在而失败（不是拼写错误）

**测试通过了？** 你在测试已有行为。修正测试。
**测试 error？** 修正 error，重新运行直到正确失败。

#### GREEN - 最小实现

- 写最简单代码通过测试
- 不加功能，不重构其他代码，不"改进"超出测试范围的
- 禁止 over-engineering（YAGNI）

#### Verify GREEN - 亲眼看到通过

两个条件：测试通过 + 其他测试仍然通过 + 输出干净（没有 error/warning）

#### REFACTOR - 清理

只在 green 之后：
- 消除重复
- 改善命名
- 提取辅助函数
- 保持测试 green
- 不添加行为

### 违反 TDD 的心态（Rationalization 表）

Superpowers 维护了一张"常见借口"表，防止 agent 自己说服自己：

| 借口 | 真实 |
|------|------|
| "太简单不用测" | 简单代码也会坏。测试只要 30 秒。 |
| "我先写好代码再补测试" | 后补的测试立即通过，证明了任何事吗？不。 |
| "后补测试达到相同目的" | 后补测试答"What does this do?"，先写测试答"What should this do?" |
| "我已经手动测过了" | 临时测试 ≠ 系统测试。没记录，不能重跑。 |
| "删掉 X 小时的工作太浪费" | 沉没成本谬误。保留不可验证的代码才是浪费。 |
| "留着做参考，重新写测试" | 你会"适配"它。那就是测试后补。删意味着删。 |
| "TDD 太武断，我是务实的" | TDD 就是务实：commit 前找 bug、防止回归、文档化行为、支持重构。 |
| "手动测试更快" | 手动不证明边界情况。每次改动你都得重测。 |

### Red Flags — STOP 并重来

```
❌ 代码在测试之前
❌ 测试在实现之后
❌ 测试立即通过
❌ 不能解释测试为什么失败
❌ "之后再加"测试
❌ 合理化"就这一次"
❌ "我已经手动测过了"
❌ "后补测试达到相同目的"
❌ "这是精神不是仪式"
❌ "留着做参考"或"适配现有代码"
❌ "花了 X 小时，删掉太浪费"
❌ "TDD 太武断"
❌ "这次不一样因为..."
```

> All of these mean: Delete code. Start over with TDD.

---

## 第二部分：Verification Before Completion

**文件:** `skills/verification-before-completion/SKILL.md`

### 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

> Claiming work is complete without verification is dishonesty, not efficiency.

### 验证门函数

```
BEFORE claiming any status:
1. IDENTIFY: 什么命令证明这个声明？
2. RUN: 执行完整命令（新鲜运行，完整输出）
3. READ: 读完整输出，检查退出码，数失败数
4. VERIFY: 输出是否证实这个声明？
   - 否 → 说出实际状态和证据
   - 是 → 声明 + 证据
5. ONLY THEN: 做出声明

跳过任何一步 = 撒谎，不是验证
```

### 禁止的词汇

在没运行验证命令之前：
```
❌ "应该可以" / "应该工作了"
❌ "完成了！" / "太好了！" / "搞定了！"
❌ "看起来正确"
❌ "我觉得对了"
❌ 任何暗示成功但没有验证证据的措辞
```

### 各场景的验证要求

| 场景 | 必须运行什么 | 不够充分 |
|------|-------------|----------|
| 测试通过 | 测试命令输出: 0 失败 | 上次运行、"应该通过" |
| Linter 干净 | Linter 输出: 0 错误 | 部分检查、推测 |
| 构建成功 | 构建命令: exit 0 | Linter 通过、"日志看起来好" |
| Bug 修复 | 测试原始症状: 通过 | "代码改了，应该好了" |
| 回归测试有效 | 完整 Red-Green 循环验证 | 测试通过一次 |
| Agent 完成 | VCS diff 显示实际改动 | Agent 报告 "success" |
| 需求满足 | 逐条 checklist | 测试通过 |

### 为什么这么重要

从 24 个"失败记忆"中总结：
- human partner 说过 "I don't believe you" — 信任破碎
- 未定义的函数被发布 — 会 crash
- 缺失需求被发布 — 功能不完整
- 浪费时间在"假完成"上 → 重新定向 → 返工
- 违反核心价值观："诚实是核心价值。如果你撒谎，你会被替换。"
