# graphify 产物用途和使用方法

> 一句话版：第一次接触项目先看 `GRAPH_REPORT.md`；要精确回答问题看 `graph.json` + `query/path/explain`；想图形化探索看 `graph.html`；想像文档一样逐层浏览看 `wiki/`。

本文说明 graphify 常见产物分别解决什么问题、适合谁使用，以及推荐的使用方式。内容是通用说明，不依赖某个具体项目当前是否已经生成这些文件。

> 命令说明：下文统一使用终端写法 `graphify`。如果你在支持 slash skill 的 AI 助手中使用，可以把它理解为 `/graphify`；如果你在 PowerShell 里执行，也建议直接使用 `graphify`。

## 先看结论

如果你只想知道“现在该打开哪个文件”，先看这张表：

| 你的目标 | 最先看什么 | 为什么 |
| --- | --- | --- |
| 快速认识项目全局 | `GRAPH_REPORT.md` | 它把 God Nodes、社区、意外连接和建议问题都先整理好了 |
| 精确追一个问题 | `graph.json` + `graphify query/path/explain` | 它是完整知识图底座，适合做精确关系查询 |
| 看整体拓扑和桥接点 | `graph.html` | 适合交互式探索社区、节点和跨模块连接 |
| 按主题慢慢读 | `wiki/` | 更像文档站，适合顺着目录和主题往下走 |
| 沉淀到个人知识库 | `obsidian/` | 适合长期维护、补充笔记和手工整理 |
| 给团队讲架构 | `callflow-html` | 比原始图更强调“讲清楚”，更接近可交付文档 |
| 导出到 Gephi / yEd | `graph.graphml` | 方便接专业图工具继续分析和布局 |
| 接入 Neo4j | `cypher.txt` 或 `--neo4j-push` | 适合图库查询、可视化和跨项目聚合 |
| 提速重跑、做增量更新 | `manifest.json`、`cache/` | 这是增量更新和缓存命中的关键辅助产物 |

## 输出目录长什么样

执行 graphify 后，常见产物通常会落在 `graphify-out/` 目录下：

```text
graphify-out/
├── GRAPH_REPORT.md
├── graph.json
├── graph.html
├── wiki/
├── obsidian/
├── graph.svg
├── graph.graphml
├── cypher.txt
├── manifest.json
├── cost.json
├── cache/
├── converted/
├── .graphify_labels.json
├── .graphify_analysis.json
└── .graphify_root
```

并不是每次都会生成全部文件。一般可以这样理解：

- `GRAPH_REPORT.md`、`graph.json` 是最核心的两份产物。
- `graph.html`、`wiki/`、`obsidian/`、`graph.svg`、`graph.graphml`、`cypher.txt` 属于按需导出或按场景使用。
- `manifest.json`、`cache/`、`converted/`、`.graphify_*` 属于增量更新、分析结果或导出辅助文件。

## 核心产物

### `graph.json`

- **解决的问题：**提供完整知识图数据，回答“具体哪个概念和哪个概念有关系、跨过了哪些节点、证据在哪里”。
- **最适合的人：**CLI 查询用户、Agent / MCP / 自动化流程、想做二次分析或接外部图工具的人。
- **推荐打开方式：**通常不手工读 JSON，而是通过命令消费它。
- 通过以下命令生成：

```bash
graphify extract .
```

- **常用命令：**

```bash
graphify query "what connects auth to the database?"
graphify path "UserService" "DatabasePool"
graphify explain "RateLimiter"
```

如果你要显式指定图文件：

```bash
graphify query "what connects DigestAuth to Response?" --graph graphify-out/graph.json
```

- **一句话提醒：**如果你要把 graphify 接进自己的工具链，`graph.json` 通常是第一优先级输入文件。

### `GRAPH_REPORT.md`

- **解决的问题：**快速回答“这个项目大概长什么样、重点在哪、哪里值得继续深挖”。
- **最适合的人：**第一次接手项目的人、做架构理解的 Agent、代码评审或项目交接场景。
- **推荐打开方式：**先看 `God Nodes`、`Surprising Connections`、`Suggested Questions`、`Knowledge Gaps` 这几块，再决定下一步查什么。
- **通过以下命令生成（需要先执行 `graphify extract .`）：**

```bash
graphify cluster-only .
```

