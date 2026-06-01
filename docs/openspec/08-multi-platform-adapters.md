# OpenSpec 08 — 多平台支持与 25+ 适配器

## 适配器架构

OpenSpec 的独特设计：**CLI 管状态，AI 工具管执行。** `openspec update` 通过适配器将 workflow 输出为每个平台识别的格式。

```
src/core/command-generation/
├── adapters/
│   ├── claude.ts           # Claude Code
│   ├── cursor.ts           # Cursor
│   ├── github-copilot.ts   # GitHub Copilot
│   ├── codex.ts            # OpenAI Codex CLI
│   ├── gemini.ts           # Gemini CLI
│   ├── windsurf.ts         # Windsurf
│   ├── roocode.ts          # RooCode (VS Code)
│   ├── cline.ts            # Cline (VS Code)
│   ├── factory.ts          # Factory Droid
│   ├── opencode.ts         # OpenCode
│   ├── codebuddy.ts        # CodeBuddy
│   ├── qwen.ts             # Qwen Code
│   ├── pi.ts               # PI
│   ├── auggie.ts           # Auggie
│   ├── bob.ts              # Bob
│   ├── kiro.ts             # Kiro
│   ├── kilocode.ts         # KiloCode
│   ├── junie.ts            # Junie
│   ├── qoder.ts            # Qoder
│   ├── costrict.ts         # Costrict
│   ├── crush.ts            # Crush
│   ├── continue.ts         # Continue
│   ├── iflow.ts            # iFlow
│   ├── lingma.ts           # Lingma (Alibaba)
│   ├── antigravity.ts      # Antigravity
│   └── amazon-q.ts         # Amazon Q Developer
├── generator.ts            # 生成引擎
├── registry.ts             # 适配器注册
└── types.ts                # 类型定义
```

## 适配器的工作原理

每个适配器是一个 TypeScript 模块，职责：

1. **声明该平台的特性和限制**（支持 skill？支持 slash command？支持 rule 文件？）
2. **定义输出格式**（Markdown skill / JSON rule / Yaml command / etc.）
3. **提供文件写入路径**（`.claude/skills/` / `.cursor/rules/` / `.github/prompts/`）
4. **处理平台特定的语法要求**（缩进、frontmatter、特殊标记）

生成引擎 (`generator.ts`) 读取 schema + profile 配置，然后对选中的每个 adapter 调用生成方法。

## 各平台输出差异

| 平台 | 输出格式 | 位置 |
|------|---------|------|
| Claude Code | Skill (.md) | `.claude/skills/openspec-*.md` |
| Cursor | Rules (.mdc) | `.cursor/rules/openspec-*.mdc` |
| GitHub Copilot | Prompts (.md) | `.github/prompts/openspec-*.prompt.md` |
| Codex | Skills (.md) | `.codex/skills/openspec-*.md` |
| Gemini CLI | Extensions | `gemini-extension.json` |
| OpenCode | Plugins (.js) | `.opencode/plugins/` |
| Windsurf | Rules | `.windsurfrules` |
| RooCode | Custom modes | `.roomodes` |

## 生成的内容

`openspec update` 生成两类内容：

### 1. Skill / Slash Command 文件

对应每个 workflow action（propose, apply, verify, archive, explore...）生成一个指令文件。内容是将 `src/core/templates/workflows/` 中的模板按平台格式输出。

### 2. AGENTS.md

生成的 `AGENTS.md` 包含：
- 项目级别的 OpenSpec 配置摘要
- 所有可用 slash command 的描述
- 如何发现和使用 change 的指导
- workspace 级别（如果适用）

## Profile × Adapter 组合

```
用户选择:
  schema: spec-driven
  profile: core (或 expanded)
  tools: [claude, cursor]

openspec update:
  对每个 tool:
    生成 core（或 expanded）profile 对应的 command 文件

core profile 生成:
  - propose
  - explore
  - apply
  - sync
  - archive

expanded profile 生成:
  + new, continue, ff, verify, bulk-archive, onboard
```

## 与 Superpowers 多平台支持的对比

| | OpenSpec | Superpowers |
|--|----------|-------------|
| 适配方式 | 25+ 独立 adapter 文件，每个平台显式支持 | 1 个 bash hook 检测环境变量 |
| 输出格式 | 各平台原生格式 | 统一 Markdown + JSON 注入 |
| 需要 CLI | ✅ | ❌ |
| 配置文件 | 自动生成 + 用户可编辑 | hook 自动检测 |
| 平台发现 | 用户显式选择 | 通过 `CLAUDE_PLUGIN_ROOT` / `CURSOR_PLUGIN_ROOT` 等环境变量 |
| 脆弱性 | 每个 adapter 需要单独维护 | 依赖环境变量稳定性 |
