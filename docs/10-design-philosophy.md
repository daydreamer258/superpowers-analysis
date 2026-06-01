# 10 - 设计哲学总结

## 核心定位

> Superpowers 本质上是一个**用 prompt engineering 约束 AI agent 行为的纪律系统**。
>
> 它不是框架库代码，而是一套层层相扣的提示词规则，通过 session hook 注入到 agent 的"潜意识"里，让 agent 在每个阶段自动触发正确的 workflow skill。
>
> 整个系统没有一行业务逻辑代码，全部靠自然语言指令驱动 agent 的行为。

## 9 个核心设计原则

| 原则 | 体现 |
|------|------|
| **先想后做** | brainstorming 不可跳过，设计必须 approved 才能写代码 |
| **拆到不能再拆** | plan 的每个 step 2-5 分钟，每个 step 是可独立追踪的 checkbox |
| **不信任实现者** | 双阶段 review gate，reviewer 核心指令是"Do Not Trust the Report" |
| **上下文隔离** | 每个 subagent 拿到的是精心构造的 prompt，不继承 session 历史 |
| **失败可降级** | BLOCKED → 更强模型 / 更小 task / 上报 human（四层递进） |
| **成本意识** | 机械任务用便宜模型，架构/审核用最强模型，按需分配 |
| **铁律不可协商** | TDD + Verification，违反就是 rationalization，有专用"借口表"防御 |
| **不留尾巴** | 完成后的 worktree 清理、分支管理、PR 创建 |
| **用户最终控制** | User instructions > Superpowers skills > Default system prompt |

## 为什么这样设计：对 AI Agent 弱点的针对性防御

| Agent 的弱点 | Superpowers 的防御 |
|-------------|-------------------|
| 跳过思考直接写代码 | brainstorming 硬性 Gate |
| 写出模糊的"TODO" | writing-plans 的"No Placeholders"规则 |
| 声称完成但没验证 | Verification Before Completion 铁律 |
| 自己审查自己（放水） | 独立 reviewer subagent + "Do Not Trust the Report" |
| 过度工程化（YAGNI 违反） | Spec Review 检查冗余代码 |
| 找借口跳测试 | TDD Rationalization 表做防御 |
| 多任务混乱 | 上下文隔离 + 顺序执行 |
| 在 main 分支上直接操作 | Git Worktree 隔离 |
| 成本无意识 | 模型分级策略 |

## 关键术语

| 术语 | 含义 | 为什么选这个词 |
|------|------|---------------|
| **"your human partner"** | 用户 | 强调协作关系，非从属关系 |
| **"skill"** | 工作流规则文件 | 类比技能——需要主动调用 |
| **"subagent"** | 子 agent | 强调独立性和上下文隔离 |
| **"controller"** | 编排者（父 session） | 精确形容其职责：调度、审核、决策 |
| **"hard gate"** | 不可绕过的检查点 | 强调强制执行，无例外 |
| **"iron law"** | 铁律 | 强调不可协商，违反 = 失败 |

## 与其他方案的对比

| 方案 | 核心理念 | 控制粒度 |
|------|---------|---------|
| **Superpowers** | 纪律系统：prompt 约束行为 | 每个 step 2-5 分钟 |
| **OpenSpec** | 先写 spec 再执行 | 阶段级 |
| **Cline / Aider** | 交互式 TDD | 对话轮次 |
| **Devin / Factory** | 全自主 | 任务级 |
| **GitHub Copilot** | 内联补全 + chat | 行级 / 对话级 |

## 适用场景判断

| 场景 | Superpowers 适合吗？ |
|------|-------------------|
| 复杂的多文件 feature 开发 | ✅ 完美 — 这是它的主战场 |
| 需要 review 的团队协作 | ✅ 完美 — review gate 无处不在 |
| 快速原型 / 探索 | ⚠️ 偏重 — 但要过 brainstorming 的硬 Gate |
| 单文件小改动 | ⚠️ 可能过度 — 但 Superpowers 认为不存在"太简单" |
| 紧急 hotfix | ⚠️ 流程太长 — 但 TDD 和 Verification 仍然有价值 |
| 配置修改 / 文档 | ❌ 不是它的设计目标 |

## 哲学名言

来自代码中的一些值得玩味的话：

> "Violating the letter of the rules is violating the spirit of the rules."

> "Claiming work is complete without verification is dishonesty, not efficiency."

> "The time is already gone. Your choice now: delete and rewrite with TDD (more hours, high confidence) or keep it and add tests after (30 min, low confidence, likely bugs)."

> "Code review requires technical evaluation, not emotional performance."

> "External feedback = suggestions to evaluate, not orders to follow."

> "If you catch yourself about to write 'Thanks': DELETE IT. State the fix instead."

> "Never silently produce work you're unsure about."
