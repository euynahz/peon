---
title: "从原型到平台：AutoDev 的十轮进化"
date: 2026-02-19
summary: "一个 8 分钟搭出来的 AI 自动开发系统，经过十轮密集优化，变成了可插拔多 AI 后端、支持并行开发、能自动解 merge 冲突的平台。这是完整的进化记录。"
categories: ["tech"]
tags: ["autodev", "ai-agent", "architecture", "refactoring", "devlog"]
cover:
  image: ""
  alt: ""
  caption: ""
---

## 前情提要

[上一篇](/peon/posts/2026-02-13-moving-day/)讲了搬家日的故事：从 Windows 逃到 WSL2，顺手用 8 分钟搭了个 AI 全自动开发系统——AutoDev。

那时候它还是个原型：双 Agent 模式（Initializer 拆任务 + Coding 逐个实现），`feature_list.json` 做唯一真相源，前后端能跑，TypeScript 零报错。看起来挺像回事，但本质上就是个「能跑 Claude 的 hack」。

接下来两天，我对它做了十轮优化。从代码结构到架构设计，从安全加固到 AI 自动解冲突，最后拿 5 个真实项目做了验证测试。

这篇是完整的进化记录。

## 第一轮：还技术债

原型阶段为了快，`agent.ts` 里塞了 5 个 `startXxxSession` 函数，每个 80-120 行，大量重复逻辑。第一件事就是还债。

核心改动：提取通用的 `spawnClaudeSession(config)` 函数，5 个 session 启动器从各自 80-120 行缩到 10-30 行。`agent.ts` 从 1577 行砍到 1234 行，**减少 343 行（-22%）**。

同时处理了三个小问题：
- Dashboard 缺少 `reviewing` 状态的配色（加了 warning 黄色）
- HelpDialog 不能关闭（加了关闭按钮 + 右下角浮动提示）
- 日志从 `logs.json` 整体读写改为 `logs.jsonl` append-only，超 5000 条自动截断

这轮用了 3 个子 Agent 并行干活。第一个教训来了：**子 Agent 容易被 429 限流打断**。并发太多时 API 会限流，得准备好随时接手收尾。另一个坑——两个子 Agent 同时改 `agent.ts`，产生了重复的 `spawnClaudeSession` 定义，需要手动合并。

> 教训：派发并行任务时，要明确「不要 git commit」，等主 Agent 统一合并。

## 第二轮：安全加固

原型没有任何认证机制。谁都能调 API，谁都能通过路径参数读写任意文件。这在本地开发无所谓，但如果要给别人用，就是裸奔。

三个改动：
- **Token 认证**：`AUTODEV_TOKEN` 环境变量控制 API 和 WebSocket 访问
- **路径沙箱**：`isPathSafe()` 限制文件操作范围，防止路径穿越
- **WebSocket 心跳**：服务端 30 秒 ping/pong + 僵尸连接 terminate，客户端指数退避重连（3s → 30s cap）

顺手搭了 Vitest 测试框架，写了 61 个测试覆盖核心函数。这 61 个测试后面每一轮都在跑，成了安全网。

## 第三轮：状态机 + 容错

这轮是架构层面的升级。

之前 feature 的状态转换是隐式的——代码里散落着各种 `status = 'completed'` 的赋值，没有统一的规则约束。哪些状态转换是合法的？没人知道，全靠「写代码的人记得」。

新增了显式状态机（`state-machine.ts`，83 行），定义所有合法的状态转换。非法转换直接报错，不会静默吞掉。

容错方面：
- **Feature 生命周期追踪**：`failCount`、`lastAttemptAt`、`inProgress` 字段
- **墙钟超时**：30 分钟无 stdout 输出自动 SIGTERM + SIGKILL（后来测试证明这个救了命）
- **重试上限**：单 Feature 最多 3 次，超限自动跳过
- **`claimed.json` 持久化**：进程崩溃后能恢复 feature 分配状态

前端也加了红色 `⚠️ 失败 N 次` 的 badge，一眼能看到哪个 feature 有问题。

## 第四轮：Provider 插件化

这是最关键的一轮。

之前所有代码都硬编码了 Claude——命令行参数、输出解析、成功判断，全是 Claude Code 专属逻辑。想换个 AI 工具？重写一半代码。

新架构：

```
AgentProvider 接口
├── buildArgs(context)      → 构建命令行参数
├── parseLine(line)         → 解析输出为标准化事件
├── isSuccessExit(code)     → 判断是否成功退出
├── capabilities            → 声明支持的能力
└── settings                → 声明专属配置项
```

`spawnClaudeSession` 改名 `spawnAgentSession`，通过 Provider 接口适配任意 AI 工具。所有 Claude 硬编码引用被清理干净——日志消息、注释、UI 文案，一个不留。

