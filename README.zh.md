# workshop

🇬🇧 **English users**: see [README.md](README.md) for the English version.

> AI 协作的持久工作台协议。

AI 助手在两次对话之间会失忆。`workshop` 用一套目录约定 + skill 给它一个可持续的工作台 —— 下一次会话能接着上一次干，知道你之前在做什么、为什么、下一步该干嘛。

## 它是什么

一个 `workshop/` 目录布局，AI 按约定读和写：

```
workshop/
├── board.md            # 当前状态 —— 先读它，最后改它
│
├── specs/              # 设计文档              （设计时）
├── plans/              # 实施计划              （拆任务时）
├── playbooks/          # 可复用流程            （命令 + checklist）
├── scars/              # 错题集（禁止"忘了"）  （反复掉的坑）
├── reference/          # 外部资料              （RFC、竞品源码）
│
├── sketches/           # 长期想法              （未启动）
├── critiques/          # 外部 review 反馈      （原文 + 处置）
└── wip/                # 会话 scratch          （只活一个会话）
```

**按"何时读它"组织**，不按主题。文件夹名直接告诉 AI 什么场景下查。

## 为什么需要它

大多数 "AI memory" 系统死于**全部都存**。信号被淹没，最后变垃圾堆 —— 连 AI 自己都懒得读。

`workshop` 反过来：

- **严格守门**：只写那些"会改变未来行为 / 影响决策 / 反复引用"的内容。
- **每个目录有生命周期**：`wip/` 只活一个会话；scar 反复 3 次升级成项目规则；spec/plan ship 后归档到 `finish/`。
- **权威序**：多个来源冲突时，协议明确谁说了算。
- **Exit ritual**：90% 的会话末分类都是显然的，只有真模糊才调 brainstorming。

完整规范见 [`skills/workshop-workflow/SKILL.md`](skills/workshop-workflow/SKILL.md)（英文，AI 实际加载的版本）。

## 安装

### Claude Code ✅ 已测试

在任意 Claude Code session 里：

```text
/plugin marketplace add Yuelioi/workshop
/plugin install workshop@workshop-marketplace
```

更新：重跑 `/plugin install`。卸载：`/plugin uninstall workshop`。

### Codex CLI ⚠️ manifest 已到位、行为未验证

```text
/plugins
```

搜 "workshop" → 选 → `Install Plugin`。

### Cursor ⚠️ manifest 已到位、行为未验证

在 Cursor Agent chat 里：

```text
/add-plugin workshop
```

或者在 plugin marketplace 里搜 "workshop"。

### Gemini CLI ⚠️ manifest 已到位、行为未验证

```bash
gemini extensions install https://github.com/Yuelioi/workshop
```

更新：

```bash
gemini extensions update workshop
```

### 备选 —— 直接 skill 安装（仅 Claude Code，不走 marketplace）

适合不想用 plugin marketplace 的用户，直接复制到 `~/.claude/skills/`：

```powershell
# Windows
git clone https://github.com/Yuelioi/workshop.git
cd workshop
.\install.ps1
```

```bash
# macOS / Linux
git clone https://github.com/Yuelioi/workshop.git
cd workshop
./install.sh
```

### 在项目里创建一个 `workshop/` 骨架

```powershell
.\install.ps1 -Scaffold minimal     # 只 board.md
.\install.ps1 -Scaffold full        # 完整 10 子目录
```

```bash
./install.sh --scaffold=minimal
./install.sh --scaffold=full
```

## 用法

装完后，项目里只要有 `workshop/` 目录，skill 就会自动加载。也可以强制调起来。

### Day 1 —— 给新项目开张

Claude Code 里，`cd` 进你的项目，输入：

```text
/workshop:workshop-workflow
```

Skill 检测到没 `workshop/`，问你要不要建，然后简短问两句（Active focus、Next session 第一条），生成 `workshop/board.md`。下次会话开始 SessionStart hook 自动加载，不用再敲 slash。

**其他工具 / 脚本化安装**：clone 本仓库跑 `install.sh --scaffold=minimal`，或者手动 copy `scaffolds/minimal/workshop/` 到你的项目。

