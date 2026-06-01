# 09 - 并行 Agent 调度

**文件:** `skills/dispatching-parallel-agents/SKILL.md`

## 使用场景

> 当你有多个无关的失败（不同测试文件、不同子系统、不同 bug）时，顺序调查浪费时间的每一秒都是浪费。

## 适用条件

**应该使用:**
- 3+ 测试文件因不同根因失败
- 多个子系统独立损坏
- 每个问题不需要其他问题的上下文就能理解
- 调查之间没有共享状态

**不应该使用:**
- 失败是相关的（修复一个可能修复其他）
- 需要理解完整系统状态
- Agent 会互相互干扰

## 使用模式

```
1. 识别独立的问题域
2. 为每个域创建聚焦的 agent task
3. 并行 dispatch
4. 审查和整合结果
```

### 决策流

```
Multiple failures?
  ├── 相关 → 一个 agent 一起调查
  └── 独立 → 可以并行吗？
      ├── 有共享状态 → 顺序 agent
      └── 无共享状态 → 并行 dispatch
```

### Agent Prompt 结构

好的并行 agent prompt 必须：

1. **聚焦** — 一个清晰的问题域
2. **自包含** — 理解问题需要的所有上下文
3. **具体** — 指定输出格式

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture"
2. "should handle mixed completed and aborted tools"
3. "should properly track pendingToolCount"

These are timing/race condition issues. Your task:
1. Read the test file and understand what each test verifies
2. Identify root cause
3. Fix by event-based waiting (NOT just increasing timeouts)

Return: Summary of what you found and what you fixed.
```

## 常见错误

| ❌ | ✅ |
|----|-----|
| "Fix all the tests" — agent 迷失 | "Fix agent-tool-abort.test.ts" — 聚焦范围 |
| "Fix the race condition" — 不知道在哪儿 | 附上错误消息和测试名 |
| 无约束 — agent 可能重构一切 | "Do NOT change production code" 或 "Fix tests only" |
| "Fix it" — 你不知道改了什么 | "Return summary of root cause and changes" |

## 整合

所有 agent 返回后：
1. **读每个摘要** — 理解改了什么
2. **检查冲突** — agent 编辑了相同代码吗？
3. **跑完整测试套件** — 验证所有修复一起工作
4. **抽查** — agent 可能有系统性错误

## 与 Subagent-Driven Development 的区别

| | subagent-driven-dev | dispatching-parallel-agents |
|--|---------------------|----------------------------|
| 用途 | 顺序执行 plan 的每个 task | 同时调试多个独立问题 |
| agent 数量 | 每次 1 个（顺序） | 多个同时（并行） |
| 顺序保证 | 严格顺序（Task 1→2→3...） | 无顺序（独立问题） |
| 审核 | 两个 formal gate per task | 人工 review 结果 |
| 适用 | 实现了 plan 的有序任务 | 调试/探索阶段 |

## 效率

从实际调试会话 (2025-10-03):
- 6 个失败分布在 3 个文件
- 3 个 agent 并行 dispatch
- 所有调查并行完成
- 所有修复集成成功
- agent 变更之间零冲突
- **时间节省: 3 个问题在 1 个问题的时间内解决**
