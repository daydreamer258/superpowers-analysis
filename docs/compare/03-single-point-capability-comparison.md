# OpenSpec vs Superpowers — 单点流程能力对比

这份文件深入对比两套框架在具体流程节点上的做法差异。

---

## 一、需求澄清阶段

### OpenSpec：Explore → Propose

```
用户输入 "我想做暗色模式"
          │
          ▼
/opsx:explore（可选）
  → Agent 探索代码库、分析现有架构
  → 交互式讨论方案，不产生正式 artifact
          │
          ▼
/opsx:propose add-dark-mode
  → 调用 openspec new change "add-dark-mode"
  → 按 schema 依赖顺序创建 artifacts:
    proposal.md → specs/ → design.md → tasks.md
  → 每个 artifact 通过 openspec instructions 获取模板和上下文
  → 用户可以控制：/opsx:continue（逐步）或 /opsx:ff（全自动）
```

**OpenSpec 的需求澄清特点：**
- Explore 和 Propose 是分离的——可以不探索直接提
- 没有"必须一个个问"的硬性约束
- proposal 模板固定（Why / What Changes / Capabilities / Impact）
- 结构偏向"决策文档"而非"对话引导"

### Superpowers：Brainstorming 9 步流程

```
用户输入 "我做暗色模式"
          │
          ▼
superpowers:brainstorming（强制，不可跳过）
  ├── Step 1: 探索项目上下文
  ├── Step 2: [可选] 提供视觉预览
  ├── Step 3: 逐个提问（一次一个问题，多选题优先）
  ├── Step 4: 提出 2-3 种方案 + trade-off
  ├── Step 5: 分章节展示设计，逐节征求确认
  ├── Step 6: 写入 design doc → commit
  ├── Step 7: 自审 spec
  ├── Step 8: 用户审核 spec
  └── Step 9: 转入 writing-plans
```

**Superpowers 的需求澄清特点：**
- 必须执行完整 9 步（硬 Gate）
- 交互式对话为主，一次一个问题
- 强调用户批准后才进入下一步
- 产出 design doc 而非 proposal + delta spec

### 对比

| | OpenSpec | Superpowers |
|--|----------|-------------|
| 是否必须？ | ❌ 可选（可跳过 explore） | ✅ 强制（硬 Gate） |
| 交互模式 | 批量问答或全自动（/ff） | 逐个问题、一次一个 |
| 产出格式 | proposal + design + spec + tasks | 单一 design doc |
| 方案探索 | explore 命令独立 | 内置 2-3 个方案对比 |
| 视觉辅助 | 无 | 有 visual companion |
| 用户批准 | 无显式 gate | 每个设计章节 + 最终 spec 都要批准 |

---

## 二、计划拆解阶段

### OpenSpec：Tasks.md（轻量 Checklist）

```markdown
# Tasks

## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
- [ ] 2.2 Add toggle to settings page
```

**特征：**
- 任务列表是 markdown 中的 checkbox
- 分组 + 层级编号（1.1, 1.2...）
- 没有代码，只有描述
- 不做分步粒度限制
- tasks.md 和 design.md 是独立文件

### Superpowers：Writing Plans（完整执行手册）

```markdown
### Task 1: ThemeContext

**Files:**
- Create: src/contexts/ThemeContext.tsx
- Test: src/contexts/__tests__/ThemeContext.test.tsx

- [ ] Step 1: Write the failing test
  ```typescript
  import { render, screen } from '@testing-library/react';
  import { ThemeProvider, useTheme } from '../ThemeContext';

  test('provides default light theme', () => {
    function TestComponent() {
      const { theme } = useTheme();
      return <div data-testid="theme">{theme}</div>;
    }

    render(
      <ThemeProvider>
        <TestComponent />
      </ThemeProvider>
    );

    expect(screen.getByTestId('theme').textContent).toBe('light');
  });
  ```

- [ ] Step 2: Run test to verify it fails
  Run: `npx jest src/contexts/__tests__/ThemeContext.test.tsx`
  Expected: FAIL - "ThemeContext is not defined"

- [ ] Step 3: Write minimal implementation
  [包含完整代码]

- [ ] Step 4: Run test to verify it passes
  Run: `npx jest ...`
  Expected: PASS

- [ ] Step 5: Commit
  git add ... && git commit -m "..."
```

