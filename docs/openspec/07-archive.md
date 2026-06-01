# OpenSpec 07 — Archive: 归档与 Spec 生命周期管理

## Archive 是什么

Archive 是 OpenSpec 工作流的**终点**——它将一次 change 的 delta specs 合并到主 specs，并将 change 文件夹移动到 archive 目录保留历史。

`src/core/templates/workflows/archive-change.ts` 和 `src/core/archive.ts` 定义了完整逻辑。

## 完整流程

```
/opsx:archive
     │
     ├── 1. 检查 delta specs 是否已 sync
     │    ├── 未 sync → 提示 sync
     │    └── 已 sync → 继续
     │
     ├── 2. 执行 delta merge:
     │    ├── ADDED requirements    → append 到对应主 spec
     │    ├── MODIFIED requirements → replace 主 spec 中的对应项
     │    └── REMOVED requirements  → delete from 主 spec
     │
     ├── 3. 移动 change 文件夹
     │    openspec/changes/<name>/
     │    → openspec/changes/archive/YYYY-MM-DD-<name>/
     │
     └── 4. 主 spec 现在是更新后的 source of truth
```

## Delta Merge 详解

### ADDED Requirements

```markdown
# 在 change 的 delta spec 中:
## ADDED Requirements
### Requirement: Two-Factor Authentication
...

# 合并后 → append 到 openspec/specs/auth/spec.md 末尾
```

### MODIFIED Requirements

```markdown
# 在 change 的 delta spec 中:
## MODIFIED Requirements
### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes.
(Previously: 30 minutes)

# 合并后 → openspec/specs/auth/spec.md 中的对应 requirement 被替换
```

### REMOVED Requirements

```markdown
# 在 change 的 delta spec 中:
## REMOVED Requirements
### Requirement: Remember Me
(Deprecated in favor of 2FA)

# 合并后 → 从 openspec/specs/auth/spec.md 中删除该 requirement
```

## 归档前后对比

```
Before archive:

openspec/
├── specs/
│   └── auth/
│       └── spec.md          ← 不含 2FA
└── changes/
    └── add-2fa/
        ├── proposal.md
        ├── design.md
        ├── tasks.md
        └── specs/
            └── auth/
                └── spec.md  ← delta

After archive:

openspec/
├── specs/
│   └── auth/
│       └── spec.md          ← 现在含 2FA requirements
└── changes/
    └── archive/
        └── 2025-01-24-add-2fa/
            ├── proposal.md  ← 完整保留
            ├── design.md
            ├── tasks.md
            └── specs/
                └── auth/
                    └── spec.md
```

## Archive 的价值

1. **干净的状态。** `changes/` 只显示进行中的工作。完成的移走。
2. **审计追踪。** archive 保留了每次 change 的完整上下文——不只改了什么，还有为什么、怎么做的、做了什么工作。
3. **Spec 演进。** Specs 随着每次 archive 有机增长，构建出全面的规范。

## 代码层面: `src/core/archive.ts`

archive 模块负责：
- 解析 delta spec 的三个 section
- 定位对应的主 spec 文件
- 执行 merge（append / replace / delete）
- 移动 change 文件夹到 archive/
- 验证 merge 后的主 spec 结构完整性

## Bulk Archive

`/opsx:bulk-archive` 支持一次归档多个 change：

```
1. 列出所有 task 已完成的 change
2. 检查 spec 冲突（多个 change 修改同一个主 spec）
3. 按时间顺序合并
4. 逐一归档
```

## 与 Superpowers Finishing 的对比

| | OpenSpec Archive | Superpowers Finishing |
|--|-----------------|----------------------|
| 做什么 | 合并 delta spec + 移动文件夹 | 合并代码 + 清理环境 |
| spec 管理 | ✅ 核心功能 | ❌ 不管 |
| 代码合并 | ❌ 不管 | ✅ 核心（merge/PR/discard） |
| 历史保留 | ✅ 完整保留 in archive/ | 通过 git history |
| worktree 管理 | ❌ 不支持 | ✅ 完整支持 |
| spec 作为 source of truth | ✅ 核心设计 | ❌ 不跟踪 spec 演进 |
