# OpenSpec vs Superpowers — 项目结构差异

## 一、OpenSpec：CLI 驱动的多平台框架

### 代码仓库结构

```
Fission-AI/OpenSpec/
├── bin/openspec.js              # CLI 入口
├── package.json                 # npm 包: @fission-ai/openspec
├── tsconfig.json                # TypeScript 配置
├── vitest.config.ts             # 测试框架配置
│
├── src/
│   ├── cli/index.ts             # CLI 命令路由
│   ├── commands/                # 20+ 命令实现
│   │   ├── change.ts            # openspec change ...
│   │   ├── config.ts            # openspec config ...
│   │   ├── spec.ts              # openspec spec ...
│   │   ├── validate.ts          # openspec validate
│   │   ├── workspace.ts         # openspec workspace ...
│   │   └── workflow/            # 内部 workflow 辅助
│   │       ├── instructions.ts
│   │       ├── new-change.ts
│   │       ├── schemas.ts
│   │       ├── templates.ts
│   │       └── status.ts
│   │
│   ├── core/                    # ~100 个核心模块
│   │   ├── artifact-graph/      # artifact 依赖图引擎
│   │   │   ├── graph.ts
│   │   │   ├── resolver.ts
│   │   │   ├── schema.ts
│   │   │   ├── state.ts
│   │   │   └── instruction-loader.ts
│   │   ├── command-generation/  # 为 AI 工具生成 skill/command
│   │   │   ├── adapters/        # 25+ 平台适配器
│   │   │   │   ├── claude.ts
│   │   │   │   ├── cursor.ts
│   │   │   │   ├── github-copilot.ts
│   │   │   │   ├── codex.ts
│   │   │   │   ├── gemini.ts
│   │   │   │   └── ... (windsurf, roocode, cline, factory, etc.)
│   │   │   ├── generator.ts
│   │   │   └── registry.ts
│   │   ├── templates/           # 模板和 skill 生成
│   │   │   ├── skill-templates.ts
│   │   │   └── workflows/       # 每个 workflow action 的模板
│   │   │       ├── propose.ts
│   │   │       ├── apply-change.ts
│   │   │       ├── verify-change.ts
│   │   │       ├── archive-change.ts
│   │   │       ├── new-change.ts
│   │   │       ├── ff-change.ts
│   │   │       ├── explore.ts
│   │   │       └── sync-specs.ts
│   │   ├── config.ts            # 配置管理
│   │   ├── init.ts              # 项目初始化
│   │   ├── update.ts            # 更新配置
│   │   ├── archive.ts           # 归档逻辑
│   │   ├── list.ts              # 列出 changes
│   │   ├── profiles.ts          # 工作流 profile 选择
│   │   ├── context-store/       # 跨会话上下文存储
│   │   ├── collections/         # initiatives 集合
│   │   ├── workspace/           # workspace 管理（beta）
│   │   ├── validation/          # 验证引擎
│   │   ├── parsers/             # markdown/spec 解析
│   │   └── converters/          # 格式转换（JSON 等）
│   │
│   └── utils/                   # 工具函数
│
├── schemas/                     # Schema 定义 + 模板
│   ├── spec-driven/
│   │   ├── schema.yaml          # artifact 依赖图定义
│   │   └── templates/           # 每个 artifact 的文件模板
│   │       ├── proposal.md
│   │       ├── design.md
│   │       ├── spec.md
│   │       └── tasks.md
│   └── workspace-planning/      # workspace 用的 schema
│       └── ...
│
├── docs/                        # 用户文档
│   ├── concepts.md
│   ├── workflows.md
│   ├── commands.md
│   ├── customization.md
│   └── ...
│
├── test/                        # ~80+ 测试文件
│   ├── cli-e2e/
│   ├── commands/
│   ├── core/
│   └── fixtures/
│
└── openspec/                    # OpenSpec 自己吃自己的 dogfood
    ├── specs/                   # OpenSpec 的行为 spec
    │   ├── cli-init/spec.md
    │   ├── artifact-graph/spec.md
    │   └── ... (40+ spec 文件)
    └── changes/                 # OpenSpec 自己的 change 历史
        ├── add-global-install-scope/
        ├── add-change-stacking-awareness/
        └── archive/             # ~15 个已归档 change
```

### OpenSpec 在你的项目中的结构

当你在项目里运行 `openspec init` 后：

```
your-project/
├── openspec/
│   ├── config.yaml             # OpenSpec 配置
│   ├── specs/                  # 系统行为 spec（source of truth）
│   │   ├── auth/spec.md
│   │   ├── payments/spec.md
│   │   └── ui/spec.md
│   └── changes/                # 进行中的改动
│       ├── add-dark-mode/
│       │   ├── proposal.md     # 为什么 + 范围
│       │   ├── design.md       # 技术方案
│       │   ├── tasks.md        # 实现 checklist
│       │   ├── .openspec.yaml  # change 元数据
│       │   └── specs/          # delta specs
│       │       └── ui/spec.md  # 对 ui/spec.md 的增量修改
│       └── fix-login-redirect/
│           ├── proposal.md
│           ├── tasks.md
│           └── specs/auth/spec.md
├── AGENTS.md                   # 生成的 agent 指导
├── CLAUDE.md                   # 如果选了 Claude Code
└── .cursor/rules/              # 如果选了 Cursor
```

---

## 二、Superpowers：纯 Prompt 驱动的 Skills 集合

### 代码仓库结构

