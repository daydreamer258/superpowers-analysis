# OpenSpec vs Superpowers — 核心流程对比

## 两句话总结

- **OpenSpec** = 用文件系统结构管理"改了什么、为什么改"的决策追溯系统，agent 是可插拔的
- **Superpowers** = 用 prompt 规则约束 agent 行为的纪律执行系统，agent 是受严格管控的

---

## 一、整体流程对比

### OpenSpec 流程

```
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ PROPOSE   │───►│  APPLY   │───►│ VERIFY   │───►│ ARCHIVE  │
  │           │    │          │    │(optional)│    │          │
  │ proposal  │    │ checklist│    │ completeness│  │ merge   │
  │ design    │    │ → code   │    │ correctness │  │ deltas  │
  │ specs     │    │          │    │ coherence   │  │ → specs │
  │ tasks     │    │          │    │             │  │         │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘

  关键特征：
  - 4 个独立 action，不是排他阶段（"fluid, not rigid"）
  - 用户可以在任何时机调用任何 action
  - 每个 change 是独立的文件夹，可以并行存在多个 change
  - spec 是"活的文档"——archive 后 delta 合并回主 spec
```

### Superpowers 流程

```
  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐    ┌────────────┐
  │ BRAINSTORMING │───►│WRITING PLANS │───►│ SUBAGENT-DRIVEN   │───►│ FINISHING  │
  │              │    │              │    │ DEVELOPMENT       │    │            │
  │ 9步需求澄清  │    │ 2-5分钟/step │    │                  │    │ 4选项      │
  │ →设计→spec  │    │ 零placeholder│    │ Per task:        │    │ merge/PR/  │
  │ →design doc │    │ →plan doc    │    │ Implementer →    │    │ keep/      │
  │ →user approve│   │              │    │ Spec Review →    │    │ discard    │
  └──────────────┘    └──────────────┘    │ Code Review → ✅ │    └────────────┘
                                          └──────────────────┘

  关键特征：
  - 严格的阶段顺序，每个阶段结束后只有一个合法方向
  - brainstorming → writing-plans → subagent-driven-dev → finishing
  - 不可跳过任何阶段（"hard gate"）
  - TDD + Verification Before Completion 两道铁律贯穿全流程
```

### 并排对比

| 维度 | OpenSpec | Superpowers |
|------|----------|-------------|
| 阶段数量 | 4 个 action（可任意组合） | 4 个固定阶段（严格顺序） |
| 强制顺序？ | ❌ "fluid, not rigid" | ✅ "hard gate" |
| 可跳过阶段？ | ✅ 随时可跳过（如跳过 design） | ❌ brainstorming 不能跳过任何项目 |
| 可并行工作？ | ✅ 多个 change 文件夹并行 | ❌ 同一时间只执行一个计划 |
| 谁在做实现？ | 用户自己或用任何 agent | 固定 subagent 模式 |
| 实现时审查？ | 事后 verify（可选） | 每个 task 之间两阶段 review |
| TDD 要求？ | 无内置要求 | 铁律，违反要删代码重来 |

---

## 二、关键哲学差异

### "流体" vs "纪律"

**OpenSpec：Actions, Not Phases**

> "Traditional workflows force you through phases: planning, then implementation, then done. But real work doesn't fit neatly into boxes."

OpenSpec 的设计理念是 **fluid（流体）**——你可以在任何时候做任何 action。用户控制节奏。

```text
OpenSpec 的每种合法路径：

/propose → /apply → /archive            (最快路径)
/propose → /apply → /verify → /archive  (带验证)
/propose → /continue → ... → /apply    (逐步构建)
/explore → /propose → /ff → /apply      (探索后全速)
```

**Superpowers：Stage Gate**

> "Do NOT invoke any implementation skill, write any code until you have presented a design and the user has approved it."

Superpowers 的设计理念是 **纪律**——你不能在没有设计的情况下写代码，不能在跳过 plan 的情况下实现，不能在跳过审查的情况下完成。

### "轻量追溯" vs "重型执行"

**OpenSpec 优化的是"决策可追溯"**

核心产出物：
- `proposal.md` — 为什么做
- `design.md` — 怎么做（技术方案）
- `specs/*.md` — 行为是什么（增量 spec）
- `tasks.md` — 实现 checklist
- 归档后所有文件保留在 `archive/` 中