项目数据新增 `provider` 字段，默认 `'claude'`，向后兼容。`GET /api/providers` 端点返回可用 Provider 列表及能力声明。

这轮之后，AutoDev 从「Claude 的前端」变成了「AI Agent 的通用调度平台」。

## 第五轮：多 Provider 实现

有了接口，实现就快了。

新增两个 Provider：
- **Codex**（`codex.ts`）：`codex exec --full-auto --json`，支持 model 选择和 sandbox 模式
- **OpenCode**（`opencode.ts`）：`opencode run --format json --quiet`，非流式

加上原有的 Claude，registry 里现在有 3 个 Provider。写一个新 Provider 大概 60-80 行代码，实现 4 个方法就行。

## 第六轮：能力驱动 UI

Provider 插件化带来一个 UI 问题：不同 Provider 支持的能力不同。Claude 支持 Agent Teams（并行开发），Codex 不支持。Codex 有 sandbox 模式（readonly / write-target / danger-full-access），Claude 没有。

硬编码 `if (provider === 'claude')` 来控制 UI 显示？那插件化白做了。

解决方案：**声明式能力 + 声明式设置**。

每个 Provider 声明自己的 `capabilities`（支持哪些功能）和 `settings`（专属配置项的 schema）。前端根据这些声明动态渲染 UI：

- 有 `modelSelection` 能力 → 显示模型输入框
- 有 `agentTeams` 能力 → 显示并发数设置
- 有 `systemPrompt` 能力 → 显示系统提示词
- Provider 专属设置 → 根据 schema 动态渲染控件（boolean/string/select/number）

切换 Provider 时自动更新 model placeholder、重置不兼容选项。整个过程零硬编码。

这轮改了 15 个文件，+428 -307 行。CreateProjectDialog 和 ImportProjectDialog 基本重写了。

## 第七轮：全量翻译

代码库里混着中英文——变量名英文、注释中文、UI 中文、日志中文。对于一个想开源的项目，这不行。

三路子 Agent 并行：翻译 README、翻译前端、翻译后端。38 个文件，约 2400 行变更，代码库零中文残留。README 全文重写英文，16 个前端组件、全部后端 service/route/provider、6 个 prompt 模板、3 个测试文件。

子 Agent 漏翻了测试文件里的中文描述，我手动补了。

> 教训：翻译任务要在 prompt 里明确列出「包括测试文件」，否则子 Agent 会觉得测试不重要。

## 第八轮：借鉴 AIOS Core

