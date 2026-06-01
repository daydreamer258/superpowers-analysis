# OpenSpec — 完整结构与流程

> 基于 [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) 源码分析

---

## 一、项目结构

OpenSpec 是一个 **CLI 驱动的 spec-driven development 框架**，核心由 TypeScript 编写的 CLI 工具 + AI 模板引擎 + 多平台适配器组成。

### 代码仓库结构

```
Fission-AI/OpenSpec/
│
├── bin/openspec.js                    # CLI 入口
├── package.json                       # npm: @fission-ai/openspec
├── tsconfig.json                      # TypeScript 配置
│
├── src/
│   ├── cli/index.ts                   # CLI 命令路由
│   │
│   ├── commands/                      # 用户可见命令
│   │   ├── change.ts                  # openspec change [list|show|validate]
│   │   ├── config.ts                  # openspec config [profile|...]
│   │   ├── spec.ts                    # openspec spec [list|show|validate]
│   │   ├── validate.ts                # openspec validate
│   │   ├── workspace.ts               # openspec workspace [setup|open|link...]
│   │   ├── schema.ts                  # openspec schema [init|fork|...]
│   │   ├── show.ts                    # openspec show
│   │   ├── completion.ts              # shell 补全
│   │   ├── context-store.ts           # 跨会话上下文
│   │   ├── initiative.ts              # initiative 管理
│   │   ├── feedback.ts                # 反馈收集
│   │   └── workflow/                  # 内部 workflow 辅助
│   │       ├── instructions.ts        # artifact 生成指令
│   │       ├── new-change.ts          # 创建 change 脚手架
│   │       ├── schemas.ts             # schema 管理
│   │       ├── templates.ts           # 模板生成
│   │       ├── status.ts              # change 状态查询
│   │       └── shared.ts              # 共享逻辑
│   │
│   ├── core/                          # ~100 个核心模块
│   │   │
│   │   ├── artifact-graph/            # artifact 依赖图引擎
│   │   │   ├── graph.ts               # 核心图结构
│   │   │   ├── resolver.ts            # artifact 创建顺序解析
│   │   │   ├── schema.ts              # schema 定义
│   │   │   ├── state.ts               # artifact 状态追踪
│   │   │   ├── outputs.ts             # artifact 输出路径
│   │   │   └── instruction-loader.ts  # 指令加载
│   │   │
│   │   ├── command-generation/        # 为 AI 工具生成 skill/command
│   │   │   ├── adapters/              # 25+ 平台适配器
│   │   │   │   ├── claude.ts          # Claude Code
│   │   │   │   ├── cursor.ts          # Cursor
│   │   │   │   ├── github-copilot.ts  # GitHub Copilot
│   │   │   │   ├── codex.ts           # OpenAI Codex
│   │   │   │   ├── gemini.ts          # Gemini CLI
│   │   │   │   ├── windsurf.ts        # Windsurf
│   │   │   │   ├── roocode.ts         # RooCode
│   │   │   │   ├── cline.ts           # Cline
│   │   │   │   ├── factory.ts         # Factory Droid
│   │   │   │   ├── opencode.ts        # OpenCode
│   │   │   │   ├── codebuddy.ts       # CodeBuddy
│   │   │   │   ├── qwen.ts            # Qwen
│   │   │   │   ├── pi.ts              # PI
│   │   │   │   ├── auggie.ts          # Auggie
│   │   │   │   ├── bob.ts             # Bob
│   │   │   │   ├── kiro.ts            # Kiro
│   │   │   │   ├── kilocode.ts        # KiloCode
│   │   │   │   ├── junie.ts           # Junie
│   │   │   │   ├── qoder.ts           # Qoder
│   │   │   │   ├── costrict.ts        # Costrict
│   │   │   │   ├── crush.ts           # Crush
│   │   │   │   ├── continue.ts        # Continue
│   │   │   │   ├── iflow.ts           # iFlow
│   │   │   │   ├── lingma.ts          # Lingma
│   │   │   │   ├── antigravity.ts     # Antigravity
│   │   │   │   └── amazon-q.ts        # Amazon Q
│   │   │   ├── generator.ts           # 生成引擎
│   │   │   └── registry.ts            # 适配器注册
│   │   │
│   │   ├── templates/                 # 模板和 skill 生成
│   │   │   ├── skill-templates.ts     # skill 模板定义
│   │   │   ├── types.ts               # 类型定义
│   │   │   └── workflows/             # 每个 workflow action 的模板
│   │   │       ├── propose.ts         # /opsx:propose
│   │   │       ├── apply-change.ts    # /opsx:apply
│   │   │       ├── verify-change.ts   # /opsx:verify
│   │   │       ├── archive-change.ts  # /opsx:archive
│   │   │       ├── new-change.ts      # /opsx:new
│   │   │       ├── continue-change.ts # /opsx:continue
│   │   │       ├── ff-change.ts       # /opsx:ff
│   │   │       ├── explore.ts         # /opsx:explore
│   │   │       ├── sync-specs.ts      # /opsx:sync
│   │   │       ├── bulk-archive-change.ts # /opsx:bulk-archive
│   │   │       ├── onboard.ts         # /opsx:onboard
│   │   │       └── feedback.ts        # /opsx:feedback
│   │   │
│   │   ├── config.ts                  # 配置管理
│   │   ├── init.ts                    # openspec init
│   │   ├── update.ts                  # openspec update
│   │   ├── archive.ts                 # 归档逻辑（delta merge）
│   │   ├── list.ts                    # 列出 changes
│   │   ├── profiles.ts               # workflow profile 选择
│   │   ├── config-schema.ts           # 配置 JSON Schema
│   │   ├── project-config.ts          # 项目级配置
│   │   ├── global-config.ts           # 全局配置
│   │   │
│   │   ├── context-store/             # 跨会话上下文存储
│   │   │   ├── foundation.ts
│   │   │   ├── binding.ts
│   │   │   └── registry.ts
│   │   │
│   │   ├── collections/               # initiatives 集合
│   │   │   └── initiatives/
│   │   │       ├── collection.ts
│   │   │       ├── operations.ts
│   │   │       ├── resolution.ts
│   │   │       └── templates.ts
│   │   │
│   │   ├── workspace/                 # workspace 管理（beta）
│   │   │   ├── foundation.ts
│   │   │   ├── registration.ts
│   │   │   ├── openers.ts
│   │   │   └── skills.ts
│   │   │
│   │   ├── validation/                # 验证引擎
│   │   │   ├── validator.ts
│   │   │   └── constants.ts
│   │   │
│   │   ├── parsers/                   # markdown/spec 解析
│   │   │   ├── change-parser.ts
│   │   │   ├── markdown-parser.ts
│   │   │   ├── requirement-blocks.ts
│   │   │   └── spec-structure.ts
│   │   │
│   │   ├── converters/                # 格式转换
│   │   │   └── json-converter.ts
│   │   │
│   │   ├── completions/               # shell 补全
│   │   │   ├── generators/            # bash/zsh/fish/powershell
│   │   │   └── installers/
│   │   │
│   │   ├── telemetry/                 # 遥测
│   │   └── shared/                    # 共享工具
│   │
│   └── utils/                         # 通用工具
│
├── schemas/                           # Schema + 模板定义
│   ├── spec-driven/
│   │   ├── schema.yaml                # artifact 依赖图
│   │   └── templates/                 # artifact 文件模板
│   │       ├── proposal.md
│   │       ├── design.md
│   │       ├── spec.md
│   │       └── tasks.md
│   └── workspace-planning/
│       └── ...
│
├── docs/                              # 文档
│   ├── concepts.md
│   ├── workflows.md
│   ├── commands.md
│   ├── customization.md
│   ├── getting-started.md
│   └── ...
│
├── test/                              # ~80+ 测试文件
│
└── openspec/                          # dogfood：用 OpenSpec 管理自身
    ├── specs/                         # 40+ spec 文件
    └── changes/                       # 15+ 已归档 change
```