**特征：**
- 每个 step 2-5 分钟
- 包含完整代码和精确命令
- 命令后跟随预期输出
- 零 placeholder（"TBD" / "类似 Task N" 都是失败）
- 自审：spec 覆盖检查 + placeholder 扫描 + 类型一致性检查

### 对比

| | OpenSpec Tasks | Superpowers Plan |
|--|---------------|-----------------|
| 粒度 | 无限制 | 每步 2-5 分钟 |
| 代码内容 | 无，只有描述 | 完整代码 + 命令 + 预期输出 |
| 读者 | 人类开发者 | subagent（执行机器） |
| Placeholder 容忍度 | 未限定 | 零容忍 |
| 自审要求 | 无 | 有（3 维度） |
| 与 design 的关系 | 独立文件 | plan 内置设计上下文 |
| TDD 嵌入 | 无 | 每个 step 都是 Red-Green-Commit |

---

## 三、代码实现阶段

这是差异最大的环节。

### OpenSpec：/opsx:apply（单 Agent 顺序实现）

```
1. openspec status → 了解 schema 和状态
2. openspec instructions apply → 获取上下文文件列表
3. 读取 proposal + specs + design + tasks
4. 展示进度
5. 循环执行 task:
   - 显示正在处理的 task
   - 做代码改动
   - 标记 task 为 [x]
6. 全部完成 → 提示 archive
   如果卡住 → 暂停 + 等待用户指导
```

**OpenSpec apply 的核心逻辑：用户还在 loop 里。** Agent 只是辅助执行，用户可以随时中断、切换 change、重来。

### Superpowers：Subagent-Driven Development（多层 Agent 编排）

```
Controller (父 session):
├── 读取 plan，提取所有 task
├── 顺序 dispatch implementer subagent（每个 task 一个）
│   每个 implementer:
│   ├── 收到完整 task 文本（paste，不读文件）
│   ├── 收到场景上下文（独立 session，不继承父历史）
│   ├── 有问题现在问（Before You Begin gate）
│   ├── 实现 → 测试 → 验证 → commit → 自审
│   └── 汇报: DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT
│
├── 每个 task 后：
│   ├── D❶ispatch spec compliance reviewer
│   │   "CRITICAL: Do Not Trust the Report — verify independently"
│   │   检查: 遗漏 / 冗余 / 误解
│   │   ❌ → implementer 修复 → 重新 review
│   │
│   └── spec review ✅ 后：
│       D❷ispatch code quality reviewer
│       返回: Strengths + Issues (Critical/Important/Minor)
│       ❌ → implementer 修复 → 重新 review
│       ✅ → 下一个 task
│
└── 所有 task 完成后 → final code review → finishing
```

**Superpowers 的核心逻辑：用户被移出执行 loop。** 用户只审批 spec + plan，然后看 agent 跑。

### 对比

| | OpenSpec apply | Superpowers subagent-driven |
|--|---------------|---------------------------|
| 执行者 | 当前 agent | 每个 task 一个 fresh subagent |
| 上下文隔离 | 否（共享 session） | 是（不继承父 session） |
| 独立 review | 否 | 是（两阶段 review gate） |
| 用户干预点 | 每个 task 期间都可 | 只在遇到 BLOCKED |
| TDD | 无内置规则 | 强制 Red-Green-Refactor |
| 模型选择 | 固定 | 按 task 复杂度分级 |
| 成本 | 低 | 高（每个 task: 1 implementer + 2 reviewers） |
| 失败处理 | 暂停 → 等用户 | BLOCKED → 更强模型/更小 task/报 human |
| "跳过"行为 | 用户自主决策 | 被规则阻止（Red Flags） |

---

## 四、验证阶段

### OpenSpec：/opsx:verify（三维度分析报告）

```
1. openspec status → 了解 schema
2. openspec instructions apply → 获取 context files
3. 三维度分析:

   Completeness（完整性）
   ├── Task 完成度：checkbox 统计
   └── Spec 覆盖度：需求 → 代码映射

   Correctness（正确性）
   ├── 需求实现映射：spec 需求 vs 实际代码
   └── Scenario 覆盖：spec 场景 vs 测试/代码

   Coherence（一致性）
   ├── Design 遵循度：设计决策 vs 实现
   └── 代码模式一致性：命名/结构/风格

4. 生成报告：
   CRITICAL → 必须修复
   WARNING → 应该修复
   SUGGESTION → 可选
```

