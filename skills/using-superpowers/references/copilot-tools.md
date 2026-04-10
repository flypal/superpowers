# Copilot CLI 工具映射

技能使用 Claude Code 的工具名。遇到下列名称时，请使用你所在平台的等价工具：

| 技能中的名称 | Copilot CLI 等价 |
|-----------------|----------------------|
| `Read`（读文件） | `view` |
| `Write`（新建文件） | `create` |
| `Edit`（编辑文件） | `edit` |
| `Bash`（执行命令） | `bash` |
| `Grep`（搜内容） | `grep` |
| `Glob`（按名搜文件） | `glob` |
| `Skill`（调用技能） | `skill` |
| `WebFetch` | `web_fetch` |
| `Task`（派发子智能体） | `task`（见下文「智能体类型」） |
| 多次 `Task`（并行） | 多次 `task` |
| 任务状态/输出 | `read_agent`、`list_agents` |
| `TodoWrite`（待办） | 用 `sql` 与内置 `todos` 表 |
| `WebSearch` | 无直接等价——用搜索引擎 URL 配合 `web_fetch` |
| `EnterPlanMode` / `ExitPlanMode` | 无等价——留在主会话 |

## 智能体类型

Copilot CLI 的 `task` 接受 `agent_type` 参数：

| Claude Code 智能体 | Copilot CLI 等价 |
|-------------------|----------------------|
| `general-purpose` | `"general-purpose"` |
| `Explore` | `"explore"` |
| 命名插件智能体（如 `superpowers:code-reviewer`） | 从已安装插件自动发现 |

## 异步 Shell 会话

Copilot CLI 支持持久异步 shell，Claude Code 无直接对应：

| 工具 | 用途 |
|------|--------|
| `bash` 且 `async: true` | 后台启动长时间命令 |
| `write_bash` | 向运行中的异步会话发送输入 |
| `read_bash` | 读取异步会话输出 |
| `stop_bash` | 终止异步会话 |
| `list_bash` | 列出所有活动 shell 会话 |

## 其他 Copilot CLI 工具

| 工具 | 用途 |
|------|--------|
| `store_memory` | 持久保存关于代码库的事实供后续会话使用 |
| `report_intent` | 用当前意图更新 UI 状态行 |
| `sql` | 查询会话 SQLite 数据库（待办、元数据等） |
| `fetch_copilot_cli_documentation` | 查阅 Copilot CLI 文档 |
| GitHub MCP（`github-mcp-server-*`） | 原生 GitHub API（issue、PR、代码搜索等） |