这些文件的**目标读者是人**——未来的开发者、代码审查者、新人接手者。

**Superpowers 优化的是"执行质量"**

核心产出物：
- `design.md`（brainstorming 输出）
- `plan.md`（writing-plans 输出，内置完整代码和命令）
- `tasks.md`（plan 的一部分，每个 step 2-5 分钟）
- git worktree + git commit 历史

这些文件的**目标读者是 subagent**——plan 里的代码片段是直接给 implementer subagent 执行的。

### 对用户的假设不同

| 假设 | OpenSpec | Superpowers |
|------|----------|-------------|
| 用户在编程吗？ | 用户可以是任何角色 | 用户是"human partner"，是决策者 |
| 用户想管多细？ | 用户控制节奏和调用时机 | 用户审批 spec 和 plan，然后看 agent 跑 |
| 用户信任 agent 吗？ | 信任（verify 是可选的） | 不信任（"Do Not Trust the Report"） |
| 用户需要学什么？ | 4 个 slash command | 整个 workflow skill 体系 |

---

## 三、技术实现差异

### OpenSpec：CLI + 模板驱动的文件系统框架

```text
OpenSpec 的技术构成：
├── openspec CLI (Node.js, ~200+ 源文件)
│   ├── 命令: init, config, change, spec, validate, workspace 等
│   ├── schema 系统：定义 artifact 依赖图
│   ├── 模板引擎：为 25+ AI 工具自动生成 skill/slash command
│   └── 状态管理：artifact graph, change status, context store
├── 文件结构（用户 repo 中的约定）
│   ├── openspec/specs/     ← 行为契约（source of truth）
│   ├── openspec/changes/   ← 进行中的改动
│   └── openspec/changes/archive/ ← 已完成改动（历史）
└── AI 集成方式：生成 tool-specific 配置文件
    ├── 25+ 适配器（claude, cursor, copilot, codex, gemini...）
    └── 为每个平台生成对应的 skill 或 slash command
```

**关键洞察：OpenSpec 的核心是 CLI 工具和管理状态。** Agent 交互只是它支持的一个输出格式。

### Superpowers：纯 Prompt 工程（无代码逻辑）

```text
Superpowers 的技术构成：
├── hooks/session-start     ← bash 脚本，注入 using-superpowers 到会话
├── skills/ (15 个 markdown 文件)
│   ├── 流程编排层: brainstorming, writing-plans, subagent-driven-dev, finishing
│   ├── 纪律约束层: TDD, verification, code review (请求+接收)
│   └── 辅助层: git-worktrees, parallel-agents, systematic-debugging
├── 子 agent prompt 模板
│   ├── implementer-prompt.md
│   ├── spec-reviewer-prompt.md
│   └── code-quality-reviewer-prompt.md
└──  支持的 harness: Claude Code, Cursor, Copilot CLI, Gemini CLI, Codex, OpenCode
```

**关键洞察：Superpowers 没有一行业务逻辑代码。** 它全部靠自然语言 prompt 规则驱动 agent 行为。session-start hook 是唯一的"代码"——它的任务是把 `using-superpowers` 注入到 agent 的系统提示词里。

---

## 四、组合使用：理论上的最佳实践

OpenSpec 和 Superpowers 不是互斥的，它们在不同层面解决问题：

```text
┌──────────────────────────────────────────────────┐
│                  OpenSpec                         │
│  决策追溯层：为什么改、改了什么、谁同意的            │
│  产出：proposal / design / delta specs / tasks     │
│  目标读者：人类开发者（当前和未来的）                │
└──────────────┬───────────────────────────────────┘
               │  spec 作为输入
               ▼
┌──────────────────────────────────────────────────┐
│               Superpowers                         │
│  执行质量层：怎么写、怎么测试、怎么审查              │
│  产出：符合 spec 的高质量代码 + 测试 + review 记录  │
│  目标读者：subagent（执行机器）                     │
└──────────────────────────────────────────────────┘
```

**组合策略：**
- 用 OpenSpec 管理 spec 生命周期和决策追溯（repo 级别）
- 用 Superpowers 作为执行引擎（单次实现环节）
- OpenSpec 的 design/tasks 输出可以作为 Superpowers brainstorming 的输入
- Superpowers 产出的 review 记录可以写回 OpenSpec 的 change 文件夹
