# Gemini CLI 工具映射

技能使用 Claude Code 的工具名。遇到下列名称时，请使用你所在平台的等价工具：

| 技能中的名称 | Gemini CLI 等价 |
|-----------------|----------------------|
| `Read`（读文件） | `read_file` |
| `Write`（新建文件） | `write_file` |
| `Edit`（编辑文件） | `replace` |
| `Bash`（执行命令） | `run_shell_command` |
| `Grep`（搜内容） | `grep_search` |
| `Glob`（按名搜文件） | `glob` |
| `TodoWrite`（待办） | `write_todos` |
| `Skill`（调用技能） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task`（派发子智能体） | **无等价** — Gemini CLI 不支持子智能体 |

## 无子智能体支持

Gemini CLI 没有与 Claude Code `Task` 对等的工具。依赖子智能体派发的技能（`subagent-driven-development`、`dispatching-parallel-agents`）会退化为在单会话中用 `executing-plans` 执行。

## 其他 Gemini CLI 工具

下列工具在 Gemini CLI 可用，但 Claude Code 无直接对应：

| 工具 | 用途 |
|------|--------|
| `list_directory` | 列出文件与子目录 |
| `save_memory` | 持久化事实到跨会话的 GEMINI.md |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 丰富任务管理（创建、更新、列表、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 改动前切换到只读调研模式 |