- **一句话提醒：**它适合建立全局感，不适合替代精确查询；一旦问题变成“谁和谁到底怎么连起来”，就该切到 `query/path/explain`。

## 可视化与导航产物

### `graph.html`

- **解决的问题：**从“图”的角度看项目结构，而不是只看文字报告。
- **最适合的人：**人类开发者、做架构演示的人、想快速找跨模块桥接点的人。
- **推荐打开方式：**生成后直接用浏览器打开，搜索节点、切 community、观察桥接关系。
- **常用命令：**

```bash
graphify export html
```

- **一句话提醒：**它适合看全局结构和拓扑感；默认会在 `GRAPHIFY_VIZ_NODE_LIMIT=20000` 以内直接出 HTML，超过当前阈值时会切到聚合视图或建议你改走 `query/path/explain` / `--no-viz`。

### `wiki/`

- **解决的问题：**把知识图变成可导航的 Markdown 文档站。
- **最适合的人：**Agent、偏文档驱动的团队、喜欢按主题逐层浏览的人。
- **典型结构：**通常会有 `index.md` 作为入口，再加上每个 community 和关键节点的主题页面。
- **推荐打开方式：**从 `graphify-out/wiki/index.md` 开始浏览，而不是直接翻散落页面。
- **常用命令：**

```bash
graphify export wiki
```

- **一句话提醒：**`wiki/` 更像“知识导航站”，比 `graph.html` 更适合按主题阅读和顺着上下文深入。

### `obsidian/`

- **解决的问题：**把知识图导出成可长期维护的 Obsidian vault。
- **最适合的人：**做长期知识沉淀的人、研究型团队、会手工补充注释和笔记的人。
- **附带内容：**通常会一起生成节点笔记、community 概览、`graph.canvas`，以及给 Obsidian 图视图使用的配置文件。
- **推荐打开方式：**把 `graphify-out/obsidian/` 当成一个 vault 直接打开。
- **常用命令：**

```bash
graphify export obsidian
```

- **一句话提醒：**如果只是临时排查一个代码问题，它往往不是第一站；如果你要把项目知识持续沉淀下来，它就很有价值。

### `graph.canvas`

- **解决的问题：**把 community 和节点排成可空间化浏览的画布。
- **最适合的人：**需要做演示、喜欢按空间布局理解结构、想手工整理社区关系的人。
- **推荐打开方式：**通常由 `--obsidian` 自动生成，在 Obsidian 里直接打开对应 canvas 文件即可。
- **一句话提醒：**它更偏“视觉布局”和“演示表达”，而不是精确查询工具。

### `callflow-html`

- **解决的问题：**生成更像架构说明书的 HTML，而不是纯图探索页面。
- **最适合的人：**要给团队做交付、要讲清模块关系和调用流、要生成更可读的架构页面的人。
- **推荐打开方式：**把它当作“可交付的说明页”而不是“图工具”，重点看 Mermaid 调用流、分区概览和关键文件卡片。
- **常用命令：**

```bash
graphify export callflow-html
graphify export callflow-html --output docs/architecture.html
```

- **一句话提醒：**如果 `graph.html` 强调“自由探索”，那 `callflow-html` 更强调“讲清楚”。

### `GRAPH_TREE.html`

- **解决的问题：**从目录 / 模块树的角度看项目，而不是从语义关系网看项目。
- **最适合的人：**想先理解目录层级和文件分布的人。
- **推荐打开方式：**把它当作 `graph.html` 的补充视角，用来回答“文件都放在哪、模块层级怎么分”。
- **常用命令：**

```bash
graphify tree
graphify tree --output docs/graph-tree.html
```

- **一句话提醒：**当你想看“结构层级”而不是“概念连接”时，它会比知识图更直接。

## 外部工具导出产物

| 产物 | 作用 | 常用命令 | 适合场景 |
| --- | --- | --- | --- |
| `graph.svg` | 导出静态图，方便嵌入 README、Notion、Obsidian 或文档 | `graphify . --svg` | 想要可归档、易分享、不依赖 JS 的图谱结果 |
| `graph.graphml` | 导出给 Gephi、yEd 等专业图工具 | `graphify . --graphml` | 想继续做布局、过滤、统计或专业图分析 |
| `cypher.txt` | 为 Neo4j 生成导入脚本 | `graphify . --neo4j` | 团队已经使用 Neo4j，希望通过 Cypher 查询图谱 |
| 直接推送到 Neo4j | 跳过中间文件，把图直接写入运行中的 Neo4j | `graphify . --neo4j-push bolt://localhost:7687` | 想把图直接接入图库、做跨项目聚合或数据库可视化 |