### OpenSpec 在你的项目中的结构

运行 `openspec init` 后：

```
your-project/
├── openspec/
│   ├── config.yaml              # OpenSpec 配置
│   ├── specs/                   # 系统行为 spec（source of truth）
│   │   ├── auth/spec.md
│   │   ├── payments/spec.md
│   │   └── ui/spec.md
│   └── changes/                 # 进行中的改动
│       ├── add-dark-mode/
│       │   ├── proposal.md      # 为什么 + 范围
│       │   ├── design.md        # 技术方案
│       │   ├── tasks.md         # 实现 checklist
│       │   ├── .openspec.yaml   # change 元数据
│       │   └── specs/           # delta specs
│       │       └── ui/spec.md   # 对主 ui/spec.md 的增量
│       └── archive/             # 已完成的 change（保留历史）
│           └── 2025-01-24-add-2fa/
├── AGENTS.md                    # 生成的 agent 指导
└── .claude/                     # 如果选了 Claude Code
```

---

## 二、完整工作流程

### 总体流程

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ PROPOSE   │───►│  APPLY   │───►│ VERIFY   │───►│ ARCHIVE  │
│           │    │          │    │(optional)│    │          │
│ proposal  │    │ checklist│    │ completeness│  │ delta →  │
│ design    │    │ → code   │    │ correctness │  │ main     │
│ delta     │    │          │    │ coherence   │    │ specs    │
│ specs     │    │          │    │             │    │ → archive│
│ tasks     │    │          │    │             │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘

  "actions, not phases" — 可以在任何时候调用任何 action
  多个 change 文件夹可并行存在
