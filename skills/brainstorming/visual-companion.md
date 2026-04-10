# 可视化伴侣指南

基于浏览器的可视化头脑风暴伴侣，用于展示原型、示意图与选项。

## 何时使用

**按题决策，不是按会话。** 检验标准：**用户是「看到」比「读到」更清楚吗？**

**用浏览器**，当内容**本身**是视觉的：

- **UI 原型** — 线框、布局、导航结构、组件设计  
- **架构图** — 系统组件、数据流、关系图  
- **并排视觉对比** — 两种布局、配色、设计方向  
- **视觉打磨** — 关于观感、间距、视觉层级的问题  
- **空间关系** — 状态机、流程图、实体关系图画成图  

**用终端**，当内容是文字或表格：

- **需求与范围** — 「X 是什么意思？」「哪些在范围内？」  
- **概念性 A/B/C** — 用文字描述的方案选择  
- **权衡列表** — 利弊、对比表  
- **技术决策** — API 设计、建模、架构选型  
- **澄清问题** — 答案是文字而非视觉偏好  

*关于* UI 的话题不一定是视觉题。「你想要哪种向导？」是概念题——用终端。「这几种向导布局哪个顺眼？」是视觉题——用浏览器。

## 如何工作

服务器监视目录中的 HTML 文件，向浏览器提供**最新**的一个。你把 HTML 写到 `screen_dir`，用户在浏览器里看到并可点击选项。选择写入 `state_dir/events`，你下一回合读取。

**内容片段 vs 完整文档：** 若 HTML 以 `<!DOCTYPE` 或 `<html` 开头，服务器原样提供（只注入辅助脚本）。否则服务器自动用框架模板包裹——加页眉、CSS 主题、选择指示与交互。**默认写内容片段。** 只有需要完全控制页面时才写完整文档。

## 启动会话

```bash
# 带持久化启动（原型保存到项目）
scripts/start-server.sh --project-dir /path/to/project

# 返回：{"type":"server-started","port":52341,"url":"http://localhost:52341",
#        "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#        "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

保存响应里的 `screen_dir` 与 `state_dir`。告诉用户打开 URL。

**找连接信息：** 启动 JSON 写入 `$STATE_DIR/server-info`。若后台启动且未捕获 stdout，读该文件拿 URL 与端口。使用 `--project-dir` 时，在 `<project>/.superpowers/brainstorm/` 下找会话目录。

**注意：** 把项目根作为 `--project-dir` 传入，原型会落在 `.superpowers/brainstorm/` 并在服务器重启后保留。不传则文件在 `/tmp` 易被清理。提醒用户将 `.superpowers/` 加入 `.gitignore`（若尚未忽略）。

**按平台启动：**

**Claude Code（macOS / Linux）：**
```bash
# 默认模式即可——脚本会自行后台起服务
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code（Windows）：**
```bash
# Windows 自动检测并用前台模式，会阻塞工具调用。
# Bash 工具调用时设 run_in_background: true，服务才能跨回合存活。
scripts/start-server.sh --project-dir /path/to/project
```
调用 Bash 时设 `run_in_background: true`。下一回合读 `$STATE_DIR/server-info` 获取 URL 与端口。