## 增量更新与辅助产物

这些文件多数不是“阅读入口”，但对增量更新、调试导出和成本控制很重要。

| 文件 / 目录 | 作用 | 平时要不要主动打开 |
| --- | --- | --- |
| `manifest.json` | 记录文件的 `mtime`、`ast_hash`、`semantic_hash`；`graphify update` 走 AST 增量，`graphify extract` 走语义增量；如果某次语义抽取失败，对应文件会保留空的 `semantic_hash` 以便下次只重试失败项 | 通常不用手工编辑；团队提交时一般会忽略它 |
| `cache/` | 缓存 AST 和语义提取结果，加速重跑并降低 token 成本 | 平时不用看；排查提取问题时很有帮助 |
| `converted/` | 保存某些非纯文本输入的转换结果，比如 Office 文档转成的 Markdown sidecar | 平时不是主入口；排查“为什么抽成这样”时很关键 |
| `cost.json` | 记录本地成本或 token 统计 | 只在审计、复盘或成本控制时需要 |
| `.graphify_labels.json` | 保存 community 编号到显示名称的映射 | 做导出定制或调试显示标签时有用 |
| `.graphify_analysis.json` | 保存 community、god nodes、surprising connections 等分析结果 | 平时不用看；如果 `wiki` 或导出提示缺失，可运行 `graphify extract .` 或 `graphify cluster-only .` 重新生成 |
| `.graphify_root` | 标记当前输出目录对应的项目根 | 基本不需要手工操作 |

## 推荐使用顺序

### 场景一：第一次接触一个项目

1. 先看 `GRAPH_REPORT.md`，建立全局印象。
2. 对具体问题用 `graphify query/path/explain` 往下追。
3. 需要拓扑感或桥接点时打开 `graph.html`。
4. 需要文档式浏览时再导出 `wiki/`。

### 场景二：日常开发排查

1. 先问 `graphify query`。
2. 追两个概念之间的关系时用 `graphify path`。
3. 只想看单个节点或概念时用 `graphify explain`。
4. 改的是代码就运行 `graphify update .`；改的是文档、论文或图片，则运行 `/graphify --update` 或 `graphify extract <path>` 做语义重抽。

### 场景三：做团队交付或架构说明

1. 先用 `GRAPH_REPORT.md` 提炼重点。
2. 再生成 `callflow-html` 作为可读性更强的交付物。
3. 需要图示时补上 `graph.svg` 或 `graph.html`。

### 场景四：接入外部知识系统

1. 优先用 `graph.json` 作为底层输入。
2. 需要外部图工具时导出 `graph.graphml`。
3. 需要图库查询时使用 `cypher.txt` 或直接 `--neo4j-push`。

## 常用命令速查

```bash
graphify extract .                # 构建当前目录知识图，语义增量/全量抽取（headless，可重试失败语义文件）
graphify update .                 # 代码图增量更新（AST-only）
graphify cluster-only .           # 只重跑聚类和报告

graphify query "what connects auth to the database?"
graphify path "UserService" "DatabasePool"
graphify explain "RateLimiter"

graphify export html              # 生成 graph.html
graphify export callflow-html     # 生成架构文档 HTML
graphify tree                     # 生成树视图 HTML

graphify . --wiki                 # 生成 wiki
graphify . --obsidian             # 生成 Obsidian vault
graphify . --svg                  # 导出 SVG
graphify . --graphml              # 导出 GraphML
graphify . --neo4j                # 导出 cypher.txt
graphify . --neo4j-push bolt://localhost:7687
```

## 最后只记住这几条

- **想先看全局：**`GRAPH_REPORT.md`
- **想精确问问题：**`graph.json` + `graphify query/path/explain`
- **想图形化探索：**`graph.html`
- **想文档式导航：**`wiki/`
- **想沉淀到个人知识库：**`obsidian/`
- **想给团队交付架构说明：**`callflow-html`
- **想接 Gephi / yEd：**`graph.graphml`
- **想接 Neo4j：**`cypher.txt` 或 `--neo4j-push`
- **想提速重跑：**`manifest.json`、`cache/`

