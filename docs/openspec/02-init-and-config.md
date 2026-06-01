# OpenSpec 02 — 初始化和配置注入机制

## openspec init 做了什么

```bash
cd your-project
openspec init
```

**步骤：**

1. **创建 `openspec/` 目录结构**
   ```
   openspec/
   ├── config.yaml
   ├── specs/        ← 空目录，等待添加 spec
   └── changes/      ← 空目录，等待创建 change
   ```

2. **生成 `config.yaml`**
   ```yaml
   # openspec/config.yaml
   schema: spec-driven    # 默认 schema
   # profile: core        # 默认 profile（快速路径）
   ```

3. **交互式配置（可选）**
   - 选择 workflow profile（core vs expanded）
   - 选择 AI 工具（Claude Code / Cursor / Copilot...）

4. **运行 `openspec update` 生成 AI 配置**

## openspec update 做了什么

这是关键的"注入"步骤——将 OpenSpec workflow 注入到 AI 环境中。

```bash
openspec update
```

**过程：**

```
1. 读取 openspec/config.yaml → 确定 schema 和 profile
2. 读取全局配置 → 确定目标工具和 delivery mode
3. 对每个选中的工具：
   a. 调用对应的 adapter（如 claude.ts）
   b. 生成该工具的 skill/command 文件
   c. 写入到工具识别的路径
4. 生成/更新 AGENTS.md（通用 agent 指导）
```

**生成的产物因平台而异：**

| 平台 | 生成位置 | 格式 |
|------|---------|------|
| Claude Code | `.claude/skills/openspec-*.md` | Markdown skill 文件 |
| Cursor | `.cursor/rules/openspec-*.mdc` | Cursor rules |
| GitHub Copilot | `.github/copilot-instructions.md` + `.github/prompts/` | Prompt 文件 |
| Codex | `.codex/skills/` | Codex skill 文件 |
| OpenCode | `.opencode/` | OpenCode 插件格式 |

## Profile 系统

OpenSpec 提供两级 workflow profile：

### Core Profile（默认，快速路径）

```
可用 slash command:
/opsx:propose    — 创建 change + 生成所有 artifacts
/opsx:explore    — 探索想法
/opsx:apply      — 实现 tasks
/opsx:sync       — 合并 delta specs
/opsx:archive    — 归档完成
```

### Expanded Profile（完整控制）

```
额外的 slash command:
/opsx:new         — 只创建 change 脚手架
/opsx:continue    — 逐步创建下一个 artifact
/opsx:ff          — 一次性创建所有 artifact
/opsx:verify      — 验证实现
/opsx:bulk-archive — 批量归档
/opsx:onboard     — 引导新项目
```

使用 `openspec config profile` 切换。

## 指令注入的工作原理

不同于 Superpowers 在 session 启动时注入整个 skill 内容，OpenSpec 的 AI 指令通过 **CLI 查询 + 模板拼接** 动态生成。

**当 AI agent 执行 `/opsx:propose` 时：**

```
1. Agent 读取 skill 文件（由 openspec update 生成）
2. Skill 文件中有完整的 propose 流程指令
3. 流程中调用 openspec CLI:
   ├── openspec new change "name"        # 创建脚手架
   ├── openspec status --change "name" --json  # 获取 artifact 状态
   └── openspec instructions <id> --change "name" --json  # 获取 artifact 指令
4. openspec instructions 返回:
   ├── context: 项目背景（constraints for agent）
   ├── rules: artifact 特定规则（constraints for agent）
   ├── template: 文件结构模板
   ├── instruction: schema 特定指导
   ├── resolvedOutputPath: 输出路径
   └── dependencies: 已完成 artifact 的路径列表
5. Agent 用这些信息生成 artifact 内容
```

**关键设计：** CLI 在运行时通过 `--json` 输出为 agent 提供上下文，而不是一次性注入所有内容。

## 与 Superpowers Hook 注入的对比

| | OpenSpec | Superpowers |
|--|----------|-------------|
| 注入方式 | `openspec update` 生成静态文件 + CLI `--json` 动态查询 | SessionStart hook 注入完整 skill 内容 |
| 注入时机 | 文件一次生成 + 运行时按需查询 | 每次会话启动 |
| 灵活性 | 高（可按 artifact 查询上下文） | 低（整个 skill 文本一次注入） |
| 需要 CLI | ✅ 必须 | ❌ 不需要 |
| 平台适配 | 25+ 适配器显式支持 | 通过环境变量判断 |
