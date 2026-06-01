# 03 - Brainstorming 需求澄清流程

**文件:** `skills/brainstorming/SKILL.md`

## 硬性 Gate（不可绕过）

```
<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any
project, or take any implementation action until you have presented a
design and the user has approved it.
This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>
```

## Anti-Pattern："这太简单了不需要设计"

Superpowers 明确禁止这种想法。**任何项目都要走完整流程。** "简单"项目是未经验证的假设造成最多浪费的地方。设计可以短（简单项目几句话即可），但必须展示并获得批准。

## 9 步检查清单

1. **探索项目上下文** — 检查文件、文档、最近提交
2. **提供视觉预览**（可选）— 当话题涉及视觉内容时，单发一条消息询问是否要浏览器 mockup
3. **逐个提问** — 一次一个问题，理解目的/约束/成功标准
4. **提出 2-3 种方案** — 带 trade-off 和推荐选项
5. **分章节展示设计** — 逐节征求确认
6. **写设计文档** → `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` → git commit
7. **自审设计文档** — 扫 TBD/TODO、矛盾、范围、歧义
8. **用户审核 spec** — 等待用户确认后再进入下一步
9. **转入 implementation** — 调用 writing-plans skill（禁止跳到任何其他实现 skill）

## 核心设计原则

- **一次一个问题** — 不给用户认知负担
- **多选题优先** — 降低回答门槛，但不排斥开放式问题
- **YAGNI** — 无情地删除不必要的功能
- **探索替代方案** — 收敛前总要摆出 2-3 种方案
- **增量验证** — 展示设计 → 获取批准 → 再继续
- **灵活回退** — 发现不对就回去澄清

## 设计文档规范

路径: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

**自审（写完后立即检查）：**
1. **Placeholder 扫描** — 有没有 TBD、TODO、不完整的章节？
2. **内部一致性** — 章节之间有矛盾吗？架构和描述匹配吗？
3. **范围检查** — 是否聚焦到可以被一个 plan 覆盖？
4. **歧义检查** — 有没有需求可以被两种方式理解？选一个并明确下来。

发现问题就内联修复，不需要重新审查。

## 多子系统项目处理

如果请求描述了多个独立子系统（如"做一个有聊天、文件存储、计费、分析功能的平台"），立即标记：
- 不要花时间细化这种大项目的细节，先拆解
- 拆成独立子项目，按顺序逐个 brainstorming
- 每个子项目独立走 spec → plan → implementation 周期

## 设计中的代码组织

- 将系统拆分成小而精的单元，每个单元有清晰的职责、定义良好的接口
- **能独立理解和测试**
- 有人能不看内部就理解一个单元做什么吗？能在不改消费者的情况下改内部吗？
- 文件为单元服务——如果一个文件太大，往往是它管了太多事

## 在既有代码库中工作

- 先探索现有结构，再提改动
- 如果现有代码有问题影响你的工作（文件太大、边界不清、职责混乱），作为设计的一部分进行针对性改进
- 不要提无关重构，专注服务于当前目标

## 关键终止状态

```
brainstorming 结束后 → 唯一合法下一步是 writing-plans
禁止直接跳到 frontend-design、mcp-builder 或任何其他实现 skill
```
