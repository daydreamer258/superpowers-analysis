# 02 - 启动与注入机制

## SessionStart Hook 是整个 Superpowers 的启动器

**文件:** `hooks/hooks.json` + `hooks/session-start`

## hooks.json — 事件绑定规则

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "run-hook.cmd session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

**关键设计点:**
- 在 3 个场景触发: `startup`（新建会话）、`clear`（清空上下文）、`compact`（上下文压缩）
- `async: false` → 同步阻塞执行，确保注入完成后 agent 才可用
- 这覆盖了 agent 所有"重新开始"的时刻

## session-start 脚本做了什么

1. **读取 `skills/using-superpowers/SKILL.md` 全文**
2. **转义为 JSON 字符串**（自定义的 bash `escape_for_json` 函数，不用 jq，极致性能）
3. **注入到系统提示词**，根据运行平台用不同的 JSON 字段格式：

| 平台 | 注入方式 |
|------|---------|
| Cursor | `additional_context` |
| Claude Code | `hookSpecificOutput.additionalContext` |
| Copilot CLI | `additionalContext` (SDK 标准) |

## 注入后的效果：Agent 的"潜意识"被改写

Agent 在每个会话启动时就看到这段内容：

```
<EXTREMELY_IMPORTANT>
You have superpowers.

**Below is the full content of your 'superpowers:using-superpowers' skill...**
</EXTREMELY_IMPORTANT>
```

`using-superpowers` 的核心信条：

> "IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT."
>
> "甚至 1% 的可能性 → 也必须调用"
>
> "这不是可选的，这不是可协商的。你不能合理化逃避。"

同时给了一组 Red Flags 表来防止 agent 给自己找借口：

| 内心戏 | 现实 |
|--------|------|
| "这只是一个简单问题" | 问题也是任务，检查 skill |
| "我需要先了解上下文" | skill 检查必须先于上下文探索 |
| "让我先探索代码库" | skill 告诉你怎么探索，先检查 |
| "这个 skill 我知道内容" | skill 会演化，读最新版 |
| "skill 对这个场景太重了" | 简单的事也变复杂，必须用 |

## 为什么用 Hook 而不是直接放在系统提示词

Superpowers 最初是一个 Claude Code 插件。Claude Code 的插件系统提供了 SessionStart Hook，允许插件在每次会话启动时注入上下文。

这样做的优势：
- **零配置安装** — 用户安装插件后自动生效
- **不污染项目级提示词** — 不修改用户的 CLAUDE.md
- **多平台适配** — 同一个 hook 脚本通过环境变量判断平台，输出对应格式
- **可演化** — 更新插件就能更新所有用户的注入内容

## Skill 调用优先级

```
1. 用户指令 (CLAUDE.md, GEMINI.md, AGENTS.md, 直接请求) — 最高
2. Superpowers skills — 覆盖默认系统行为
3. 默认系统 prompt — 最低
```

**如果用户说 "不要用 TDD" 而 skill 说 "必须用 TDD" → 用户指令获胜。用户拥有最终控制权。**

## Skill 类型

- **刚性技能** (TDD, debugging): 严格遵循，不能"灵活调整"。违反其中的纪律 = 失败。
- **柔性技能** (patterns): 根据上下文调整原则，但原则本身不可弃。

每个 skill 本身会告诉你它是哪种类型。