**Codex：**
```bash
# Codex 会回收后台进程。脚本检测 CODEX_CI 并切前台。正常跑即可——无需额外参数。
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# 使用 --foreground，shell 工具调用设 is_background: true
# 进程才能跨回合存活
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他环境：** 服务器必须在回合间保持后台运行。若环境会回收 detached 进程，用 `--foreground` 并结合平台的后台执行机制。

若 URL 在浏览器不可达（远程/容器常见），绑定非 loopback：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

用 `--url-host` 控制返回 JSON 里打印的主机名。

## 循环

1. **确认服务存活**，然后向 `screen_dir` **写入新 HTML 文件**：  
   - 每次写入前确认 `$STATE_DIR/server-info` 存在。若不存在（或存在 `$STATE_DIR/server-stopped`），服务已关——用 `start-server.sh` 重启再继续。无操作 30 分钟服务会自动退出。  
   - 语义化文件名：`platform.html`、`visual-style.html`、`layout.html`  
   - **永远不要复用文件名** —— 每屏一个新文件  
   - 用 Write 工具 —— **不要用 cat/heredoc**（终端噪音大）  
   - 服务器自动提供**最新修改**的文件  

2. **告诉用户会看到什么并结束本回合：**  
   - 每步都提醒 URL（不只第一步）  
   - 简短文字概括屏幕上是什么（例如：「展示首页的 3 种布局」）  
   - 请用户在**终端**回复：「看一下并告诉我想法。需要可点击选项。」  

3. **下一回合** —— 用户在终端回复后：  
   - 若存在则读 `$STATE_DIR/events` —— 浏览器交互（点击、选择）的 JSON 行  
   - 与终端文字合并理解  
   - **终端消息是主反馈**；`state_dir/events` 提供结构化交互数据  

4. **迭代或前进** —— 若反馈改变当前屏，写新文件（如 `layout-v2.html`）。仅当本步验证通过再进入下一问。  

5. **回到终端时卸载** —— 下一步不需要浏览器时（澄清问题、权衡讨论），推一个等待屏以清掉陈旧画面：

   ```html
   <!-- 文件名：waiting.html 或 waiting-2.html 等 -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">继续在终端中……</p>
   </div>
   ```

   避免用户盯着已结束的选项而对话已进入下一阶段。下一道视觉题再照常推新内容文件。

6. 重复直到结束。

## 写内容片段

只写放进页面里的内容。服务器会自动用框架模板包裹（页眉、主题 CSS、选择条与交互）。

**最小示例：**

```html
<h2>哪种布局更好？</h2>
<p class="subtitle">考虑可读性与视觉层级</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>单栏</h3>
      <p>干净、专注阅读</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>双栏</h3>
      <p>侧栏导航 + 主内容</p>
    </div>
  </div>
</div>
```

无需 `<html>`、自建 CSS 或 `<script>`。服务器已提供。

## 可用 CSS 类

框架模板为内容提供这些类：

### 选项（A/B/C）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>标题</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

**多选：** 容器加 `data-multiselect`，点击切换选中。指示条显示数量。

```html
<div class="options" data-multiselect>
  <!-- 同上 option 标记——可多选/取消 -->
</div>
```

### 卡片（视觉稿）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- 原型内容 --></div>
    <div class="card-body">
      <h3>名称</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

### 原型容器

```html
<div class="mockup">
  <div class="mockup-header">预览：控制台布局</div>
  <div class="mockup-body"><!-- 你的原型 HTML --></div>
</div>
```

### 分栏（并排）

```html
<div class="split">
  <div class="mockup"><!-- 左 --></div>
  <div class="mockup"><!-- 右 --></div>
</div>
```

### 利弊

```html
<div class="pros-cons">
  <div class="pros"><h4>优点</h4><ul><li>好处</li></ul></div>
  <div class="cons"><h4>缺点</h4><ul><li>坏处</li></ul></div>
</div>
```

### 线框块

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">导航</div>
  <div class="mock-content">主内容区</div>
</div>
<button class="mock-button">操作按钮</button>
<input class="mock-input" placeholder="输入框">
<div class="placeholder">占位区</div>
```

### 排版与区块

- `h2` — 页标题  
- `h3` — 小节标题  
- `.subtitle` — 标题下副文  
- `.section` — 带下边距的内容块  
- `.label` — 小号大写标签  

## 浏览器事件格式

用户点击选项时，交互写入 `$STATE_DIR/events`（每行一个 JSON）。推新屏时文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
```

完整流显示探索路径——用户可能多点几次再定。通常最后一条 `choice` 是最终选择，但点击模式可暴露犹豫或偏好，值得追问。

若不存在 `$STATE_DIR/events`，用户未在浏览器交互——仅用终端文字。

## 设计提示

- **保真度与问题匹配** —— 布局用线框，「观感」题再抛光  
- **每页说清在问什么** —— 「哪种更专业？」而不只是「选一个」  
- **前进前先迭代** —— 反馈改变当前屏就写新版本  
- **每屏最多 2–4 个选项**  
- **该用真实内容时就用** —— 摄影作品集可用真实图（如 Unsplash）。占位内容会掩盖设计问题。  
- **原型保持简单** —— 聚焦布局与结构，不必像素级完美  

## 文件命名

- 语义名：`platform.html`、`visual-style.html`、`layout.html`  
- 永不复用文件名 —— 每屏新文件  
- 迭代：`layout-v2.html`、`layout-v3.html`  
- 服务器按修改时间取最新文件  

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

若用过 `--project-dir`，原型保留在 `.superpowers/brainstorm/` 供日后参考。仅 `/tmp` 会话会在 stop 时删。

## 参考

- 框架模板（CSS 参考）：`scripts/frame-template.html`  
- 客户端辅助脚本：`scripts/helper.js`  
