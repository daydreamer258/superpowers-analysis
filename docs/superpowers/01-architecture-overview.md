# 01 - 整体架构概览

## 三层结构

```
┌─────────────────────────────────────────────────────┐
│  Session Start Hook                                 │
│  注入 using-superpowers 到每个会话                     │
│  效果: Agent 被"附魔"，获得强制使用 skill 的意识          │
└───────────────┬─────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────┐
│  流程编排层 (5 个阶段固定流转)                           │
│  brainstorming → writing-plans → subagent-driven      │
│  → finishing-a-development-branch                     │
└───────────────┬─────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────┐
│  纪律约束层 (交叉切割的技能)                             │
│  TDD / verification-before-completion / git-worktrees │
│  requesting-code-review / receiving-code-review       │
└─────────────────────────────────────────────────────┘
```

## 15 个核心 Skill

| Skill | 层 | 角色 |
|-------|-----|------|
| `using-superpowers` | Hook | 启动时注入，告诉 agent 如何使用 skill |
| `brainstorming` | 编排 | 需求澄清 → 设计 → 输出 spec 文档 |
| `writing-plans` | 编排 | spec → 拆成 bite-sized task |
| `subagent-driven-development` | 编排 | 核心执行引擎，per-task subagent + 双 review |
| `executing-plans` | 编排 | 备选方案：当前会话顺序执行 |
| `dispatching-parallel-agents` | 编排 | 多独立问题并行调试 |
| `finishing-a-development-branch` | 编排 | 完成后的合并/PR/清理 |
| `using-git-worktrees` | 约束 | 隔离 workspace |
| `test-driven-development` | 约束 | 铁律：没有失败测试就没有生产代码 |
| `verification-before-completion` | 约束 | 铁律：声称完成前必须跑验证命令 |
| `requesting-code-review` | 约束 | 发起 review 的标准模板 |
| `receiving-code-review` | 约束 | 接收 review 反馈的行为准则 |
| `systematic-debugging` | 约束 | 系统性调试方法论 |
| `writing-skills` | 元 | 编写 skill 的最佳实践 |
| `brainstorming/visual-companion` | 辅助 | 浏览器端可视化 mockup |

## 完整状态机

```
User Request
     │
     ▼
[SessionStart Hook: 注入 using-superpowers]
     │
     ▼
superpowers:brainstorming (9 步)
     │  输出: spec 文档 → git commit
     ▼
superpowers:writing-plans
     │  输出: plan 文档 (所有 task 完整定义)
     ▼
├── superpowers:using-git-worktrees (隔离 workspace)
│
▼
superpowers:subagent-driven-development
     │
     │  ┌─ Task 1 ────────────────────────────┐
     │  │  Implementer → Spec Review →         │
     │  │  Code Quality Review → ✅            │
     │  └──────────────────────────────────────┘
     │  ┌─ Task 2 ────────────────────────────┐
     │  │  Implementer → BLOCKED               │
     │  │  → 加强模型 → ✅                      │
     │  └──────────────────────────────────────┘
     │  ...
     │  ┌─ Final Code Review (全量) ──────────┐
     │  └──────────────────────────────────────┘
     ▼
superpowers:finishing-a-development-branch
     │  4 个选项 → 用户选择 → 执行 → 清理
     ▼
Done ✅
```

**贯穿全程的垂直约束:**
- `test-driven-development` — 开发者 subagent 在每个 task 内必须遵守
- `verification-before-completion` — 任何时候声称"完成"前必须跑验证
- `requesting-code-review` / `receiving-code-review` — 审核的标准模板和接收行为

---

## 与 OpenSpec 的架构对比

| 维度 | Superpowers | OpenSpec |
|------|------------|----------|
| 层数 | 3 层（Hook → 编排 → 纪律） | 2 层（CLI + AI Skill） |
| 核心载体 | Markdown prompt 文件 | TypeScript CLI + YAML schema |
| 平台适配 | 1 个 bash hook 检测环境变量 | 25+ 独立 adapter 文件 |
| 状态管理 | 无（依赖会话上下文） | artifact graph 引擎 |
| 阶段强制性 | 严格顺序（hard gate） | 自由顺序（fluid, not rigid） |
| 产出管理 | spec/plan 作为工作文档 | spec 是 source of truth（delta merge） |