**特点：**
- 报告式验证，不阻止 archive
- 渐进降级：有 task → 只查 task；有 task+spec → 查完整性和正确性；全有 → 查所有三维度
- 目标是"给你信息让你决策"

### Superpowers：Verification Before Completion（铁律）

```
任何声称"完成"之前：

1. IDENTIFY — 什么命令能证明？
2. RUN — 执行完整命令（必须新鲜运行）
3. READ — 读完整输出，检查退出码
4. VERIFY — 输出确认声明？

禁止的表达（在验证前）：
❌ "应该工作了" "完成了！" "看起来正确"
❌ 任何暗示成功但没有验证证据的表述
```

**特点：**
- 行为铁律，不是可选步骤
- 与 TDD 铁律配合：TDD 管开发中，Verification 管交接时
- "Claims without verification is dishonesty, not efficiency."

### 对比

| | OpenSpec verify | Superpowers verification |
|--|----------------|-------------------------|
| 性质 | 可选 command | 不可协商的铁律 |
| 触发时机 | 用户主动调用 | 任何"完成"声明时 |
| 深度 | 三维度结构化报告 | 命令级证据（"run it, show output"） |
| 阻止能力 | 不阻止 archive | 阻止任何虚假完成声明 |
| 自动执行 | 用户发起 | agent 被规则约束自动执行 |

---

## 五、完成和归档阶段

### OpenSpec：Archive（Delta Merge + 历史保留）

```
/opsx:archive
  ├── 检查 delta specs 是否已 sync
  ├── 如果未 sync → 提示 sync
  ├── 执行：
  │   ├── ADDED 需求 → append 到主 spec
  │   ├── MODIFIED 需求 → replace 主 spec 中的对应项
  │   ├── REMOVED 需求 → delete from 主 spec
  │   └── 整个 change 文件夹 → archive/YYYY-MM-DD-<name>/
  └── 主 spec 现在是更新后的 source of truth
```

**核心价值：spec 是活文档，随开发迭代演进。**

### Superpowers：Finishing（4 选项操作菜单）

```
superpowers:finishing-a-development-branch
  ├── Step 1: 验证测试（必须通过）
  ├── Step 2: 检测环境（normal/worktree/detached HEAD）
  ├── Step 3: 确定 base branch
  ├── Step 4: 呈现选项
  │   1. 本地合并回去
  │   2. Push + 创建 PR
  │   3. 保留分支
  │   4. 丢弃所有（需输入 "discard" 确认）
  └── Step 5-6: 执行 + 清理 worktree
```

**核心价值：安全完成——不遗漏、不毁坏、不遗留垃圾 worktree。**

### 对比

| | OpenSpec archive | Superpowers finishing |
|--|-----------------|----------------------|
| 做什么 | 合并 delta spec + 归档 change | 合并代码 + 清理环境 |
| spec 管理 | ✅ 核心功能（delta merge） | ❌ 不管 spec 生命周期 |
| 代码合并 | ❌ 不管代码 | ✅ 核心功能（merge/PR/discard） |
| worktree | 不支持 | 完整支持 |
| spec 作为 source of truth | ✅ 核心设计 | ❌ 不跟踪 spec 演进 |

---

## 六、总结矩阵

```
                     OpenSpec          Superpowers
需求澄清    ──────── 轻量、可选 ─────── 强制、深度交互
计划拆解    ──────── checklist ─────── 完整执行手册（含代码）
代码实现    ──────── 单 agent 辅助 ──── 多层 subagent 编排 + 双 review
TDD        ──────── 无 ────────────── 铁律
验证       ──────── 可选报告 ──────── 铁律
完成后     ──────── delta merge ───── merge/PR + 清理
spec 管理  ──────── 核心功能 ──────── 不管
代码管理   ──────── 不管 ──────────── 核心功能
用户参与度 ──────── 全程可控 ──────── 审批 spec/plan 后放手
agent 自主 ──────── 低 ───────────── 高（受严格规则约束）
成本       ──────── 低 ───────────── 高（多层 subagent）
学习曲线   ──────── 低 ───────────── 中
```

**一句话：OpenSpec 管理"改什么"，Superpowers 管理"怎么改得好"。**