```

### 核心理念

> **Fluid, not rigid** — 没有阶段锁，工件可以在任何有意义的顺序创建
> **Iterative, not waterfall** — 需求会变，理解会加深，OpenSpec 拥抱这个现实
> **Easy, not complex** — 几秒初始化，立即开工
> **Brownfield-first** — delta 方式适合修改既有系统，不只是描述新系统

---

### Action 1: Propose（创建 change + 生成 artifacts）

**CLI 命令链：**

```bash
# 1. 创建 change 脚手架
openspec new change "add-dark-mode"

# 2. 获取 artifact 构建顺序
openspec status --change "add-dark-mode" --json

# 3. 按依赖顺序创建每个 artifact
openspec instructions proposal --change "add-dark-mode" --json
# → 获取 context + rules + template + instruction
# → 写入 proposal.md

openspec instructions specs --change "add-dark-mode" --json
# → 读取 proposal.md 作为上下文
# → 写入 delta spec 到 specs/ 目录

openspec instructions design --change "add-dark-mode" --json
# → 可以 parallel 于 specs 创建（都只依赖 proposal）
# → 写入 design.md

openspec instructions tasks --change "add-dark-mode" --json
# → 需要 specs + design 都完成后才能创建
# → 写入 tasks.md
```

**Artifact 依赖图（spec-driven schema）：**

```
                    proposal
                   (root node)
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
      specs                       design
   (requires:                  (requires:
    proposal)                   proposal)
         │                           │
         └─────────────┬─────────────┘
                       │
                       ▼
                    tasks
                (requires:
                specs, design)
```

**Proposal 模板：**

```markdown
## Why
<!-- 动机和问题 -->

## What Changes
<!-- 具体会改变什么 -->

## Capabilities
### New Capabilities
- `<name>`: <描述>

### Modified Capabilities
- `<existing-name>`: <什么需求在变>

## Impact
<!-- 受影响的代码/API/依赖 -->
```

**Delta Spec 格式：**

```markdown
## ADDED Requirements
### Requirement: Two-Factor Authentication
The system MUST support TOTP-based 2FA.

#### Scenario: 2FA enrollment
- GIVEN a user without 2FA enabled
- WHEN the user enables 2FA in settings
- THEN a QR code is displayed

## MODIFIED Requirements
### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

## REMOVED Requirements
### Requirement: Remember Me
(Deprecated in favor of 2FA)
```

---

### Action 2: Apply（实现 change）

```bash
# 1. 了解状态
openspec status --change "add-dark-mode" --json

# 2. 获取实现上下文
openspec instructions apply --change "add-dark-mode" --json
# → contextFiles: proposal.md, design.md, specs/*.md, tasks.md
# → progress: total/complete/remaining

# 3. 循环执行 tasks
for each task:
    - 展示正在做的 task
    - 做代码改动
    - 标记 [ ] → [x] in tasks.md
    - 继续下一个
