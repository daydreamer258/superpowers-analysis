# OpenSpec 01 — 整体架构概览

## 双层架构

OpenSpec 由两层组成：

```
┌─────────────────────────────────────────────────────┐
│  CLI 工具层 (openspec)                                │
│  Node.js CLI, ~200+ TypeScript 源文件                 │
│  职责: 管理状态、生成 AI 配置、验证、归档              │
└───────────────┬─────────────────────────────────────┘
                │ 通过 openspec update 生成
┌───────────────▼─────────────────────────────────────┐
│  AI Skill / Slash Command 层                         │
│  为 25+ AI 工具自动生成的指令文件                      │
│  职责: 驱动 AI agent 执行 spec-driven workflow        │
└─────────────────────────────────────────────────────┘
```

**关键设计决策：CLI 管状态，AI 管执行。** 两者之间通过 `openspec update` 生成的文件耦合。

## 项目仓库中的产物结构

```
openspec/
├── config.yaml              ← OpenSpec 配置
├── specs/                   ← source of truth（系统行为）
│   ├── auth/spec.md
│   ├── payments/spec.md
│   └── ui/spec.md
└── changes/                 ← 进行中的改动
    ├── add-dark-mode/
    │   ├── proposal.md         ← 为什么 + 范围
    │   ├── design.md           ← 技术方案
    │   ├── tasks.md            ← 实现 checklist
    │   ├── .openspec.yaml      ← change 元数据
    │   └── specs/              ← delta specs（增量变更）
    └── archive/                ← 已完成并归档的 change
        └── 2025-01-24-add-2fa/
```

## 两个 spec 区域的关系

```
 specs/ (main)          changes/<name>/specs/ (delta)
┌──────────────┐       ┌──────────────────────┐
│ source of    │◄──────│ ADDED requirements    │
│ truth        │ merge │ MODIFIED requirements │
│ (current)    │       │ REMOVED requirements  │
└──────────────┘       └──────────────────────┘

   archive 后:
   - delta 合并到 main specs
   - change 文件夹移动到 archive/
   - main specs 成为更新后的 source of truth
```

## Schema 系统：定义 artifact 依赖图

OpenSpec 通过 YAML schema 定义每个 workflow 的 artifact 类型和依赖：

```yaml
# schemas/spec-driven/schema.yaml
name: spec-driven
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []              # 无依赖，可以先创建

  - id: specs
    generates: specs/**/*.md
    requires: [proposal]      # 需要 proposal 先完成

  - id: design
    generates: design.md
    requires: [proposal]      # 可以和 specs 并行创建

  - id: tasks
    generates: tasks.md
    requires: [specs, design] # 需要 specs + design 都完成
```

**依赖图：**
```
                    proposal
                       │
         ┌─────────────┴─────────────┐
         │                           │
      specs                       design
         │                           │
         └─────────────┬─────────────┘
                       │
                    tasks
```

## 25+ AI 工具适配器

`src/core/command-generation/adapters/` 为每个平台生成不同的指令格式：

| 适配器 | 平台 |
|--------|------|
| claude.ts | Claude Code |
| cursor.ts | Cursor |
| github-copilot.ts | GitHub Copilot |
| codex.ts | OpenAI Codex |
| gemini.ts | Gemini CLI |
| windsurf.ts | Windsurf |
| roocode.ts | RooCode |
| cline.ts | Cline |
| factory.ts | Factory Droid |
| opencode.ts | OpenCode |
| codebuddy.ts | CodeBuddy |
| ... | +15 更多 |

每个适配器知道该平台的 command/skill 格式，`openspec update` 自动生成对应文件。

## 完整状态机

```
User Request
     │
     ▼
/opsx:propose (或 /opsx:new → /opsx:continue → /opsx:ff)
     │  输出: proposal.md + specs/ + design.md + tasks.md
     ▼
/opsx:apply
     │  循环执行 tasks.md 中的 checkbox
     │  标记 [ ] → [x]
     ▼
/opsx:verify (可选)
     │  三维度报告: Completeness / Correctness / Coherence
     ▼
/opsx:archive
     │  delta merge → main specs
     │  change → archive/
     ▼
Done ✅
```

**"fluid" 特性：** 这些 action 可以乱序调用。比如先 /apply 然后再 /propose 补文档——虽然不推荐，但技术上不阻止。

## 核心命令一览

| 命令 | 用途 |
|------|------|
| `openspec init` | 初始化项目（创建 openspec/ 结构） |
| `openspec update` | 刷新 AI 工具配置文件 |
| `openspec config profile` | 选择 workflow profile |
| `openspec new change <name>` | 创建 change 脚手架 |
| `openspec status --change <name>` | 查看 change 状态 |
| `openspec instructions <artifact> --change <name>` | 获取 artifact 生成指令 |
| `openspec list` | 列出所有 change |
| `openspec validate` | 验证 spec 结构 |
| `openspec schema init/fork` | 创建/复制自定义 schema |
| `openspec workspace setup` | 创建多 repo workspace（beta） |
