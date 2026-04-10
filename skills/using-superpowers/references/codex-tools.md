# Codex 工具映射

技能使用 Claude Code 的工具名。遇到下列名称时，请使用你所在平台的等价工具：

| 技能中的名称 | Codex 等价 |
|-----------------|------------------|
| `Task`（派发子智能体） | `spawn_agent`（见下文「命名智能体派发」） |
| 多次 `Task`（并行） | 多次 `spawn_agent` |
| 任务返回结果 | `wait` |
| 任务自动结束 | `close_agent` 释放槽位 |
| `TodoWrite`（待办） | `update_plan` |
| `Skill`（调用技能） | 技能原生加载——直接遵循说明 |
| `Read`、`Write`、`Edit` | 使用原生文件工具 |
| `Bash` | 使用原生 shell 工具 |

## 子智能体派发需要多智能体支持

在 Codex 配置 `~/.codex/config.toml` 中加入：

```toml
[features]
multi_agent = true
```

这样会启用 `spawn_agent`、`wait`、`close_agent`，供 `dispatching-parallel-agents`、`subagent-driven-development` 等技能使用。

## 命名智能体派发

Claude Code 技能会引用如 `superpowers:code-reviewer` 这类命名类型。  
Codex 没有命名注册表——`spawn_agent` 用内置角色（`default`、`explorer`、`worker`）创建通用智能体。

当技能要求派发命名类型时：

1. 找到该智能体的提示文件（如 `agents/code-reviewer.md` 或技能内 `code-quality-reviewer-prompt.md`）  
2. 读取提示内容  
3. 填写模板占位符（`{BASE_SHA}`、`{WHAT_WAS_IMPLEMENTED}` 等）  
4. 以填充后的内容作为 `message` 派发 `worker` 智能体  

| 技能指令 | Codex 等价 |
|-------------------|------------------|
| `Task tool (superpowers:code-reviewer)` | `spawn_agent(agent_type="worker", message=...)`，内容为 `code-reviewer.md` |
| `Task tool (general-purpose)` 且内联提示 | `spawn_agent(message=...)`，同一提示 |

### 消息框架

`message` 是用户层输入，不是系统提示。为最大化遵循度，建议结构：

```
你的任务是执行下列内容。请严格遵循以下说明。

<agent-instructions>
[来自智能体 .md 的已填充内容]
</agent-instructions>

立即执行。仅按上述说明中规定的格式输出结构化结果。
```

- 用任务委托框架（「你的任务是……」）而非人设（「你是……」）  
- 用 XML 标签包裹说明——模型会把标签块视为权威  
- 以明确执行指令结尾，避免把说明总结掉  

### 何时可去掉该变通

该做法用于弥补 Codex 插件系统尚未在 `plugin.json` 支持 `agents` 字段。当 `RawPluginManifest` 增加 `agents` 后，插件可像现有 `skills/` 一样 symlink 到 `agents/`，技能即可直接派发命名类型。

## 环境检测

创建 worktree 或收尾分支的技能应先**只读**用 git 探测环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已在链接的 worktree 中（跳过创建）  
- `BRANCH` 为空 → 分离 HEAD（沙箱中无法分支/推送/开 PR）  

详见 `using-git-worktrees` 第 0 步与 `finishing-a-development-branch` 第 1 步如何解读这些信号。

## Codex App 收尾

若沙箱阻止分支/推送（外部托管 worktree 中分离 HEAD），智能体应提交全部工作并提示用户使用 App 原生操作：

- **「Create branch」** — 命名分支后通过 App UI 提交/推送/开 PR  
- **「Hand off to local」** — 将工作转到用户本地检出  

智能体仍可运行测试、暂存文件，并输出建议分支名、提交说明与 PR 描述供用户复制。
