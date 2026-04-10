---
name: using-git-worktrees
description: 在开始需要与当前工作区隔离的功能开发，或在执行实现计划之前使用——创建隔离的 git worktree，含智能目录选择与安全检查
---

# 使用 Git Worktree

## 概述

Git worktree 在共享同一仓库的前提下创建隔离工作区，可同时处理多个分支而无需切换。

**核心原则：** 系统化目录选择 + 安全验证 = 可靠隔离。

**开始时宣告：**「我正在使用 using-git-worktrees 技能来建立隔离工作区。」

## 目录选择流程

按以下优先级：

### 1. 检查已有目录

```bash
# 按优先级检查
ls -d .worktrees 2>/dev/null     # 优先（隐藏）
ls -d worktrees 2>/dev/null      # 备选
```

**若存在：** 使用该目录。若两者都存在，`.worktrees` 优先。

### 2. 查看 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**若写明偏好：** 直接使用，不必再问。

### 3. 询问用户

若既无目录也无 CLAUDE.md 偏好：

```
未找到 worktree 目录。应在何处创建 worktree？

1. .worktrees/（项目内、隐藏）
2. ~/.config/superpowers/worktrees/<项目名>/（全局位置）

你更倾向哪一种？
```

## 安全验证

### 项目本地目录（.worktrees 或 worktrees）

**创建 worktree 前必须确认目录已被忽略：**

```bash
# 检查目录是否被忽略（尊重本地、全局与系统 gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**若未被忽略：**

按 Jesse 的规则「坏了就立刻修」：
1. 在 .gitignore 中加入合适规则  
2. 提交该改动  
3. 再继续创建 worktree  

**为何关键：** 避免误将 worktree 内容提交进仓库。

### 全局目录（~/.config/superpowers/worktrees）

完全在项目外，无需 .gitignore 验证。

## 创建步骤

### 1. 检测项目名

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# 确定完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 新建分支并创建 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目初始化

自动检测并运行合适步骤：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净基线

运行测试，确保 worktree 从干净状态开始：

```bash
# 示例——使用项目对应的命令
npm test
cargo test
pytest
go test ./...
```

**若测试失败：** 报告失败，询问是继续还是排查。

**若测试通过：** 报告就绪。

### 5. 报告位置

```
Worktree 已就绪：<完整路径>
测试通过（<N> 个测试，0 失败）
可以开始实现 <功能名>
```

## 速查

| 情况 | 操作 |
|-----------|--------|
| 存在 `.worktrees/` | 使用（并验证已忽略） |
| 存在 `worktrees/` | 使用（并验证已忽略） |
| 两者都有 | 使用 `.worktrees/` |
| 都没有 | 查 CLAUDE.md → 问用户 |
| 目录未被忽略 | 加入 .gitignore 并提交 |
| 基线测试失败 | 报告失败并询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过忽略验证

- **问题：** worktree 内容被跟踪，污染 git status  
- **修复：** 创建项目本地 worktree 前始终 `git check-ignore`  

### 假定目录位置

- **问题：** 不一致，违背项目约定  
- **修复：** 优先级：已有 > CLAUDE.md > 询问  

### 测试失败仍继续

- **问题：** 无法区分新 bug 与既有问题  
- **修复：** 报告失败，取得明确许可后再继续  

### 写死安装命令

- **问题：** 换工具链就坏  
- **修复：** 根据项目文件自动检测（package.json 等）  

## 示例工作流

```
你：我正在使用 using-git-worktrees 技能来建立隔离工作区。

[检查 .worktrees/ - 存在]
[验证已忽略 - git check-ignore 确认 .worktrees/ 被忽略]
[创建 worktree：git worktree add .worktrees/auth -b feature/auth]
[运行 npm install]
[运行 npm test - 47 通过]

Worktree 已就绪：/Users/jesse/myproject/.worktrees/auth
测试通过（47 个测试，0 失败）
可以开始实现认证功能
```

## 危险信号

**绝不：**
- 未验证忽略状态就创建（项目本地）worktree  
- 跳过基线测试验证  
- 未询问就在测试失败时继续  
- 位置有歧义时自作主张  
- 跳过 CLAUDE.md 检查  

**务必：**
- 目录优先级：已有 > CLAUDE.md > 询问  
- 项目本地须验证目录已忽略  
- 自动检测并运行项目初始化  
- 验证测试基线干净  

## 衔接

**会调用本技能的有：**
- **brainstorming**（第 4 阶段）——设计批准且进入实现时 **必选**  
- **subagent-driven-development** ——执行任何任务前 **必选**  
- **executing-plans** ——执行任何任务前 **必选**  
- 任何需要隔离工作区的技能  

**常与之配合：**
- **finishing-a-development-branch** ——工作完成后 **必选** 清理  
