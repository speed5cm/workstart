# workstart

给 agent 用的资料库：**挑好的 MCP 与 skill，一条一句说明、一条一个能直发的安装指南**。要用哪条，就把那条的链接发给 agent，让它自己读指南执行——不需要它读整份 README。Pi 是这些条目当前的参考实现。

只收**能跨 agent 搬**的东西：MCP 服务器和 skill 包。绑死某个 runtime 的 UI 扩展、TUI 美化、权限插件一律不收——那是一层换 agent 就得重写的东西。

> 灵感来源：[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) 与 opencode 版本（[kickstart.opencode](https://github.com/orionpax1997/kickstart.opencode)）。内容源自 [kickstart.pi](https://github.com/orionpax1997/kickstart.pi)（MIT，© OrionPax），本仓库 fork 后按「通用 agent」重写。

> docs 里的安装链接全部指向本仓库（`speed5cm/workstart`，`main` 分支），所以改完 docs 推上去，别人按 README 自举安装读到的就是你自己那份。

---

## 目录

- [清单](#清单)
- [设计理念](#设计理念)
- [agent 的五层配置面](#agent-的五层配置面)
- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [如何让它为你所用](#如何让它为你所用)
- [这不是什么](#这不是什么)
- [内置](#内置)
- [节省 token](#节省-token)
- [Agentic 工作流](#agentic-工作流)
- [探索代码库](#探索代码库)
- [AGENTS.md](#agentsmd)

---

## 清单

这是索引层：**一条一个条目，一句话说清它解决什么，一个能直接发给 agent 的安装指南**。用法就一句——把某行的「指南」链接发给 agent：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-caveman.md
```

它自己会读、自己会装，不需要读整份 README。下面的详细章节是其中几个条目的展开说明，留着按需查。

### MCP —— 外部工具服务器

跨 runtime 最通用的一层：服务器本身跟 agent 无关，只改配置文件放哪。

| 名字 | 解决什么 | 装法 | 指南 |
|---|---|---|---|
| `exa` | 网页搜索（会话启动时预热连接） | `mcp-adapter.json` 里加 `{"url": "https://mcp.exa.ai/mcp", "lifecycle": "eager"}` | [installation.md](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation.md) |
| `context7` | 库 / 框架文档查询 | `{"url": "https://mcp.context7.com/mcp"}` | 同上 |
| `searchcode` | 公开仓库代码搜索 | `{"url": "https://api.searchcode.com/v1/mcp"}` | 同上 |
| `codegraph` | 代码库预索引成图谱，一次查询拿调用链和影响面 | `npx -y @colbymchenry/codegraph install -t opencode -l local -y`，再 `codegraph init` | [codegraph](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-codegraph.md) |
| `codebase-memory-mcp` | 本地 SQLite 持久图谱，158 种语言 | `npm i -g codebase-memory-mcp` | [codebase-memory-mcp](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-codebase-memory-mcp.md) |

### skills —— 方法论包（Markdown，最好搬）

换 agent 时这一层基本能整段带走，只有安装路径和触发方式不同。

| 名字 | 解决什么 | 装法 | 指南 |
|---|---|---|---|
| `mattpocock/skills` | 需求访谈 / TDD / 调试 / 评审 / issue 分流 | `npx skills@latest add mattpocock/skills --agent pi -y` | [mattpocock/skills](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-matt-pocock-skills.md) |
| `superpowers` | 头脑风暴 / 调试 / TDD / 规划 / 评审 | `pi install https://github.com/obra/superpowers` | [superpowers](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-superpowers.md) |
| `OpenSpec` | spec-driven：变更 → 规格 → 任务 → 归档 | `openspec init --tools pi` | [OpenSpec](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-openspec.md) |
| `caveman` | 压缩输出，省 65–75% token | `npx skills add https://github.com/juliusbrussee/caveman --skill caveman --agent pi -y` | [caveman](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-caveman.md) |
| `agent-browser` | 用 CDP 控制真实浏览器 | `npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser --agent pi -y` | [agent-browser](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-agent-browser.md) |

### CLI 工具 —— 靠宿主的拦截点生效

工具本身跟 agent 无关，但要让 agent「看见」改写后的输出，得挂在它的一个钩子上。`--agent` 选谁，就往谁的扩展机制里写一个壳；换 agent 是换个参数，不是重写。

| 名字 | 解决什么 | 装法 | 指南 |
|---|---|---|---|
| `rtk` | 把冗长的 shell 输出（`git status`、`pnpm list`、`vitest`、`cargo test`…）改写成紧凑形式，每次调用都省 | `rtk init --agent pi --global` | [rtk](https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-rtk.md) |

> 每条指南开头都有一行 `Generic capability` / `This page's implementation`：第一行说它解决的通用问题，第二行说哪些部分是 Pi 的。**判断能不能跨 agent 用，看第二行就够。**

---

## 设计理念

大部分"AI 配置起点"仓库给你的是一个成品。你拿到了能力，但不知道每一步在做什么、为什么这么做、它属于哪一层。

**`workstart` 反过来**：

- **先讲层，再讲工具** —— 每个工具都标注它补的是哪一层能力，而不只是"装它很好用"
- **每个决定都解释原因** —— 包括"为什么这个建议装项目级、那个建议装全局"
- **它是起点，不是框架** —— 你应当添加自己的设置、删掉你用不到的
- **不内置 skill、不内置 agent** —— 你需要时再添加
- **Pi 只作为参考实现** —— 命令是 Pi 的，判断是可迁移的；换 runtime 时改的是路径和安装方式，不是"该不该装"

---

## agent 的五层配置面

不管是哪个 agent，可配置的东西几乎都能归到这五层。Pi 的落点如下 —— 这一节是整份文档的骨架，后面每个工具都会说明自己补的是哪一层。

| 配置面 | 通用形态 | Pi 的落点 | 换 runtime 时改什么 |
|---|---|---|---|
| **指令文件** | 每次会话自动加载的上下文：语言偏好、编码规范、MCP 用法 | `AGENTS.md`（全局 `~/.pi/agent/AGENTS.md`、项目根一份） | 文件名与发现规则（`CLAUDE.md`、`.cursor/rules` 等都是同一层的不同名字） |
| **skill** | 按任务自动加载的能力包，正文是 Markdown | `.pi/skills/`（项目）、`~/.pi/agent/skills/`（全局） | 目录约定 |
| **扩展 / 插件** | 进程内加工具、加 UI、改渲染、拦截调用 | `~/.pi/agent/extensions/`、`pi install npm:...` | 安装方式与扩展 API |
| **MCP** | 跨 runtime 的外部工具服务器（这一层本来就是通用的） | `mcp-adapter.json`（由 `pi-mcp-adapter` 桥接） | 配置文件位置 |
| **会话落盘** | 会话记录，供回放、统计、外部 UI 读取 | `~/.pi/agent/sessions/*.jsonl` | 路径与格式 |

一条经验：**越靠上的层越通用，越靠下的层越绑 runtime。** 指令文件和 skill 换个 agent 基本照抄；扩展和会话格式通常得重写。

**所以这张清单只收三样东西：指令文件（AGENTS.md 那一节）、skill、MCP** —— 外加少数本身与 agent 无关、只需挂在宿主拦截点上的 CLI 工具（如 `rtk`）。扩展层不进清单 —— 它写的是某个 TUI 的渲染和某个 runtime 的事件总线上，换个 agent 归零。不是它没用，是它只对当下这个 agent 有用。

---

## 快速开始

**让 agent 帮你安装**（推荐）

把下面这段粘贴到 agent 会话里：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation.md
```

指南里的命令是 Pi 的。如果不是 Pi，有用的部分是**结构**：装配置目录 → 接 MCP → 写指令文件，三步一样，命令换掉。

**手动安装**

> ⚠️ 如果 `~/.pi/agent/` 已有配置，请先备份。

```bash
git clone https://github.com/speed5cm/workstart ~/.pi/agent
pi
```

之后在任何项目目录下启动：

```bash
cd /path/to/project
pi
```

---

## 项目结构

```
workstart/
│
├── LICENSE
├── README.md            ← 你正在阅读的这份
└── docs/
    ├── installation.md        ← 安装 / 备份 / 升级，含三个必装 MCP
    ├── installation-rtk.md    ← 可选：把 shell 输出改写成紧凑形式，省 token（全局）
    ├── installation-caveman.md   ← 可选：压缩 prose 输出，省 token（项目级）
    ├── installation-matt-pocock-skills.md   ← 可选：mattpocock/skills，写给真实工程师的 skill 集（项目级）
    ├── installation-superpowers.md  ← 可选：agentic workflow skills（项目级）
    ├── installation-openspec.md   ← 可选：spec-driven 开发工作流（项目级）
    ├── installation-agent-browser.md   ← 可选：CDP 浏览器自动化（项目级）
    ├── installation-codegraph.md   ← 可选：预索引的代码知识图谱（项目级）
    └── installation-codebase-memory-mcp.md   ← 可选：SQLite 知识图谱（项目级）
```

就这些。**没有 `settings.json`、没有内置 skill、没有内置 agent**。`workstart` 刻意保持精简 —— 你的模型、主题、其它工具，都由你自己在 `~/.pi/agent/` 中配置。唯一必装的是 [`docs/installation.md`](docs/installation.md) 里的三个 MCP 服务器（`context7`、`searchcode`、`exa`）——因为 MCP 是唯一跨 runtime 的通用层，不装它，指令文件里那些"用 X 查文档"的话就无从落地。

---

## 如何让它为你所用

1. **通读 `README.md`** —— 理解五层配置面，以及每个条目补的是哪一层。
2. `workstart` 故意不附带任何配置。首次启动时，pi 会引导你设置 provider、model、theme。
3. **先接 MCP** —— [exa、context7、searchcode](docs/installation.md) 是全局指令里那些"用 X 查文档"的落点。不接，`AGENTS.md` 里的话无从执行。这一层换 agent 只改配置文件位置，最值得先打通。
4. **全局装 rtk** —— [rtk](docs/installation-rtk.md) 是清单里唯一值得无条件全局装的：它拦的是 shell 输出，每个项目都受益，没有副作用。装一次，所有会话都省。
5. **按需装 skill，而且装项目级** —— [codegraph](docs/installation-codegraph.md) / [codebase-memory-mcp](docs/installation-codebase-memory-mcp.md) 看代码，[mattpocock/skills](docs/installation-matt-pocock-skills.md) 或 [superpowers](docs/installation-superpowers.md) 提供方法论，[OpenSpec](docs/installation-openspec.md) 走 spec-driven 开发，[caveman](docs/installation-caveman.md) 压缩 prose，[agent-browser](docs/installation-agent-browser.md) 控制浏览器。每个都只对你 `cd` 进去的项目生效。
6. **修改全局 `AGENTS.md`** —— 加你的语言偏好、工作风格、跨项目都适用的 MCP 用法提示。这一层换 agent 几乎可以照抄，投资回报最高。
7. **在需要 `AGENTS.md` 的项目根加一份** —— 写项目结构、技术栈、编码规范。
8. **写自己的 prompt 模板** —— 把重复流程做成斜杠命令（Pi 的落点是 `.pi/prompts/`，别的 agent 换个目录）。

用不到的，删掉。它是起点，不是框架。

---

## 这不是什么

- 不是多 agent 编排系统
- 不是可投产的 AI 流水线
- 不是 Pi 的成品配置 —— 是「能跨 agent 搬的条目清单」，Pi 只是这些条目的落地示范
- 不是扩展、主题、TUI 美化的收集地 —— 那一层换 agent 归零，这里一律不收
- 不是不需要读就能用的东西

---

## 内置

[`docs/installation.md`](docs/installation.md) 的安装流程把开箱即用所需的东西都装好了。按五层归类：

**指令文件** —— [`AGENTS.md`](#agentsmd)

**MCP**（跨 runtime 的通用层）

> **Pi 的原生设计是"不需要 MCP"。** Pi 以 extension 和 skill 为核心，直接加载到自己的进程里跑——MCP 不在它的核心架构里。
>
> **那为什么这里还要装一个 MCP 桥？** 因为整个 agent 生态（Context7、SearchCode、Exa，以及大部分第三方工具）都用 MCP 通信。`pi-mcp-adapter` + 几个 MCP 服务器，就是这份配置跟那个生态接轨的方式，本身不站队。换成别的 agent 时，这一层通常能直接复用，只改配置文件位置。

**MCP 桥**（Step 2 的前置依赖）：

- **pi-mcp-adapter** — 让 pi 能与 MCP 服务器通信的 adapter 包

**三个必装的 MCP 服务器**（Step 2）：

- **context7** — 库和框架文档查询
- **searchcode** — 公开仓库代码搜索
- **exa** — 网页搜索（会话启动时 eager 加载）

**不附带 `settings.json`** —— pi 首次启动时会引导你选 provider / model / theme。如果已有备份，可以用 `cp ~/.pi/agent.bak/settings.json ~/.pi/agent/` 恢复。

这三个 MCP 是全局 `AGENTS.md` 引用的目标 —— 不装的话，那些指令无从查询。

---

## 节省 token

> 这一节是**最小的通用层**：不管哪个 agent，token 都是钱，压缩点的位置也就那么几个 —— 命令输出、自然语言输出。两个工具的取舍也一样：**压缩有副作用就按项目开关，没有副作用才全局装。**

### rtk

[rtk](https://github.com/rtk-ai/rtk) 透明地把冗长的 shell 命令（`git status`、`pnpm list`、`vitest`、`cargo test` 等）改写成节省 token 的紧凑形式。装上之后每次会话都被自动拦截改写 —— 工作流不变，token 变少。

它只包 `bash` 那一层：`read` / `grep` / `find` / `ls`、`write`、`edit` 都不改写。

建议**全局安装**。把下面这段粘贴到 agent 会话：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-rtk.md
```

### caveman

[caveman](https://github.com/juliusbrussee/caveman) 把 pi 的自然语言回复压缩约 65–75%，同时保留技术准确性。六个强度等级，由 `/caveman` 或 "caveman mode" / "less tokens" 等关键词触发。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-caveman.md
```

---

## Agentic 工作流

> 这一节补的是**指令文件 + skill 层**，也是最容易照搬到别的 agent 的一节 —— 里面装的东西本质是"方法论写成 Markdown"，跟 runtime 无关。

### mattpocock/skills

[mattpocock/skills](https://github.com/mattpocock/skills) 是 Matt Pocock 整理的工程 skill 集——头脑风暴式的需求访谈、TDD、bug 诊断、代码审查、架构调研、issue 分流等等。按任务自动加载；部分 skill 还注册成 `/skill:<name>` 斜杠命令。定位是体量小、好改造、易组合。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-matt-pocock-skills.md
```

### superpowers

[superpowers](https://github.com/obra/superpowers) 提供一套精选 skill，覆盖头脑风暴、调试、TDD、规划、代码审查等场景。按任务自动加载——装好之后无需手动调用。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-superpowers.md
```

### OpenSpec

[OpenSpec](https://github.com/Fission-AI/OpenSpec) 是 spec 驱动的开发工具，用于生成和管理项目规范。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-openspec.md
```

---

## 探索代码库

> 这一节补的是 **MCP 层**：把"爬文件"换成"查图"。换 agent 时这一层最省事——MCP 服务器本身不动，只改配置文件放哪。

### codegraph

[codegraph](https://github.com/colbymchenry/codegraph) 把代码库预先索引成知识图谱。agent 一次查询就能拿到精确上下文——调用链路、影响范围、相关符号——而不是逐个文件爬取。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-codegraph.md
```

### codebase-memory-mcp

[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) 是独立的静态二进制，把代码库索引成持久化的 SQLite 知识图谱（支持 158 种语言、14 个 MCP 工具、毫秒级查询）。和 codegraph 类似，它让 agent 一次调用就能拿到结构化上下文。

建议**项目级别安装**。先 `cd` 到项目目录，再把下面这段粘贴到 pi：

```
Read the installation guide and follow it:
https://raw.githubusercontent.com/speed5cm/workstart/refs/heads/main/docs/installation-codebase-memory-mcp.md
```

---

## AGENTS.md

AGENTS.md 是 pi 在每个会话里加载的全局指令文件 —— 五层里的第一层，也是最通用的一层。保持简短——只写真正跨项目都适用的内容。

pi 还会从 cwd 向上自动发现 `AGENTS.md` / `CLAUDE.md`，所以项目根的 `AGENTS.md` 会在该项目中覆盖全局。

**全局文件建议包含**：

- **语言偏好** — 如 `Reply in Chinese.`
- **MCP 使用提示** — 如 `Use context7 to look up library and framework documentation.`
- **个人编码偏好** — 如 `Never use any type. Prefer explicit types.`
- **做事风格** — 如 `Keep responses concise. No need to explain obvious steps.`

**项目级 `AGENTS.md`** 放在项目根目录，描述项目结构、技术栈、开发规范等。在该项目中会覆盖全局设置。

> 换 agent 时，这一层几乎可以整段搬过去 —— 只要目标 agent 支持"自动加载的指令文件"这种机制（名字可能是 `CLAUDE.md`、`.cursor/rules`、`copilot-instructions.md`）。这也是为什么它值得先花时间写好。