## 维护建议

- 代码更新后尽量及时运行 `graphify update .`；如果改的是文档、论文或图片，请改用 `/graphify --update` 或 `graphify extract <path>`，因为 `update` 本身不做 LLM 语义重抽。
- 团队协作时，通常建议提交 `graphify-out/`，但忽略 `manifest.json`、`cost.json`，并按仓库体积决定是否提交 `cache/`。
- 回答代码库问题时，优先走图查询，再退回原始 `grep`；这样通常更快，也更不容易漏掉跨文件关系。

## 全部控制台命令

如果你只想查看最新参数、默认值或新增命令，直接运行：

```bash
graphify --help
```

下面这份是按用途整理的“可读版”命令索引，比直接贴原始 help 更适合查阅。

> 注：这里按命令的真实归属整理。当前 `graphify --help` 输出里，`clone` 的 `--branch` / `--out` 会和 `merge-graphs` 连在一起显示，文档这里已按实际用途拆开。

### 安装与卸载

| 命令 | 作用 | 备注 |
| --- | --- | --- |
| `graphify install [--platform P]` | 把 graphify skill 安装到指定平台配置目录 | `P` 支持 `claude`、`windows`、`codex`、`opencode`、`aider`、`claw`、`droid`、`trae`、`trae-cn`、`gemini`、`cursor`、`antigravity`、`hermes`、`kiro`、`pi` |
| `graphify uninstall` | 从所有已检测到的平台一键卸载 graphify | 适合整机清理 |
| `graphify uninstall --purge` | 卸载的同时删除 `graphify-out/` | 会连输出目录一起清理 |

### 图查询与解释

| 命令 | 作用 | 常用参数 |
| --- | --- | --- |
| `graphify path "A" "B"` | 查询两个节点之间的最短路径 | `--graph <path>` 指定图文件 |
| `graphify explain "X"` | 用自然语言解释某个节点及其邻居 | `--graph <path>` 指定图文件 |
| `graphify query "<question>"` | 基于 `graph.json` 做问题查询 | `--dfs` 改用 DFS；`--context C` 过滤边上下文；`--budget N` 限制输出 token；`--graph <path>` 指定图文件 |
| `graphify save-result` | 把一次问答结果保存到 `graphify-out/memory/` | `--question Q`、`--answer A`、`--type query\|path_query\|explain`、`--nodes N1 N2 ...`、`--memory-dir DIR` |

### 输入、更新与重建

| 命令 | 作用 | 常用参数 |
| --- | --- | --- |
| `graphify clone <github-url>` | 克隆 GitHub 仓库并输出本地路径，方便后续 `/graphify` | `--branch <branch>` 指定分支；`--out <dir>` 指定克隆目录 |
| `graphify add <url>` | 抓取 URL 内容到 `./raw`，然后立刻更新图 | `--author "Name"`、`--contributor "Name"`、`--dir <path>` |
| `graphify watch <path>` | 监听目录变化并自动重建代码图；代码改动会立即更新，文档/图片改动会写 `needs_update` 提示你补做语义抽取 | 适合本地持续开发 |
| `graphify update <path>` | 只重抽代码文件并更新图，不需要 LLM | `--force` 强制覆盖节点变少的图；也可用 `GRAPHIFY_FORCE=1`；`--no-cluster` 跳过聚类；文档/图片改动请改用 `/graphify --update` 或 `graphify extract <path>` |
| `graphify check-update <path>` | 检查 `needs_update` 标记，提示是否仍需补做语义重抽取 | 适合 cron 或自动化健康检查 |
| `graphify cluster-only <path>` | 在现有 `graph.json` 上重跑聚类并重建报告 | `--no-viz` 跳过 `graph.html`；`--graph <path>` 指定图文件 |

### 全量抽取与跨项目管理

