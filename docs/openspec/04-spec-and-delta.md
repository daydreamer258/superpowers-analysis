# OpenSpec 04 — Spec 系统与 Delta 机制

## Spec 是什么

Spec 是 OpenSpec 的核心概念——**描述系统行为的结构化文档**。

```
openspec/specs/
├── auth/
│   └── spec.md           # 认证行为
├── payments/
│   └── spec.md           # 支付处理
├── notifications/
│   └── spec.md           # 通知系统
└── ui/
    └── spec.md           # UI 行为
```

**按领域组织：** 按功能域、组件或限界上下文分组。

## Spec 格式

```markdown
# Auth Specification

## Purpose
Authentication and session management for the application.

## Requirements

### Requirement: User Authentication
The system SHALL issue a JWT token upon successful login.

#### Scenario: Valid credentials
- GIVEN a user with valid credentials
- WHEN the user submits login form
- THEN a JWT token is returned
- AND the user is redirected to dashboard

#### Scenario: Invalid credentials
- GIVEN invalid credentials
- WHEN the user submits login form
- THEN an error message is displayed
- AND no token is issued

### Requirement: Session Expiration
The system MUST expire sessions after 30 minutes of inactivity.

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 30 minutes pass without activity
- THEN the session is invalidated
- AND the user must re-authenticate
```

**关键元素：**

| 元素 | 目的 |
|------|------|
| `## Purpose` | 该 spec 域的高层描述 |
| `### Requirement:` | 系统必须具有的具体行为 |
| `#### Scenario:` | 需求在实际中的具体示例 |
| SHALL/MUST/SHOULD | RFC 2119 关键词，表示需求强度 |

## Spec 是什么（和不是）

**Spec 是行为契约，不是实现计划。**

- ✅ 可观察的行为、输入输出、错误条件、外部约束
- ❌ 内部类/函数名、框架选择、逐步实现细节

**快速测试：** 如果实现可以变而不改变外部可见行为，那就不该放在 spec 里。

## Delta Spec：OpenSpec 的核心创新

Delta spec 描述的是**变化了什么**，而不是重述整个 spec。这是 OpenSpec 能够做到 "brownfield-first" 的关键。

### Delta 格式

```markdown
# Delta for Auth

## ADDED Requirements
### Requirement: Two-Factor Authentication
The system MUST support TOTP-based 2FA.

#### Scenario: 2FA enrollment
- GIVEN a user without 2FA enabled
- WHEN the user enables 2FA in settings
- THEN a QR code is displayed for authenticator app setup
- AND the user must verify with a code before activation

## MODIFIED Requirements
### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

## REMOVED Requirements
### Requirement: Remember Me
(Deprecated in favor of 2FA. Users should re-authenticate each session.)
```

### Delta 的三个 Section

| Section | 含义 | Archive 时的操作 |
|---------|------|-----------------|
| `## ADDED Requirements` | 新行为 | append 到主 spec |
| `## MODIFIED Requirements` | 变化的行为 | replace 主 spec 中的对应项 |
| `## REMOVED Requirements` | 废弃的行为 | delete from 主 spec |

### 为什么用 Delta 而不是全量 Spec

1. **清晰。** Delta 精确展示什么在变。读全量 spec 需要脑内 diff。
2. **避免冲突。** 两个 change 可以 touch 同一个 spec 文件而不冲突，只要它们改的是不同的 requirement。
3. **审查高效。** 审查者只看变化，不是不变的上下文。
4. **天然适配既有代码库。** 大多数工作修改的既有行为——delta 让修改成为一等公民。

## Archive 时的 Delta Merge

```
Before archive:

openspec/
├── specs/
│   └── auth/spec.md        ← 原始 spec（不含 2FA）
└── changes/
    └── add-2fa/
        └── specs/auth/spec.md  ← delta（含 ADDED 2FA）

After archive:

openspec/
├── specs/
│   └── auth/spec.md        ← 现在包含 2FA requirements
└── changes/
    └── archive/
        └── 2025-01-24-add-2fa/  ← 完整保留（历史）
```

## Spec 的轻重分级

OpenSpec 提倡 "progressive rigor"：

**Lite spec（默认）：**
- 简洁的行为优先 requirement
- 清晰的范围和不在范围内的
- 几个具体的验收检查

**Full spec（高风险场景）：**
- 跨团队/跨 repo 变更
- API/合约变更、迁移、安全/隐私问题
- 歧义可能导致昂贵返工

多数 change 应保持 Lite 模式。

## 代码层面：Spec 解析

`src/core/parsers/` 负责解析 spec 文件：

- `markdown-parser.ts` — 解析 Markdown 结构
- `requirement-blocks.ts` — 提取 requirement + scenario
- `spec-structure.ts` — 验证 spec 结构完整性
- `change-parser.ts` — 解析 delta spec section

`src/core/validation/` 负责验证，包括：
- requirement 是否都有 scenario
- scenario 格式是否正确
- 命名是否一致

## 与 Superpowers 的 Spec 对比

| | OpenSpec Spec | Superpowers Design Doc |
|--|--------------|----------------------|
| 格式 | 结构化（Requirements + Scenarios） | 自由格式 Markdown |
| 是否增量 | 是（Delta Spec） | 否 |
| 是否合并到主文档 | 是（Archive merge） | 否（独立文件） |
| 作为 source of truth | ✅ 核心设计 | ❌ 是工作文档 |
| RFC 2119 关键词 | 使用 | 未强制 |