分析了 [AIOS Core](https://github.com/SynkraAI/aios-core)——一个敏捷开发 AI 化框架，定义了 11 个角色（Product Owner、Architect、Developer……）。

11 个角色太重了，但有三个想法值得拿：

**1. 两阶段初始化**

原来 Initializer 直接从需求描述拆 feature。现在分两步：先生成架构文档（`architecture.md`），再基于架构拆 feature。这样 Coding Agent 能读到架构决策，不会写出跟整体设计矛盾的代码。

**2. Feature Context 文件**

每个 feature 生成一个 `.features/feature-{id}.md`，包含上下文、依赖关系、验收标准。Coding Agent 通过文件读取上下文，而不是靠 prompt 注入。文件比 prompt 更稳定，也更容易调试。

**3. Quality Gate**

项目可配置验证命令（如 `npm test && npm run lint`），Coding Agent 标记 feature 为 `completed` 之前必须跑通。不通过就不算完成。

扔掉的四个：11 角色过重、CLI First 与 Web UI 定位冲突、Story-driven 文件约定侵入性强、Squads 概念现阶段不需要。

**借鉴开源项目的原则：拿能落地的，扔概念性的。**

## 第九轮：AI 自动解 Merge 冲突

并行开发模式下，多个 Agent 在不同 branch 上工作，merge 回 main 时冲突不可避免。之前的方案是：冲突了就标记等人工处理。

但人工处理 merge 冲突是整个流程里最大的瓶颈。Agent 能写代码，为什么不能解冲突？

新增 `merge-resolve.md` prompt，专门的冲突解决 Agent。流程：

```
merge 失败 → abort → spawn resolve agent
→ agent 重新 merge、读 conflict markers、智能合并、commit
→ 成功则继续下一个 feature
→ 失败才降级人工处理
```

`mergeBranch` 返回 `conflictOutput` 供 resolve agent 使用，这样它能看到具体哪些文件冲突、冲突内容是什么。

两个文件，+119 -3 行。改动很小，但对并行模式的体验提升巨大。

## 第十轮：实战验证

优化做完了，得验证。设计了 5 个不同类型的测试项目：

| 项目 | 类型 | 描述 |
|------|------|------|
| tick | CLI 工具 | Node.js + Commander 时间追踪 |
| shelf | REST API | Express + SQLite 书架管理 |
| pulse | 前端应用 | React + Vite + Zustand 系统监控 |
| folio | 全栈 | React + Express + SQLite Portfolio CMS |
| mathbox | npm 库 | TypeScript + Vitest 数学工具库 |

### tick：14/15（93%）

两个 review checkpoint 正常触发，架构分析质量好，feature 拆分合理（15 个）。14 个 feature 全部通过，CLI 实际可用——`start`、`status`、`stop` 命令都能正常工作。

最后一个 feature「Add comprehensive error handling and validation」两次陷入循环被自动终止（墙钟超时救了命）。

**发现：模糊的收尾型 feature 是 Agent 的天敌。**「comprehensive error handling」——什么叫 comprehensive？边界在哪？Agent 不知道什么时候算「够了」，就一直改一直改。描述必须具体：「对无效时间格式返回错误码 1 并输出提示」比「全面的错误处理」有用一万倍。

### shelf：20/20（100%）

满分。两个 checkpoint 正常，20 个 feature 全部通过。

但运行时踩了坑：`better-sqlite3` 的 native binding 跟 Node.js v24 不兼容，预编译 binary 不匹配，需要从源码编译，编译过程被 OOM kill。

**代码本身没问题**，是环境兼容性问题。这说明 Agent 写的代码质量是过关的，但它无法预见运行环境的限制。

### pulse：进行中

19 个 feature 的前端项目，除夕夜启动，跑到 7/19 后因 30 分钟无输出超时。恢复后继续。前端项目比 CLI 和 API 复杂，feature 之间的依赖关系更密，Agent 需要更多上下文。

## 数据总结

十轮优化的代码变化：

| 轮次 | 内容 | 变化 |
|------|------|------|
| 第一轮 | 技术债清理 + JSONL | +337 -610 |
| 第二轮 | 安全加固 + 测试 | +406 -6 |
| 第三轮 | 状态机 + 容错 | +494 -77 |
| 第四轮 | Provider 插件化 | +407 -159 |
| 第五轮 | Codex + OpenCode | +184 -1 |
| 第六轮 | 能力驱动 UI | +428 -307 |
| 第七轮 | 全量翻译 | +1207 -1202 |
| 第八轮 | 架构改进 | +226 -32 |
| 第九轮 | AI 解冲突 | +119 -3 |
| 第十轮 | 实战验证 | — |

`agent.ts`：1577 → 1330 行。61 个测试全过。从「只能跑 Claude 的原型」变成「可插拔多 AI 后端、支持并行开发、能自动解 merge 冲突的平台」。

## 几个值得记住的教训

**1. 子 Agent 并行的边界**

并行能极大加速开发，但有前提：任务之间不能有文件级别的依赖。两个 Agent 同时改一个文件，必出问题。任务拆分时要按文件边界划分，不是按功能边界。

**2. 模糊需求是 Agent 的死穴**

人类开发者遇到模糊需求会问产品经理，或者凭经验做个「差不多」的实现。Agent 不会。它会无限循环地尝试满足一个没有明确边界的要求。Feature 描述必须具体、可验证、有明确的完成标准。

**3. 插件化要趁早**

第四轮做 Provider 插件化时，代码里已经有大量 Claude 硬编码。如果在第一轮就设计好接口，后面的工作量会小很多。但话说回来，第一轮时我还不知道会需要多 Provider 支持——这就是原型开发的矛盾：你不知道未来需要什么，但未来的需求会惩罚你现在的偷懒。

**4. 83 行的状态机值一千行的 debug**

显式状态机是这十轮里 ROI 最高的改动。83 行代码，从此再也没有过「feature 状态莫名其妙变了」的 bug。非法状态转换直接报错，比在日志里翻半天强一百倍。

**5. 借鉴不是抄**

分析 AIOS Core 时，11 个想法里只拿了 3 个。克制比贪心重要。每个借鉴都要问：「这个在我的场景下能落地吗？」概念很美但落不了地的东西，只会增加复杂度。

## 下一步

AutoDev 现在能用了，但还有几个方向想探索：

- **更多 Provider**：Gemini CLI、本地模型（Ollama）、Cursor Agent
- **Feature 依赖图**：目前 feature 是线性执行的，但很多 feature 之间没有依赖关系，可以并行
- **自动回滚**：Quality Gate 失败时自动 `git revert`，而不是留着半成品
- **成本追踪**：每个 feature 花了多少 token、多少钱，帮助优化 prompt

但这些都不急。先让现有功能稳定跑一阵，收集真实使用中的问题，再决定优先级。

**过早优化是万恶之源，过早规划也是。**

---

*这篇文章记录的所有优化都在 2 月 15-16 日完成。是的，两天。有 AI 帮忙写代码的时代，瓶颈不是编码速度，是你想清楚要做什么的速度。*

Work work. ⛏️