```

**特征：**
- 单 agent 在当前会话中执行
- 可随时中断、切换 change
- 遇到问题 → 暂停 + 等用户指导
- 没有独立的 review agent

---

### Action 3: Verify（验证实现）[可选]

**三维度分析：**

| 维度 | 检查内容 |
|------|---------|
| **Completeness** | tasks checkbox 统计 + spec 需求 → 代码映射 |
| **Correctness** | 需求实现映射 + scenario 覆盖度 |
| **Coherence** | design 决策是否反映在代码中 + 代码模式一致性 |

**报告格式：**
```
CRITICAL → 必须修复才能 archive
WARNING → 应该修复
SUGGESTION → 可选改进
```

**渐进降级：**
- 只有 tasks → 只查完成度
- tasks + specs → 查 completeness + correctness
- 全部 artifact → 查所有三维度

---

### Action 4: Archive（归档 change）

**操作：**

```
1. 检查 delta specs 是否已 sync
2. 如果未 sync → 提示 sync
3. 执行 delta merge:

   ADDED requirements    → append 到主 spec
   MODIFIED requirements → replace 主 spec 对应项
   REMOVED requirements  → delete from 主 spec

4. 整个 change 文件夹 → changes/archive/YYYY-MM-DD-<name>/
5. 主 spec 现在是更新后的 source of truth
```

**归档前后对比：**

```
Before:                          After:
openspec/                        openspec/
├── specs/                       ├── specs/
│   └── auth/spec.md             │   └── auth/spec.md  ← 含 2FA
└── changes/                     └── changes/
    └── add-2fa/                     └── archive/
        ├── proposal.md                  └── 2025-01-24-add-2fa/
        ├── design.md                        ├── proposal.md  ← 保留
        └── specs/auth/spec.md               └── ...
```

---

## 三、Schema 系统

### 内置 Schema：spec-driven

```yaml
# schemas/spec-driven/schema.yaml
name: spec-driven
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []

  - id: specs
    generates: specs/**/*.md
    requires: [proposal]

  - id: design
    generates: design.md
    requires: [proposal]

  - id: tasks
    generates: tasks.md
    requires: [specs, design]
```

### 自定义 Schema

```bash
# 从零创建
openspec schema init research-first

# 或 fork 已有 schema
openspec schema fork spec-driven research-first
```

```yaml
# 自定义：research-first
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []

  - id: proposal
    generates: proposal.md
    requires: [research]

  - id: tasks
    generates: tasks.md
    requires: [proposal]
```

---

## 四、两个模式

### Core 模式（默认，快速路径）

```
/opsx:propose → /opsx:apply → /opsx:sync → /opsx:archive
```

可用命令：propose / explore / apply / sync / archive

### Expanded 模式（完整控制）

```bash
openspec config profile
openspec update
```

可用命令：新增 new / continue / ff / verify / bulk-archive / onboard

---

## 五、Workspace 系统（beta）

多 repo / monorepo 协调：

```yaml
# workspace.yaml
version: 1
name: platform
links:
  api: /repos/api
  web: /repos/web

# monorepo 场景
links:
  billing: /repos/platform/services/billing
  checkout: /repos/platform/apps/checkout
```

```bash
openspec workspace setup
openspec workspace open
openspec workspace link /repos/api
openspec workspace doctor
```

---

## 六、多平台支持

OpenSpec 通过 `command-generation/adapters/` 支持 25+ AI 工具，自动为每个平台生成对应的 skill 或 slash command。

CLI 工具本身不依赖特定 AI 平台——`openspec update` 命令生成平台特定文件：

```bash
openspec config profile    # 选择 workflow profile
openspec update            # 为目标工具生成配置
```

---

## 七、核心概念总结

| 概念 | 定义 |
|------|------|
| **Spec** | 行为契约（source of truth），包含 requirements + scenarios |
| **Change** | 一次改动提案，是一个文件夹 = proposal + design + delta specs + tasks |
| **Delta Spec** | 增量 spec，描述 ADDED/MODIFIED/REMOVED，archive 时合并到主 spec |
| **Artifact** | change 内的文档（proposal/design/specs/tasks） |
| **Schema** | 定义 artifact 类型和依赖关系 |
| **Archive** | 完成 change 后 delta merge + 移动到 archive 目录 |
| **Workspace** | 多 repo/文件夹协调视图 |

---

## 八、设计哲学

| 原则 | 含义 |
|------|------|
| **fluid, not rigid** | actions 不是 phases，顺序自由 |
| **iterative, not waterfall** | 需求和理解会变，拥抱变化 |
| **easy, not complex** | 轻量初始化，最少仪式 |
| **brownfield-first** | delta 方式天然适配既有代码库 |

> "Specs describe current behavior. Changes propose modifications (as deltas). Implementation makes changes real. Archive merges deltas into specs. Specs now describe the new behavior. The virtuous cycle continues."