### 每次会话开始时 AI 干嘛

```
1. 读 workshop/board.md
2. 跟 `git status` 对账（branch、未提交变更、stash）
3. 执行 "Next session" 第一条；状态不一致就先问你
```

### Slash 命令

| 命令 | 自动加载？ | 干啥的 |
| --- | --- | --- |
| `/workshop:workshop-workflow` | 是 —— `workshop/` 存在时由 SessionStart hook 自动注入 | 强制加载主协议。**也负责 bootstrap**：项目还没 `workshop/` 时，问你要不要建，然后引导填 `board.md`。之后跑 entry checklist。 |
| `/workshop:session-enter` | 否 —— 仅显式 | 长 session 偏题后重新锚回：重读 `board.md`，跟 git status / branch / stash / 提交时间线对账，上浮过期 `wip/`。 |
| `/workshop:session-exit` | 否 —— 仅显式 | Session 收尾：用 (a)–(h) 启发式分类新知识，更新 board，跑生命周期迁移，scar→playbook 晋升门触发时提示，按需 commit。 |
| `/workshop:emit-agents-md` | 否 —— 仅显式 | 从 `workshop/board.md` 重生 repo 根的 `AGENTS.md`（对接消费 AGENTS.md 的工具：Codex CLI、Copilot、Cursor、Windsurf、Continue、Cody）。`board.md` 改动后跑。 |
| `/workshop:doctor` | 否 —— 仅显式 | 全仓 8 类协议漂移审计（frontmatter 缺失、stale wip、断链、board ↔ folder 不一致、stale Blockers、Recently finished 超长、AGENTS.md 漂移、孤儿 scar）。报告 CRITICAL / WARNING / INFO，**不自动修**。 |

除 `workshop-workflow` 外，所有命令都带 `disable-model-invocation: true` —— 只在用户显式打 slash 时触发，不会从对话上下文自动 invoke。

### 路由表 —— 什么情况下进哪个文件夹

Skill 监听对话，自动路由到对应文件夹：

| 你说 / 场景 | Skill 把 AI 导到 |
| --- | --- |
| "我们上次干到哪了？" / 会话开始 | `workshop/board.md` |
| "为啥这个迁移挂了？" | `workshop/scars/`（然后开 debug） |
| "测试怎么跑？" | `workshop/playbooks/` |
| "我们设计一个新 X" | `workshop/specs/` |
| "把这个拆成任务" | `workshop/plans/` |
| "这是另一个 AI 的 review 反馈" | `workshop/critiques/`（必须带处置） |
| "先记一下以后看" | `workshop/sketches/`（或者守门拒绝） |

### 会话结束

说"收尾"之类的，AI 跑 [exit ritual](skills/workshop-workflow/exit-ritual.md)：

```
1. 对新知识应用分类启发式
   （bug → scars/、流程 → playbooks/、一次性 → 不写）
2. 更新 board.md（Last updated、Next session、In flight）
3. Commit
```

效果：下一次会话 —— 哪怕换了 AI、换了开发者 —— 能从上次结束的位置无缝接着干。

## 兼容性

| 工具 | 状态 | Manifest |
| --- | --- | --- |
| Claude Code | ✅ 已测试 | [`.claude-plugin/`](.claude-plugin/) |
| Codex CLI / App | ⚠️ 未测试 | [`.codex-plugin/`](.codex-plugin/) |
| Cursor | ⚠️ 未测试 | [`.cursor-plugin/`](.cursor-plugin/) |
| Gemini CLI | ⚠️ 未测试 | [`gemini-extension.json`](gemini-extension.json) + [`GEMINI.md`](GEMINI.md) |

[`skills/`](skills/) 下的 skill 内容是 **tool-agnostic 的 markdown**。Manifest 只是给各 AI 工具一个发现 skill 的钉子。**"未测试"的意思**：manifest 已到位、上面的安装命令应该能跑，但还没有人端到端验证过 AI 真的会跟着协议走。带验证日志的 PR 非常欢迎。

