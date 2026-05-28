<div align="center">

# ✈️ flightdeck

**AI 协作工程的操作协议。**

[![Version](https://img.shields.io/github/v/release/Yuelioi/flightdeck?style=flat-square&color=2563eb)](https://github.com/Yuelioi/flightdeck/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-已测试-success?style=flat-square)](adapters/claude/README.md)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-未测试-yellow?style=flat-square)](adapters/codex/README.md)
[![Cursor](https://img.shields.io/badge/Cursor-未测试-yellow?style=flat-square)](adapters/cursor/README.md)
[![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-未测试-yellow?style=flat-square)](adapters/gemini/README.md)
[![AGENTS.md](https://img.shields.io/badge/emit_to-AGENTS.md-blueviolet?style=flat-square)](https://agents.md)

🇬🇧 [English README](README.md) · 🇨🇳 中文

</div>

---

> AI 助手在两次对话之间会失忆。**flightdeck** 用一套目录约定 + skill 给它跨会话的操作纪律 —— 下一次会话能接着上一次干，知道你之前在做什么、为什么、下一步该干嘛。

> **从 `workshop` 改名而来（≤ v0.8.1）** —— 项目身份从"工作台"调整为"操作可靠性"。老用户请看 [MIGRATION.md](MIGRATION.md)。

## 目录

- [TL;DR](#tldr)
- [它是什么](#它是什么)
- [架构](#架构)
- [`cockpit.md` 真实长什么样](#cockpitmd-真实长什么样)
- [为什么需要它](#为什么需要它)
- [设计哲学](#设计哲学)
- [安装](#安装)
- [用法](#用法)
- [兼容性](#兼容性)
- [横向对比](#横向对比)
- [FAQ](#faq)
- [文档](#文档)
- [贡献](#贡献)
- [Roadmap](#roadmap)
- [致谢](#致谢)
- [License](#license)

## TL;DR

```text
/plugin marketplace add Yuelioi/flightdeck
/plugin install flightdeck@flightdeck-marketplace
```

然后在任意有（或将要有）`flightdeck/` 目录的项目里启动 Claude Code session：协议自动加载、读 `flightdeck/cockpit.md`、跟 `git status` 对账，无缝续上之前的工作。

新项目用 `/flightdeck:flightdeck-workflow` 走两问 bootstrap，自动生成 `flightdeck/cockpit.md`。

## 它是什么

一个 `flightdeck/` 目录布局，AI 按约定读和写：

```
flightdeck/
├── INDEX.md            # 子目录用途和关键文件的快速索引
├── cockpit.md          # 每次会话必读入口（≤ 80 行）
├── manifest.md         # 按需读：In flight + Blockers
├── logbook.md          # 极少读：Recently finished + Deferred
│
├── specs/              # 设计文档              （设计时）
├── flight-plans/       # 实施计划              （拆任务时）
│
├── checklists/         # 可复用流程            （命令 + checklist）
├── incident-reports/   # 错题集（禁止"忘了"）  （反复掉的坑）
├── charts/             # 外部资料              （RFC、竞品源码）
│
├── sketches/           # 长期想法              （未启动）
├── safety-reviews/     # 外部 review 反馈      （原文 + 处置）
├── kneeboard/          # 会话 scratch          （只活一个会话）
│
└── landed/             # 归档总仓
    ├── flight-plans/   # 已落地计划
    └── specs/          # 已落地设计
```

**按"何时读它"组织**，不按主题。文件名 / 文件夹名直接告诉 AI 什么场景下查 —— 路由确定，AI 不会浪费 token 加载无关上下文。

## 架构

```mermaid
flowchart LR
    User([👨‍💻 你]) -->|对话| AI[🤖 AI 助手]
    AI <-->|读 / 写| FD[(flightdeck/)]
    AI -.->|/flightdeck:emit-agents-md| AGENTS[AGENTS.md]
    AGENTS -.->|会话启动时读取| OtherAI[其他 AI 工具<br/>Codex · Cursor · Gemini<br/>Copilot · Windsurf]

    subgraph FD [flightdeck/]
        direction TB
        Cockpit[cockpit.md<br/>≤80 行 · 必读]
        Manifest[manifest.md<br/>按需读]
        Logbook[logbook.md<br/>极少读]
        Specs[specs/ · flight-plans/]
        Procedures[checklists/ · incident-reports/]
        Archive[landed/]
        Cockpit --- Manifest --- Logbook
        Specs --- Procedures --- Archive
    end

    style Cockpit fill:#dbeafe,stroke:#2563eb
    style AGENTS fill:#fef3c7,stroke:#d97706
    style FD fill:#f8fafc,stroke:#94a3b8
```

`flightdeck/` 是单一权威源。`AGENTS.md` 是它的衍生视图 —— 给不原生说本协议的工具用的 wire format。

## `cockpit.md` 真实长什么样

唯一必读入口。硬上限 80 行。这是你 AI 每次会话的第一读：

```markdown
# Cockpit — payment-service

**Last updated**: 2026-05-28 by alice (Stripe webhook 重构已上线)
**Active focus**: 稳住 Stripe webhook handler —— 见 incident-reports/ 里失败的边界 case

## Next session

1. 复现 incident-reports/stripe-idempotency.md 第三例的重复事件 bug。
2. 决策：幂等键放 DB 还是 Redis（成本 vs 延迟 trade-off）。
3. 用决策结果更新 flight-plans/2026-05-26-stripe-hardening.md Phase 2。

## Hanging tasks

- (none)
```

整个入场体验就这些。没有 500 行的上下文倾倒。没有 AI 每次都要扫一遍的项目背景介绍。**80 行，硬规则。** 历史性 / 情境性的内容都在更深一层 —— `manifest.md` / `logbook.md` / `specs/` —— 按需打开。

## 为什么需要它

大多数 "AI memory" 系统死于**全部都存**。信号被淹没，最后变垃圾堆 —— 连 AI 自己都懒得读。

**flightdeck 反过来：**

| 纪律 | 强制的内容 |
| --- | --- |
| **严格守门** | 只写那些"会改变未来行为 / 影响决策 / 反复引用"的内容。会话日志、debug dump、"先记一下以后看" —— 拒绝。 |
| **每个目录有生命周期** | `kneeboard/` 只活一个会话。Incident report 反复 3 次升级成项目规则。Spec/flight-plan ship 后归档到 `landed/`。 |
| **权威序** | 多个来源冲突时，协议明确谁说了算。不存在"AI 困惑了"的时刻。 |
| **Landing ritual** | 90% 的会话末分类都是显然的，只有真模糊才调 brainstorming。 |
| **读时分层** | cockpit / manifest / logbook 按"每次都读 / 按需读 / 几乎不读"物理分开。 |

## 设计哲学

> ✨ **Semantic clarity outranks thematic consistency.**（语义清晰高于主题统一）

flightdeck 的航空隐喻是用来**让操作意图更清晰**的 —— 不是统一套主题。两个文件夹（`specs/`、`sketches/`）故意保留中性名字，因为没有更准确的航空对应词。未来新增概念也走同一道考验：词如果配主题但读起来反而模糊，拒绝。

## 安装

### Claude Code &nbsp;<sub>✅ 已测试</sub>

```text
/plugin marketplace add Yuelioi/flightdeck
/plugin install flightdeck@flightdeck-marketplace
```

更新：重跑 `/plugin install`。卸载：`/plugin uninstall flightdeck`。

### 其他 AI 工具 &nbsp;<sub>⚠️ manifest 已到位、行为未验证</sub>

<details>
<summary><b>Codex CLI</b></summary>

```text
/plugins
```

搜 "flightdeck" → 选 → `Install Plugin`。详见 [adapters/codex/](adapters/codex/README.md)。

</details>

<details>
<summary><b>Cursor</b></summary>

在 Cursor Agent chat 里：

```text
/add-plugin flightdeck
```

或者在 plugin marketplace 里搜 "flightdeck"。详见 [adapters/cursor/](adapters/cursor/README.md)。

</details>

<details>
<summary><b>Gemini CLI</b></summary>

```bash
gemini extensions install https://github.com/Yuelioi/flightdeck
gemini extensions update flightdeck   # 更新
```

详见 [adapters/gemini/](adapters/gemini/README.md)。

</details>

### 直接安装 &nbsp;<sub>Claude Code，不走 marketplace</sub>

```powershell
# Windows
git clone https://github.com/Yuelioi/flightdeck.git
cd flightdeck
.\install.ps1
```

```bash
# macOS / Linux
git clone https://github.com/Yuelioi/flightdeck.git
cd flightdeck
./install.sh
```

### 在项目里创建一个 `flightdeck/` 骨架

```powershell
.\install.ps1 -Scaffold minimal     # 只 cockpit.md
.\install.ps1 -Scaffold full        # 完整 11 子目录 + 3 入口文件
```

```bash
./install.sh --scaffold=minimal
./install.sh --scaffold=full
```

## 用法

装完后，项目里只要有 `flightdeck/` 目录，skill 就会自动加载。也可以强制调起来。

### Day 1 —— 给新项目开张

```text
cd my-project
/flightdeck:flightdeck-workflow
```

flightdeck 检测到没目录，问你要不要建，简短问两句（Active focus、Next session 第一条），生成 `flightdeck/cockpit.md`。完事。下次会话开始 SessionStart hook 自动加载。

### 每次会话开始

```
1. 读 flightdeck/cockpit.md         （≤80 行，~5 秒）
2. 跟 `git status` 对账              （branch、未提交、stash）
3. 执行 "Next session" 第一条         （或上浮不一致点先问你）
```

### Slash 命令

| 命令 | 自动加载？ | 用途 |
| --- | --- | --- |
| `/flightdeck:flightdeck-workflow` | ✅ 项目有 `flightdeck/` 时 SessionStart hook 自动注入 | 主协议。也负责 bootstrap：没目录时引导填 cockpit.md。 |
| `/flightdeck:preflight` | — 仅显式 | 长 session 偏题后重新锚回 `cockpit.md` + git 状态。 |
| `/flightdeck:landing` | — 仅显式 | Session 收尾 —— 分类新知识、更新 cockpit、可选 commit。 |
| `/flightdeck:walkaround` | — 仅显式 | 8 类完整性审计 —— 协议漂移检测。 |
| `/flightdeck:emit-agents-md` | — 仅显式 | 从 `cockpit.md` 在 fenced markers 之间重生 `AGENTS.md`。 |

除 `flightdeck-workflow` 外，所有命令都带 `disable-model-invocation: true` —— 只在用户显式打 slash 时触发。

### 路由表 —— 什么情况下进哪个文件夹

Skill 监听对话，自动路由到对应文件夹：

| 你说 / 场景 | Skill 把 AI 导到 |
| --- | --- |
| *"我们上次干到哪了？"* / 会话开始 | `cockpit.md` |
| *"为啥这个迁移挂了？"* | `incident-reports/`（然后开 debug） |
| *"测试怎么跑？"* | `checklists/` |
| *"我们设计一个新 X"* | `specs/` |
| *"把这个拆成任务"* | `flight-plans/` |
| *"这是另一个 AI 的 review 反馈"* | `safety-reviews/`（必须带处置） |
| *"先记一下以后看"* | `sketches/`（或被守门拒绝） |

### 会话结束

说"收尾"之类的，AI 跑 [landing ritual](skills/flightdeck-workflow/exit-ritual.md)：

1. 对新知识应用分类启发式（bug → `incident-reports/`、流程 → `checklists/`、一次性 → 不写）。
2. 更新 `cockpit.md`（`Last updated`、`Next session`）；状态有偏离时更新 `manifest.md` / `logbook.md`。
3. Commit。

下一次会话 —— 哪怕换了 AI、换了开发者 —— 能从上次结束的位置无缝接着干。

## 兼容性

| 工具 | 状态 | Manifest |
| --- | --- | --- |
| Claude Code | ✅ 已测试 | [`.claude-plugin/`](.claude-plugin/) |
| Codex CLI / App | ⚠️ 未测试 | [`.codex-plugin/`](.codex-plugin/) |
| Cursor | ⚠️ 未测试 | [`.cursor-plugin/`](.cursor-plugin/) |
| Gemini CLI | ⚠️ 未测试 | [`gemini-extension.json`](gemini-extension.json) + [`GEMINI.md`](GEMINI.md) |

[`skills/`](skills/) 下的 skill 内容是 **tool-agnostic 的 markdown**。Manifest 只是给各 AI 工具一个发现 skill 的钉子。**"未测试"的意思**：manifest 已到位、上面的安装命令应该能跑，但还没有人端到端验证过 AI 真的会跟着协议走。**带验证日志的 PR 非常欢迎** —— template 见 [.github/PULL_REQUEST_TEMPLATE/manifest-verification.md](.github/PULL_REQUEST_TEMPLATE/manifest-verification.md)。

## 横向对比

给 AI 加持续性这件事有几个相邻方案。flightdeck 占的是这块：

| | flightdeck | [AGENTS.md](https://agents.md) | Cline Memory Bank | OpenSpec | Cursor MDC | Letta Code |
| --- | --- | --- | --- | --- | --- | --- |
| **静态项目规范** | 通过 emit | ✅ 原生 | — | — | ✅ | — |
| **跨 session 接续** | ✅ | — | ✅ | — | — | ✅ |
| **生命周期状态机**（spec → plan → landed） | ✅ | — | — | ✅ | — | — |
| **严格守门**（防 junk-drawer） | ✅ | — | — | — | — | — |
| **错题 / 教训追踪**（含根因纪律） | ✅ | — | — | — | — | — |
| **外部 review disposition** 追踪 | ✅ | — | — | — | — | — |
| **读时分层**（cockpit/manifest/logbook） | ✅ | — | — | — | — | — |
| **工具无关**（markdown + filesystem） | ✅ | ✅ | 部分 | ✅ | 仅 Cursor | — |
| **Skill 自加载**触发 | ✅ | — | ✅ | — | ✅ | — |
| **跨工具触达** | 通过 AGENTS.md emit | 原生 | — | — | — | — |

**一句话总结：**
- **AGENTS.md** 是静态规则的 wire format。flightdeck **emit 进** AGENTS.md，不竞争。
- **Cline Memory Bank** 提供原始 memory 持久化；flightdeck 提供 memory + 生命周期 + 写入纪律。
- **OpenSpec** 是 spec 演化标记的近亲；flightdeck 采纳了它的 `ADDED:` / `MODIFIED:` / `REMOVED:` 约定。
- **Cursor MDC** 是路径范围标签；flightdeck 在 incident-reports / checklists 上加 MDC frontmatter 以兼容 Cursor。
- **Letta Code** 有 skill-library 晋升模式；flightdeck 采纳了它的多准则 incident-report → checklist 晋升门。

flightdeck **态度鲜明**：写入门控先于存储、生命周期先于 memory、同行评审先于合并。如果合你的胃口，会很合。

## FAQ

<details>
<summary><b>这就是一堆 markdown 文件而已？</b></summary>

对 —— 就是这个意思。协议是 `skills/flightdeck-workflow/SKILL.md`，AI 加载它。状态是 `flightdeck/` 下面的纯 markdown。没数据库、没服务器、不用部署任何东西。代码评审里 diff 它、终端里 grep 它、编辑器里改它。AI 工具是协议的**参与者**，不是它的看门人。

</details>

<details>
<summary><b>为啥用航空隐喻？这不就是个编程工具吗？</b></summary>

航空框架反映的是协议实际在做的事：session 生命周期、不确定下的 checklist、事故追踪、operator 之间的交接、带周期性重锚定的受控自主。这些就是航空概念 —— 不是比喻，是结构。

框架自带护栏：**"语义清晰高于主题统一"**。两个文件夹（`specs/`、`sketches/`）保留中性名，因为没有更准确的航空词。隐喻是工具，不是主题。

</details>

<details>
<summary><b>80 行 cockpit 天花板真的重要吗？</b></summary>

是。这个天花板是**认知负载工程**，不是风格。cockpit.md 每次会话都被人和 AI 一起读。80 行能一屏装下，token 消耗 < 1k。没有这个天花板，board 风格文件会涨到 300-500 行，AI 光定位就要烧上下文，人就直接不读了。

撞到天花板就强制做真实决策：这条内容配得上待在入口文件吗？还是该挪到 `manifest.md`（在飞工件）/ `logbook.md`（历史）/ 干脆移出？

</details>

<details>
<summary><b>flightdeck 和直接用 AGENTS.md 区别在哪？</b></summary>

[AGENTS.md](https://agents.md) 是 Linux Foundation 主导的跨工具 AI 指令标准 —— 2026 年中已被 6 万+ 仓库采用，控制实验显示运行时间下降 28.6%、token 消耗下降 16.6%。如果你只需要"给 AI 一份静态项目规范"，**单用 AGENTS.md 就够**，flightdeck 是杀鸡用牛刀。

flightdeck **架在** AGENTS.md 之上，不是替代：

| 关注点 | 单用 AGENTS.md | flightdeck |
| --- | --- | --- |
| 静态项目规范 / 风格指南 | ✅ | （用 AGENTS.md） |
| 跨 session 接续（cockpit、交接） | — | ✅ |
| 生命周期状态机（spec → plan → landed） | — | ✅ |
| 防止 junk-drawer 堆积的写入门控 | — | ✅ |
| 错题本（出过什么 bug、根因） | — | ✅ |
| 跨工具触达 | 原生 | 通过 `/flightdeck:emit-agents-md` |

`/flightdeck:emit-agents-md` 从 `flightdeck/cockpit.md` 重新生成 AGENTS.md 里的 fenced block。`cockpit.md` 维护一份；读 AGENTS.md 的 AI 工具（Codex CLI、Copilot、Cursor、Windsurf、Continue、Cody）都能看到最新项目状态。

</details>

<details>
<summary><b>我不用 Claude Code 怎么办？</b></summary>

[`skills/`](skills/) 下的 skill 内容是 tool-agnostic 的纯 markdown。Codex / Cursor / Gemini 的 manifest 已经到位，但行为还没人端到端验证。当下最有价值的贡献就是去验证其中一个 —— 详见[贡献](#贡献)。

在那之前，[`/flightdeck:emit-agents-md`](skills/emit-agents-md/SKILL.md) 命令是通向任意读 `AGENTS.md` 的工具的桥梁。

</details>

<details>
<summary><b>这和写一份 CLAUDE.md / 项目笔记有啥不同？</b></summary>

静态规则文件（CLAUDE.md、项目笔记）是**只追加的知识**。flightdeck 是**带生命周期门的状态机**：

- 新错误 → `incident-reports/`，强制根因分析（禁用短语："忘了"、"不小心"）。
- 错误反复 3 次 → 晋升门触发，你决定是否升级到 `checklists/`。
- Checklist 被无视 → 再次晋升到项目 agent 规则。
- Spec 落地 → 移到 `landed/`，权威性让给当前状态。

正是这套生命周期防止了静态规则文件长期必然滑向的"垃圾抽屉"失败模式。

</details>

<details>
<summary><b>为啥不用向量 embedding / RAG 索引代码？</b></summary>

flightdeck 解决的问题不一样。向量检索给你**相似**内容；flightdeck 给你带明确路由的**操作相关**内容。你不会想要相似检索把一个已废弃的 incident report 和三个无关的一起翻出来 —— 你要的是当前 `cockpit.md`，没别的，除非你主动问。

flightdeck 在 embedding 做不到的方面也更耐用：纯文本，能扛过模型升级、厂商切换、AI 工具切换、人在 code review 里直接改文件。

</details>

## 文档

| 文件 | 覆盖内容 |
| --- | --- |
| [skills/flightdeck-workflow/SKILL.md](skills/flightdeck-workflow/SKILL.md) | AI 加载的入口 —— 协议、权威序、生命周期、设计哲学 |
| [skills/flightdeck-workflow/folder-semantics.md](skills/flightdeck-workflow/folder-semantics.md) | 每个文件夹的语义和职责；minimal vs full；future expansion slots |
| [skills/flightdeck-workflow/templates.md](skills/flightdeck-workflow/templates.md) | incident-report / checklist / sketch / safety-review / cockpit 模板含 frontmatter 规则 |
| [skills/flightdeck-workflow/exit-ritual.md](skills/flightdeck-workflow/exit-ritual.md) | Landing ritual —— 分类启发式、red flags、晋升门 |
| [skills/preflight/SKILL.md](skills/preflight/SKILL.md) | `/flightdeck:preflight` —— 显式入场 ritual |
| [skills/landing/SKILL.md](skills/landing/SKILL.md) | `/flightdeck:landing` —— 显式 landing ritual |
| [skills/walkaround/SKILL.md](skills/walkaround/SKILL.md) | `/flightdeck:walkaround` —— 8 类完整性审计 |
| [skills/emit-agents-md/SKILL.md](skills/emit-agents-md/SKILL.md) | `/flightdeck:emit-agents-md` —— AGENTS.md 重生 |
| [TEST_PLAN.md](TEST_PLAN.md) | RED-GREEN-REFACTOR 测试状态 |
| [MIGRATION.md](MIGRATION.md) | workshop → flightdeck 升级笔记 |
| [CHANGELOG.md](CHANGELOG.md) | 按版本的变更记录 |

## 贡献

### 端到端验证一个 manifest

Codex / Cursor / Gemini 的 manifest 都到位了，但**行为没测过**。当下最有价值的贡献：

1. 在其中一个工具上装 flightdeck。
2. 在有 `flightdeck/` 的项目里跑一段短 session。
3. 按 [release-gate 场景](flightdeck/specs/2026-05-23-v1.0-release-gate.md) 验证 AI 真的按 entry / triggers / landing 走。
4. 提 PR 带上对话记录，把矩阵从 ⚠️ 翻到 ✅。Template：[.github/PULL_REQUEST_TEMPLATE/manifest-verification.md](.github/PULL_REQUEST_TEMPLATE/manifest-verification.md)。

### Skill 本身的改进

按 **RED-GREEN-REFACTOR** 纪律：没跑测试不准改。详见 [TEST_PLAN.md](TEST_PLAN.md)。

你能开的最有价值的 issue：**一段 AI 钻空子绕过协议的对话记录**。Skill 没盖到的 rationalization 是最高信号的贡献。

## Roadmap

v1.0 ship 状态见 [TEST_PLAN.md](TEST_PLAN.md)。v1.0 之后：

| | 目标 |
| --- | --- |
| **v1.1+** | 可选文件夹 —— `briefing/`（领域背景）、`blackbox/`（原始 session log）、`crew-handover/`（跨 AI 交接）、`experiments/`（长期 probe）。v1.0 重命名时已推迟以控制范围；待实际使用证明需求再加。 |
| **Continuance benchmark** | 给任意 AI 一个中途断片的项目，让它"接着干"，量化恢复能力。 |
| **Synthesis / 压缩** | 把大量归档 specs 压成主题复盘，不丢决策历史。 |
| **INDEX 自动化** | 可选 hook 保持 `INDEX.md` 的 AUTO 段同步。 |
| **端到端验证 Codex / Cursor / Gemini**（欢迎 PR —— manifest 已就位）。 |
| **MCP server** 把 `flightdeck/` 暴露给 MCP-aware client。 |

## 致谢

flightdeck 站在这些项目的肩上：

- **[AGENTS.md](https://agents.md)** —— Linux Foundation 跨工具 AI 项目指令标准。flightdeck emit 进 AGENTS.md，把它当作 wire format。
- **[OpenSpec](https://github.com/openspec/openspec)** —— spec 演化标记（`ADDED:` / `MODIFIED:` / `REMOVED:`）直接来自 OpenSpec 约定。
- **[Cursor MDC](https://docs.cursor.com)** —— incident-reports / checklists 上的路径范围 frontmatter（`globs:` / `alwaysApply:`）用于 Cursor 兼容。
- **[Letta Code](https://github.com/letta-ai/letta)** —— skill-library 晋升模式启发了多准则 incident-report → checklist 门。
- **[superpowers](https://github.com/anthropic-experimental/superpowers)** —— 有向图协议风格、`brainstorming` / `writing-plans` skill 约定。
- **[Cline Memory Bank](https://docs.cline.bot/improving-your-prompting-skills/custom-instructions-library/cline-memory-bank)** —— 原始"AI 持久化 memory"模式，催生了 flightdeck 更严格的写入门控。
- **[Roo Boomerang](https://github.com/RooCodeInc)** —— subagent 上下文窗口求生模式，标记给 v1.x。

外加维护者自己的痛 —— 每个文件夹、每条规则、每个门都是因为前一段 session 烧了前一个上下文窗口、丢了前一个洞察才有的。

## License

[MIT](LICENSE) © 月离 (Yuelioi)

---

<div align="center">

如果 flightdeck 帮你省下过一个上下文窗口，[给个 star](https://github.com/Yuelioi/flightdeck/stargazers) —— 让更多人能找到它。

</div>
