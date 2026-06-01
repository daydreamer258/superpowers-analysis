# Superpowers — 完整结构与流程

> 基于 [obra/superpowers](https://github.com/obra/superpowers) v5.1.0 源码分析

---

## 一、项目结构

Superpowers 是一个**纯 prompt 工程**项目，没有一行业务逻辑代码。全部规则以 Markdown 文件编写，通过 session hook 注入到 AI agent 的系统提示词中。

```
obra/superpowers/
│
├── package.json                    # 元数据，无代码逻辑
├── CLAUDE.md                       # 贡献者指南
│
├── hooks/                          # 唯二的"代码"
│   ├── hooks.json                  # SessionStart hook 配置
│   ├── session-start               # bash 脚本：注入 using-superpowers
│   └── run-hook.cmd                # Windows 支持
│
├── skills/                         # 15 个 Markdown 文件 = 全部"业务逻辑"
│   │
│   ├── using-superpowers/
│   │   └── SKILL.md                # 入口规则：强制 agent 使用 skill
│   │
│   ├── brainstorming/
│   │   ├── SKILL.md                # 需求澄清 9 步流程
│   │   ├── visual-companion.md     # 浏览器 mockup
│   │   ├── spec-document-reviewer-prompt.md
│   │   └── scripts/                # 辅助脚本
│   │
│   ├── writing-plans/
│   │   ├── SKILL.md                # plan 编写规则
│   │   └── plan-document-reviewer-prompt.md
│   │
│   ├── subagent-driven-development/
│   │   ├── SKILL.md                # 核心执行引擎
│   │   ├── implementer-prompt.md           # 实现者 prompt
│   │   ├── spec-reviewer-prompt.md         # spec 审核 prompt
│   │   └── code-quality-reviewer-prompt.md # 代码质量审核 prompt
│   │
│   ├── executing-plans/
│   │   └── SKILL.md                # 备选方案：内联执行
│   │
│   ├── test-driven-development/
│   │   ├── SKILL.md                # TDD 铁律
│   │   └── testing-anti-patterns.md
│   │
│   ├── verification-before-completion/
│   │   └── SKILL.md                # 第二铁律
│   │
│   ├── finishing-a-development-branch/
│   │   └── SKILL.md                # 完成 + 清理
│   │
│   ├── using-git-worktrees/
│   │   └── SKILL.md                # 隔离 workspace
│   │
│   ├── requesting-code-review/
│   │   ├── SKILL.md
│   │   └── code-reviewer.md
│   │
│   ├── receiving-code-review/
│   │   └── SKILL.md                # 接收反馈准则
│   │
│   ├── dispatching-parallel-agents/
│   │   └── SKILL.md                # 并行调试
│   │
│   ├── systematic-debugging/
│   │   ├── SKILL.md
│   │   └── root-cause-tracing.md
│   │
│   └── writing-skills/
│       └── SKILL.md                # 元技能：如何写 skill
│
├── .opencode/plugins/              # OpenCode 平台集成
├── .cursor-plugin/                 # Cursor 平台集成
├── gemini-extension.json           # Gemini 平台集成
│
├── scripts/                        # 辅助脚本
└── tests/                          # 集成测试
```

### 三层架构

```
┌─────────────────────────────────────────────┐
│  Session Start Hook                         │
│  注入 using-superpowers 到每个会话             │
│  效果: Agent 被"附魔"，强制使用 skill          │
└───────────────────┬─────────────────────────┘
                    │
┌───────────────────▼─────────────────────────┐
│  流程编排层 (4 个阶段固定流转)                  │
│  brainstorming → writing-plans →             │
│  subagent-driven-dev → finishing             │
└───────────────────┬─────────────────────────┘
                    │
┌───────────────────▼─────────────────────────┐
│  纪律约束层 (交叉切割)                         │
│  TDD / verification / git-worktrees          │
│  requesting-code-review / receiving-review   │
└─────────────────────────────────────────────┘
```

---

## 二、完整工作流程

### 总体状态机

```
User Request
     │
     ▼
[SessionStart Hook: 注入 using-superpowers]
     │
     ▼
superpowers:brainstorming (9 步)
     │  输出: spec 文档 → git commit
     │  路径: docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md
     ▼
superpowers:writing-plans
     │  输出: plan 文档（包含完整代码 + 命令 + 预期输出）
     │  路径: docs/superpowers/plans/YYYY-MM-DD-<feature>.md
     ▼
superpowers:using-git-worktrees (隔离 workspace)
     │
     ▼
superpowers:subagent-driven-development
     │  每个 Task:
     │  Implementer → Spec Review → Code Quality Review → ✅
     │  贯穿: TDD + Verification Before Completion
     ▼
superpowers:finishing-a-development-branch
     │  验证测试 → 4 选项 (merge/PR/keep/discard) → 清理
     ▼
Done ✅
```

---

### 阶段 1: 启动注入

**文件:** `hooks/session-start`

SessionStart hook 在每次会话启动时（startup/clear/compact）同步执行：

1. 读取 `skills/using-superpowers/SKILL.md` 全文
2. 转义为 JSON 字符串
3. 根据运行平台输出对应 JSON 格式（Claude Code/Cursor/Copilot CLI）
4. 注入到 agent 的系统提示词中

**核心信条：**

> "IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT."
>
> "甚至 1% 的可能性 → 也必须调用。这不是可选的，这不是可协商的。"

---

### 阶段 2: Brainstorming（需求澄清 + 设计）

**文件:** `skills/brainstorming/SKILL.md`

**硬性 Gate：**
> Do NOT invoke any implementation skill, write any code until you have presented a design and the user has approved it.

**9 步检查清单：**

| Step | 操作 |
|------|------|
| 1 | 探索项目上下文（文件/文档/commit） |
| 2 | [可选] 提供视觉预览 |
| 3 | 逐个提问（一次一个问题，多选题优先） |
| 4 | 提出 2-3 种方案 + trade-off + 推荐 |
| 5 | 分章节展示设计，逐节征求确认 |
| 6 | 写设计文档 → `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` → git commit |
| 7 | 自审 spec（扫 TBD/矛盾/范围/歧义） |
| 8 | 用户最终审核 spec 文档 |
| 9 | 转入 writing-plans（禁止跳到任何其他 skill） |

---

### 阶段 3: Writing Plans（计划拆解）

**文件:** `skills/writing-plans/SKILL.md`

**粒度要求：每步 2-5 分钟**

**Task 结构模板：**

```markdown
### Task N: [组件名]

**Files:**
- Create: exact/path/to/file.py
- Modify: exact/path/to/existing.py:123-145
- Test: tests/exact/path/to/test.py

- [ ] Step 1: Write the failing test  ← 完整代码
- [ ] Step 2: Run test to verify it fails  ← 完整命令 + 预期输出
- [ ] Step 3: Write minimal implementation  ← 完整代码
- [ ] Step 4: Run test to verify it passes  ← 完整命令 + 预期输出
- [ ] Step 5: Commit  ← 完整 git 命令
```

**严禁 Placeholder：**
- ❌ "TBD" / "TODO" / "implement later"
- ❌ "Add appropriate error handling"（模糊指令）
- ❌ "Similar to Task N"（必须重复完整内容）
- ❌ 只有描述没有代码块

**自审：**
1. Spec 覆盖检查（每个 spec 要求有对应的 task 吗？）
2. Placeholder 扫描
3. 类型一致性检查（Task 3 的函数名和 Task 7 匹配吗？）

---

### 阶段 4: Subagent-Driven Development（核心执行引擎）

**文件:** `skills/subagent-driven-development/SKILL.md`

**核心原则：** Fresh subagent per task + two-stage review = high quality, fast iteration

**每个 Task 的完整生命周期：**

```
┌──────────────────────────────┐
│ 1. Dispatch implementer       │
│    - 完整 task 文本（paste）   │
│    - 场景上下文（独立 session）│
│    - 不问就猜 = 禁止           │
└──────────────┬───────────────┘
               │
      ┌────────▼────────┐
      │ Implementer 状态  │
      └──┬───┬───┬──────┘
         │   │   │
    DONE │ BLOCKED │ NEEDS_CONTEXT
         │   │   │
         │   ├──→ 更强模型/更小task/报human
         │   └──→ 补充上下文 → re-dispatch
         │
    ┌────▼──────────────────────┐
    │ 2. Spec Compliance Review  │
    │   核心指令：                 │
    │   "CRITICAL: Do Not         │
    │    Trust the Report"        │
    │   检查: 遗漏/冗余/误解       │
    │   ❌ → 修复 → 重新 review    │
    └────────────┬───────────────┘
                 │ ✅
    ┌────────────▼───────────────┐
    │ 3. Code Quality Review      │
    │   基于 git diff              │
    │   返回: Strengths + Issues   │
    │   (Critical/Important/Minor)│
    │   ❌ → 修复 → 重新 review    │
    └────────────┬───────────────┘
                 │ ✅
                 ▼
           Task Complete → 下一个 Task
```

**模型选择策略（成本优化）：**

| 任务类型 | 推荐模型 |
|---------|---------|
| 机械实现（1-2 文件、完整 spec） | 最快最便宜 |
| 集成判断（多文件协调） | 标准模型 |
| 架构/设计/审核 | 最强模型 |

**持续性执行：**
> Do not pause to check in with your human partner between tasks. Execute all tasks without stopping.

---

### 阶段 5: Finishing（完成 + 清理）

**文件:** `skills/finishing-a-development-branch/SKILL.md`

```
Step 1: 验证测试（必须通过）
Step 2: 检测环境（normal repo / worktree / detached HEAD）
Step 3: 确定 base branch
Step 4: 呈现 4 个选项
Step 5: 执行用户选择
Step 6: 清理 worktree
```

**4 个选项：**

| 选项 | 合并 | 推送 | 保留 worktree | 清理分支 |
|-----|------|------|--------------|---------|
| 1. 本地合并 | ✅ | - | - | ✅ |
| 2. Push + PR | - | ✅ | ✅ | - |
| 3. 保留 | - | - | ✅ | - |
| 4. 丢弃 | - | - | - | ✅ (force) |

---

## 三、纪律约束层

### TDD 铁律

> NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
>
> 违反 → 删除代码，重来。

**完整 Red-Green-Refactor 循环：**
```
RED（写失败测试）→ 验证失败 → GREEN（最小实现）→ 验证通过 → REFACTOR（清理）
```

**防御机制：常见借口表**

| 借口 | 真实 |
|------|------|
| "太简单不用测" | 简单代码也会坏 |
| "我先写好代码再补测试" | 后补测试立即通过，证明了什么都？ |
| "已经手动测过了" | 临时测试 ≠ 系统测试 |
| "删掉 X 小时太浪费" | 沉没成本谬误 |

### Verification Before Completion 铁律

> NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
>
> 声称完成却不验证 = 撒谎，不是效率。

```
BEFORE claiming any status:
1. IDENTIFY → 什么命令证明？
2. RUN → 执行完整命令
3. READ → 完整输出 + 退出码
4. VERIFY → 输出真的确认了吗？
5. ONLY THEN → 做出声明
```

### Code Review 体系

**请求侧：** `requesting-code-review`
- 每 task 完成后 / major feature 后 / merge 前必须 review
- 发起: git SHA → dispatch reviewer subagent → 根据反馈行动

**接收侧：** `receiving-code-review`
- 禁止表演性同意（"You're absolutely right!"）
- 验证后实现 > 盲目接受
- 外部反馈 = 需要评估的建议，不是必须执行的命令

### Git Worktree 隔离

- 已有隔离 → 跳过
- 原生工具 → 优先
- 手动 git worktree → 后备
- 必须验证目录被 `.gitignore` 忽略

---

## 四、15 个 Skill 分类

| 层 | Skill | 角色 |
|----|-------|------|
| **Hook** | `using-superpowers` | 启动注入，强制使用 skill |
| **编排** | `brainstorming` | 需求澄清 → spec |
| **编排** | `writing-plans` | spec → bite-sized tasks |
| **编排** | `subagent-driven-development` | 核心执行引擎 |
| **编排** | `executing-plans` | 备选：内联执行 |
| **编排** | `dispatching-parallel-agents` | 多问题并行调试 |
| **编排** | `finishing-a-development-branch` | 完成 + 清理 |
| **约束** | `using-git-worktrees` | 隔离 workspace |
| **约束** | `test-driven-development` | TDD 铁律 |
| **约束** | `verification-before-completion` | 验证铁律 |
| **约束** | `requesting-code-review` | 发起 review |
| **约束** | `receiving-code-review` | 接收 review 反馈 |
| **约束** | `systematic-debugging` | 系统性调试 |
| **元** | `writing-skills` | 编写 skill 最佳实践 |
| **辅助** | `brainstorming/visual-companion` | 浏览器 mockup |

---

## 五、设计哲学

| 原则 | 体现 |
|------|------|
| **先想后做** | brainstorming 硬 Gate |
| **拆到不能再拆** | 每 step 2-5 分钟 |
| **不信任实现者** | 双 review gate, "Do Not Trust the Report" |
| **上下文隔离** | subagent 不继承父 session 历史 |
| **失败可降级** | BLOCKED → 更强模型/更小 task/human |
| **成本意识** | 模型按 task 复杂度分级 |
| **铁律不可协商** | TDD + Verification，有借口表防御 |
| **不留尾巴** | worktree 清理 + 分支管理 |

> "Claims without verification is dishonesty, not efficiency."
>
> "Code review requires technical evaluation, not emotional performance."
>
> "External feedback = suggestions to evaluate, not orders to follow."