## 文档

skill 内容只有英文版（AI 加载用，所有跨语言项目都能命中触发器）：

- [SKILL.md](skills/workshop-workflow/SKILL.md) —— AI 加载的入口
- [folder-semantics.md](skills/workshop-workflow/folder-semantics.md) —— 每个文件夹的语义和职责
- [templates.md](skills/workshop-workflow/templates.md) —— scar / sketch / critique / board / INDEX 模板
- [exit-ritual.md](skills/workshop-workflow/exit-ritual.md) —— 会话结束决策树
- [session-enter SKILL.md](skills/session-enter/SKILL.md) —— 显式 `/workshop:session-enter` slash 命令
- [session-exit SKILL.md](skills/session-exit/SKILL.md) —— 显式 `/workshop:session-exit` slash 命令
- [TEST_PLAN.md](TEST_PLAN.md) —— RED-GREEN-REFACTOR 测试状态（当前 v0.x，pre-test）

## 贡献

### 验证某个 AI 工具的 manifest 真能用

Codex / Cursor / Gemini 的 manifest 都到位了，但**行为没测过**。当下最有价值的 PR：在其中一个工具上装一遍，在有 `workshop/` 的项目里跑一段短会话，确认 AI 真的按 entry / triggers / exit 走。提 PR 带上对话记录，把兼容性矩阵从 ⚠️ 未测试 翻到 ✅ 已测试。

### Skill 本身的改进

按 RED-GREEN-REFACTOR 纪律：**没跑测试不准改**。流程见 [TEST_PLAN.md](TEST_PLAN.md)。

如果你发现 skill 没盖到的 rationalization（AI 钻空子绕过协议），开 issue 贴对话记录 —— 这是最有价值的贡献。

## 为什么不直接用 AGENTS.md？

[AGENTS.md](https://agents.md) 是 Linux Foundation 主导的跨工具 AI 指令标准，2026 年中已被 6 万+ 仓库采用 —— 有控制实验显示运行时间下降 28.6%、token 消耗下降 16.6%。如果你只需要"给 AI 一份静态的项目规范"，单用 AGENTS.md 已经够用；workshop 是杀鸡用牛刀。

workshop **架在** AGENTS.md 之上，不是替代品。两者解决不同问题：

| 关注点 | 单用 AGENTS.md | Workshop |
| --- | --- | --- |
| 静态项目规范 / 风格指南 | ✓ | （用 AGENTS.md） |
| 跨 session 接续（看板、交接） | — | ✓ |
| 生命周期状态机（spec → plan → done） | — | ✓ |
| 防止 junk-drawer 堆积的写入门控 | — | ✓ |
| 错题本（出过什么 bug、根因） | — | ✓ |
| 外部 critique 的 disposition 追踪 | — | ✓ |
| 跨工具触达 | 原生 | 通过 `/workshop:emit-agents-md` |

workshop **emit** 到 AGENTS.md —— `/workshop:emit-agents-md` 从 `workshop/board.md` 重新生成 AGENTS.md 里的 workshop fenced block。它不替代或竞争 AGENTS.md：workshop 是操作协议，AGENTS.md 是 wire format。

如果你用的工具原生消费 AGENTS.md（Codex CLI、Copilot、Cursor、Windsurf、Continue、Cody）：workshop 的 emitter 就是桥梁。`board.md` 维护一份；读 AGENTS.md 的 AI 工具能看到最新项目状态。

## Roadmap

v1.0 release gate 见 [TEST_PLAN.md](TEST_PLAN.md)。v1.0 之后：

- **Continuance benchmark**：给任意 AI 一个中途断片的项目，让它"接着干"，量化恢复能力。
- **Synthesis / 压缩**：把大量归档 specs 压成主题复盘。
- **INDEX 自动化**：可选 hook 保持 `INDEX.md` 的 AUTO 段同步。
- **端到端验证 Codex / Cursor / Gemini**（欢迎 PR —— manifest 已就位）。

## License

MIT，见 [LICENSE](LICENSE)。