```
obra/superpowers/
├── package.json                # 只有元数据，无代码逻辑
├── CLAUDE.md                   # 贡献者指南（著名的 94% PR 拒绝率）
│
├── hooks/                      # 唯二的"代码"
│   ├── hooks.json              # SessionStart hook 配置
│   ├── session-start           # ~40 行 bash 脚本
│   └── run-hook.cmd            # Windows 支持
│
├── skills/                     # 15 个 Markdown 文件 = 全部"业务逻辑"
│   ├── using-superpowers/
│   │   └── SKILL.md            # 入口 skill：强制使用 skill 的规则
│   │
│   ├── brainstorming/
│   │   ├── SKILL.md            # 需求澄清 9 步流程
│   │   ├── visual-companion.md # 浏览器 mockup 辅助
│   │   ├── spec-document-reviewer-prompt.md
│   │   └── scripts/            # 辅助脚本
│   │       ├── server.cjs
│   │       ├── start-server.sh
│   │       └── stop-server.sh
│   │
│   ├── writing-plans/
│   │   ├── SKILL.md            # plan 编写规则（零 placeholder）
│   │   └── plan-document-reviewer-prompt.md
│   │
│   ├── subagent-driven-development/
│   │   ├── SKILL.md            # 核心执行引擎
│   │   ├── implementer-prompt.md        # 实现者 prompt 模板
│   │   ├── spec-reviewer-prompt.md      # spec 审核 prompt 模板
│   │   └── code-quality-reviewer-prompt.md # 代码质量审核模板
│   │
│   ├── executing-plans/
│   │   └── SKILL.md            # 备选方案：内联执行
│   │
│   ├── test-driven-development/
│   │   ├── SKILL.md            # TDD 铁律 + Rationalization 表
│   │   └── testing-anti-patterns.md
│   │
│   ├── verification-before-completion/
│   │   └── SKILL.md            # 第二铁律：声称完成前必须验证
│   │
│   ├── finishing-a-development-branch/
│   │   └── SKILL.md            # 完成+清理流程
│   │
│   ├── using-git-worktrees/
│   │   └── SKILL.md            # 隔离 workspace 管理
│   │
│   ├── requesting-code-review/
│   │   ├── SKILL.md
│   │   └── code-reviewer.md    # 审核模板
│   │
│   ├── receiving-code-review/
│   │   └── SKILL.md            # 接收反馈的行为准则
│   │
│   ├── dispatching-parallel-agents/
│   │   └── SKILL.md            # 多独立问题并行调试
│   │
│   ├── systematic-debugging/
│   │   ├── SKILL.md
│   │   ├── root-cause-tracing.md
│   │   ├── defense-in-depth.md
│   │   ├── condition-based-waiting.md
│   │   └── find-polluter.sh
│   │
│   └── writing-skills/
│       ├── SKILL.md            # 元技能：如何写 skill
│       ├── anthropic-best-practices.md
│       ├── persuasion-principles.md
│       └── testing-skills-with-subagents.md
│
├── .opencode/plugins/          # OpenCode 平台集成
│   └── superpowers.js
│
├── .cursor-plugin/             # Cursor 平台集成
│   └── plugin.json
│
├── gemini-extension.json       # Gemini 平台集成
│
├── scripts/                    # 辅助脚本
│   ├── bump-version.sh
│   └── sync-to-codex-plugin.sh
│
└── tests/                      # 集成测试
    ├── subagent-driven-dev/    # 端到端测试用例
    │   ├── go-fractals/
    │   └── svelte-todo/
    └── claude-code/            # Claude Code 上跑集成测试
```

### Superpowers 在你的环境中的结构

安装后，Superpowers 以插件形式存在，不在你的项目里创建文件：

```text
~/.claude/plugins/superpowers/     (Claude Code)

或

~/.cursor/plugins/superpowers/     (Cursor)

或

~/.config/superpowers/             (历史全局路径)
```

你的项目里只会生成：
```text
your-project/
├── docs/superpowers/
│   ├── specs/
│   │   └── 2026-01-15-add-auth-design.md     # brainstorming 产出
│   └── plans/
│       └── 2026-01-15-add-auth-plan.md        # writing-plans 产出
└── .worktrees/                                 # git worktree 隔离
    └── add-auth/
```

---

## 三、关键结构差异总结

| 维度 | OpenSpec | Superpowers |
|------|----------|-------------|
| **代码量** | ~200+ TypeScript 源文件 | 0 行业务逻辑代码 |
| **核心载体** | CLI 工具（npm 包） | Markdown 提示词文件 |
| **在用户项目里** | 创建 `openspec/` 文件夹结构 | 只在 `docs/superpowers/` 写 spec/plan 文档 |
| **版本管理** | spec 文件 commit 到 repo（source of truth） | spec/plan 作为工作文档 commit 到 repo |
| **多平台适配** | 25+ 适配器，自动生成配置文件 | 通过 hook 脚本检测平台（环境变量） |
| **状态管理** | artifact graph + JSON status | 无需状态（agent 在会话中靠 skill 规则执行） |
| **自定义能力** | 自定义 schema（YAML 定义 artifact 依赖图） | 自定义 skill（写 markdown 文件） |
| **自我 dogfood** | OpenSpec 仓库自己用 OpenSpec 管理 | Superpowers 自己没有 dogfood 机制 |
| **语言** | TypeScript (Node.js >= 20.19) | Bash + Markdown (零运行时依赖) |

### 为什么会有这样的差异？

这是两种完全不同的思路：

**OpenSpec = 基础设施思维**
- 需要状态管理 → artifact graph 引擎
- 需要多平台兼容 → 25 个适配器
- 需要验证 → 解析器 + validator
- 这是典型的"框架式"开发

**Superpowers = 约束工程思维**
- 不需要状态 → 会话上下文自然携带
- 不需要跨平台代码 → 用一个 hook + 环境变量判断
- 不需要验证器 → 靠 agent 自己读规则并遵守
- 这是极致的"prompt engineering"
