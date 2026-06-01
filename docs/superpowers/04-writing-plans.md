# 04 - Writing Plans 计划拆解

**文件:** `skills/writing-plans/SKILL.md`

## 启动自我宣告

> "I'm using the writing-plans skill to create the implementation plan."

## 核心理念

> Write comprehensive implementation plans assuming the engineer has **zero context for our codebase and questionable taste.**

假设你的实现工程师：
- 是熟练开发者，但对工具链和问题领域几乎一无所知
- 不太懂好的测试设计

因此你需要把**所有东西**都写在 plan 里：
- 每一步该碰哪个文件
- 完整代码（不能引用"类似 Task N"）
- 精确的命令和预期输出
- 需要参考的文档

## Bite-Sized - 粒度的极致

**每步 2-5 分钟，每个步骤是一个可独立追踪的 checkbox：**

```
"写失败的测试" → 一步
"运行测试确保失败" → 一步
"写最小实现代码" → 一步
"运行测试确保通过" → 一步
"Commit" → 一步
```

## Plan Header 模板

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
>   (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]
**Architecture:** [2-3 sentences about approach]
**Tech Stack:** [Key technologies/libraries]
```

## Task 结构模板

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## 严禁 Placeholder

这些是 **plan 失败**，永远不能出现：

| ❌ 失败模式 | 为什么失败 |
|------------|-----------|
| "TBD" / "TODO" / "implement later" / "fill in details" | 留了未完成的坑 |
| "Add appropriate error handling" / "add validation" / "handle edge cases" | 模糊指令，没人知道怎么做 |
| "Write tests for the above" （不写实际测试代码） | 没有可执行的指导 |
| "Similar to Task N" | 阅读者可能不按顺序，必须重复完整内容 |
| 只有描述没有代码块 | 代码步骤必须有完整代码 |
| 引用未在任何 task 中定义的类型/函数/方法 | 孤立的引用 |

## 文件结构设计

在定义 task 之前，先映射哪些文件会被创建或修改，每个文件负责什么：

- 设计有清晰边界的单元
- 偏好小而聚焦的文件，而不是大而全的
- 文件按职责拆分，不是按技术层
- 在既有代码库中遵循已有模式

## 自审（写完 Plan 后自我检查）

1. **Spec 覆盖检查** — 浏览 spec 的每个章节，能不能指到对应的 task？
2. **Placeholder 扫描** — 搜索所有红标记（TBD/TODO/模糊指令）
3. **类型一致性** — Task 3 里的 `clearLayers()` 和 Task 7 里的 `clearFullLayers()` 匹配吗？

发现问题就内联修复。如果发现 spec 要求没有对应的 task → 补上。

## 范围检查

如果 spec 覆盖多个独立子系统，应该在 brainstorming 阶段就拆分掉了。如果没拆，建议分成多个 plan——每个子系统一个。每个 plan 必须产出可运行、可测试的软件。

## 执行交接

Plan 写完后，给用户两个选项：

> "Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:
>
> 1. **Subagent-Driven (recommended)** — 每 task 一个 fresh subagent + 双阶段 review
> 2. **Inline Execution** — 当前会话顺序执行 + checkpoint
>
> Which approach?"

---

## 与 OpenSpec 计划拆解阶段的对比

| 维度 | Superpowers Writing Plans | OpenSpec Tasks |
|------|--------------------------|---------------|
| 粒度 | 每步 2-5 分钟（强制） | 无限制（用户定义） |
| 代码内容 | 完整代码 + 命令 + 预期输出 | 只有 checkbox 描述 |
| 目标读者 | subagent（执行机器） | 人类开发者 |
| Placeholder 容忍度 | 零容忍 | 未限定 |
| 自审要求 | 有（3 维度） | 无 |
| TDD 嵌入 | 每个 step 都是 Red-Green-Commit | 无内置 TDD |
| 与设计的关系 | plan 内置设计上下文 | tasks 和 design 是独立文件 |
