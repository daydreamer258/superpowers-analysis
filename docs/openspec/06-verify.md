# OpenSpec 06 — Verify: 三维度验证

## /opsx:verify 是什么

一个**可选**的验证 command，在 archive 之前检查实现是否匹配 spec 中的约定。

`src/core/templates/workflows/verify-change.ts` 中的 `getVerifyChangeSkillTemplate()` 定义了完整逻辑。

## 完整流程

### 步骤 1-2: 选择 change + 检查状态

```bash
openspec status --change "add-dark-mode" --json
```

### 步骤 3: 获取上下文文件

```bash
openspec instructions apply --change "add-dark-mode" --json
```

读取所有 contextFiles。

### 步骤 4-7: 三维度验证

```
Verification Report
├── 1. Completeness（完整性）
├── 2. Correctness（正确性）
└── 3. Coherence（一致性）
```

---

## 维度 1: Completeness（完整性）

**Task completion:**
- 读取 tasks.md
- 解析 checkbox: `[ ]` (未完成) vs `[x]` (已完成)
- 统计完成率
- 未完成 → CRITICAL issue

**Spec coverage:**
- 从 delta specs 提取所有 `### Requirement:`
- 在代码库中搜索相关关键词
- 评估实现是否存在
- 未实现 → CRITICAL issue

## 维度 2: Correctness（正确性）

**Requirement implementation mapping:**
- 对 specs 中的每个 requirement
- 在代码库中搜索实现证据
- 评估实现是否匹配 requirement 意图
- 偏离 → WARNING

**Scenario coverage:**
- 对每个 `#### Scenario:`
- 检查代码中是否处理了条件
- 检查是否有测试覆盖场景
- 未覆盖 → WARNING

## 维度 3: Coherence（一致性）

**Design adherence:**
- 从 design.md 提取关键决策（"Decision:", "Approach:", "Architecture:"）
- 验证实现是否遵循
- 矛盾 → WARNING

**Code pattern consistency:**
- 审查新代码与项目模式的用一致性
- 检查文件命名、目录结构、编码风格
- 显著偏离 → SUGGESTION

---

## 报告格式

```markdown
## Verification Report: add-dark-mode

### Summary
| Dimension    | Status           |
|--------------|------------------|
| Completeness | 12/12 tasks, 3/3 reqs |
| Correctness  | 3/3 reqs covered |
| Coherence    | Followed         |

### Issues

**CRITICAL** (Must fix before archive):
- [如果有]

**WARNING** (Should fix):
- [如果有]

**SUGGESTION** (Nice to fix):
- [如果有]

### Final Assessment
All checks passed. Ready for archive.
```

## 渐进降级

```
只有 tasks.md     → 只查 task completion
tasks + specs     → 查 completeness + correctness
全部 artifact     → 查所有三个维度
```

## 验证的启发式原则

- **Completeness:** 聚焦于客观 checklist（checkbox, requirements list）
- **Correctness:** 关键词搜索 + 文件路径分析 + 合理推断，不需要完美元确定性
- **Coherence:** 找明显的不一致，不要过度纠缠风格
- **False Positive:** 不确定时，倾向于 SUGGESTION 而不是 WARNING，WARNING 而不是 CRITICAL
- **Actionability:** 每个 issue 必须有具体的可执行建议，带 file:line 引用

## 与 Superpowers 验证机制的对比

| | OpenSpec Verify | Superpowers Verification |
|--|----------------|-------------------------|
| 性质 | 可选 command | 不可协商的铁律 |
| 触发方式 | 用户主动调用 | 任何"完成"声明时自动触发 |
| 深度 | 三维度结构化报告 | 命令级证据（"run it, show output"） |
| 是否阻止 | 不阻止 archive | 阻止虚假完成声明 |
| 检查粒度 | Requirement / scenario / design decision 级别 | 测试通过 / 命令成功 级别 |
