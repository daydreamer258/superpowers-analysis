# 06 - Finishing & Git Worktree

## Finishing a Development Branch

**文件:** `skills/finishing-a-development-branch/SKILL.md`

### 流程

```
Step 1: 验证测试 → Step 2: 检测环境 → Step 3: 确定 base branch
→ Step 4: 呈现选项 → Step 5: 执行选择 → Step 6: 清理 worktree
```

### 环境检测

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

| 状态 | 判断条件 | 菜单 | 清理 |
|------|---------|------|------|
| Normal repo | `GIT_DIR == GIT_COMMON` | 4 个选项 | 无 worktree 可清理 |
| 命名分支 worktree | `GIT_DIR != GIT_COMMON`, 有分支名 | 4 个选项 | 来源检查 |
| Detached HEAD | `GIT_DIR != GIT_COMMON`, 无分支名 | 3 个选项 | 不清理（外部管理） |

### 4 个选项（Normal/命名分支）

| 选项 | 合并 | 推送 | 保留 worktree | 清理分支 |
|-----|------|------|--------------|---------|
| 1. 本地合并回去 | ✅ | - | - | ✅ |
| 2. Push + 创建 PR | - | ✅ | ✅（需要迭代） | - |
| 3. 原样保留 | - | - | ✅ | - |
| 4. 丢弃所有 | - | - | - | ✅ (force) |

**选项 4 需要用户键入 `discard` 确认后才能执行。**

### Worktree 清理规则

只清理 Superpowers 自己创建的 worktree（路径在 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下）。

如果 worktree 是平台（harness）创建的 → 不清理，让平台自己管。

清理时必须先 cd 到主 repo root，防止在 worktree 内部执行 `git worktree remove`。

---

## Using Git Worktrees

**文件:** `skills/using-git-worktrees/SKILL.md`

### 核心原则

> 先检测已有的隔离。再尝试原生工具。最后才 fallback 到手动 git worktree。永远不要和 harness 对着干。

### 流程

#### Step 0: 检测已有隔离

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

- `GIT_DIR != GIT_COMMON` (且不是 submodule) → 已在 worktree 中 → 跳过创建
- **Submodule 守卫**：`GIT_DIR != GIT_COMMON` 在 submodule 中也成立 → 用 `git rev-parse --show-superproject-working-tree` 排除

#### Step 1a: 原生工具（首选）

平台有 `EnterWorktree` / `WorktreeCreate` / `/worktree` 等原生工具？直接用。原生工具会处理目录位置、分支创建和清理。

#### Step 1b: 手动 Git Worktree（后备）

仅在没有原生工具时使用。

**目录优先级：**
1. 用户指令中声明的偏好
2. 已有的 `.worktrees/` (隐藏目录，优先)
3. 已有的 `worktrees/` (备选)
4. 已有的 `~/.config/superpowers/worktrees/<project>/`
5. 默认 `.worktrees/`

**安全验证：** 项目本地目录必须先通过 `git check-ignore` 验证被忽略。如果没被忽略，先加入 `.gitignore` 并 commit。

#### Step 3: 项目 Setup

自动检测并安装依赖：
```bash
# Node.js
if [ -f package.json ]; then npm install; fi
# Rust
if [ -f Cargo.toml ]; then cargo build; fi
# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
# Go
if [ -f go.mod ]; then go mod download; fi
```

#### Step 4: 验证清洁基线

运行测试套件确保 workspace 开始时一切正常。失败了就问用户：继续还是先修？

---

## Sandbox 后备

如果 `git worktree add` 因为权限错误失败（sandbox 拒绝），告诉用户 sandbox 阻止了 worktree 创建，然后在当前目录继续工作。

---

## 与 OpenSpec 完成/归档阶段的对比

| 维度 | Superpowers Finishing | OpenSpec Archive |
|------|----------------------|-----------------|
| 做什么 | 合并代码 + 清理环境 | 合并 delta spec + 移动文件夹 |
| spec 管理 | ❌ 不管 spec 生命周期 | ✅ 核心功能（delta merge） |
| 代码合并 | ✅ 核心（merge/PR/discard） | ❌ 不管代码 |
| 历史保留 | 通过 git history | 完整保留在 archive/ 目录中 |
| worktree 管理 | ✅ 完整支持 | ❌ 不支持 |
| 选项数量 | 4 个明确选项 | 2 步（sync → archive） |