| 命令 | 作用 | 常用参数 |
| --- | --- | --- |
| `graphify extract <path>` | 头less 全量/增量抽取（AST + 语义 LLM），适合 CI 或脚本；结束时会输出 `Error Summary`，失败语义文件会在下次增量时单独重试 | `--backend`、`--model`、`--max-workers`、`--token-budget`、`--max-concurrency`、`--api-timeout`、`--out`、`--google-workspace`、`--no-cluster`、`--global`、`--as` |
| `graphify merge-driver <base> <current> <other>` | `graph.json` 的 Git merge driver，用并集合并两个图 | 通常配合 `graphify hook install` 使用 |
| `graphify merge-graphs <g1> <g2> [...]` | 合并多个 `graph.json`，生成跨仓库图 | `--out <path>` 指定输出文件，默认是 `graphify-out/merged-graph.json` |
| `graphify global add <graph.json>` | 把一个项目图加入全局图 | `--as <tag>` 指定仓库标签，默认取父目录名 |
| `graphify global remove <tag>` | 从全局图移除某个项目 | 无 |
| `graphify global list` | 列出全局图里已注册的项目 | 无 |
| `graphify global path` | 输出全局图文件路径 | 无 |

### 导出与度量

| 命令 | 作用 | 常用参数 / 备注 |
| --- | --- | --- |
| `graphify tree` | 生成 D3 v7 可折叠树 HTML | `--graph PATH`、`--output HTML`、`--root PATH`、`--max-children N`、`--top-k-edges N`、`--label NAME` |
| `graphify benchmark [graph.json]` | 估算相对“直接喂完整语料”的 token 缩减效果 | 适合评估 graphify 的上下文压缩收益 |
| `graphify export callflow-html` | 导出 Mermaid 架构 / 调用流 HTML | 适合团队讲架构、沉淀架构文档或做交付说明 |

### Hook 与平台集成

| 命令 | 作用 | 备注 |
| --- | --- | --- |
| `graphify hook install` | 安装 `post-commit` / `post-checkout` hooks | 通常也会把 `graph.json` 的 merge driver 一并配好 |
| `graphify hook uninstall` | 移除 hooks | 用于回退本地自动化 |
| `graphify hook status` | 检查 hooks 是否已安装 | 适合排查“为什么没有自动更新” |

如果你更喜欢显式的平台命令，而不是通用的 `install --platform`，可以直接使用下面这张表：

| 平台 | 安装命令 | 卸载命令 | 会写入 / 移除 |
| --- | --- | --- | --- |
| Gemini CLI | `graphify gemini install` | `graphify gemini uninstall` | `GEMINI.md` 段落和 `BeforeTool` hook |
| Cursor | `graphify cursor install` | `graphify cursor uninstall` | `.cursor/rules/graphify.mdc` |
| Claude Code | `graphify claude install` | `graphify claude uninstall` | `CLAUDE.md` 中的 graphify 段落和 `PreToolUse` hook |
| Codex | `graphify codex install` | `graphify codex uninstall` | `AGENTS.md` 中的 graphify 段落 |
| OpenCode | `graphify opencode install` | `graphify opencode uninstall` | `AGENTS.md` 段落和 `tool.execute.before` plugin |
| Aider | `graphify aider install` | `graphify aider uninstall` | `AGENTS.md` 中的 graphify 段落 |
| GitHub Copilot CLI | `graphify copilot install` | `graphify copilot uninstall` | `~/.copilot/skills/graphify/` |
| VS Code Copilot Chat | `graphify vscode install` | `graphify vscode uninstall` | Copilot Chat 配置和 `.github/copilot-instructions.md` |
| OpenClaw | `graphify claw install` | `graphify claw uninstall` | `AGENTS.md` 中的 graphify 段落 |
| Factory Droid | `graphify droid install` | `graphify droid uninstall` | `AGENTS.md` 中的 graphify 段落 |
| Trae | `graphify trae install` | `graphify trae uninstall` | `AGENTS.md` 中的 graphify 段落 |
| Trae CN | `graphify trae-cn install` | `graphify trae-cn uninstall` | `AGENTS.md` 中的 graphify 段落 |
| Google Antigravity | `graphify antigravity install` | `graphify antigravity uninstall` | `.agents/rules`、`.agents/workflows` 和 skill |
| Hermes | `graphify hermes install` | `graphify hermes uninstall` | `~/.hermes/skills/graphify/` |
| Kiro IDE / CLI | `graphify kiro install` | `graphify kiro uninstall` | `.kiro/skills/graphify/` 和 steering file |
| Pi coding agent | `graphify pi install` | `graphify pi uninstall` | `~/.pi/agent/skills/graphify/` |

> 补充：`windows` 平台当前走的是通用入口 `graphify install --platform windows`，不在这张平台快捷命令表里单独列出。
