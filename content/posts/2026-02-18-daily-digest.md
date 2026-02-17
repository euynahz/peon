---
title: "📰 每日资讯 | 2026-02-18"
date: 2026-02-18
categories:
  - digest
tags:
  - Anthropic
  - Claude
  - OpenAI
  - Codex
  - Qwen
  - Stratechery
  - AI 安全
  - 开发工具
  - 基础设施
draft: false
---

## 🔥 头条

### Anthropic 发布 Claude Sonnet 4.6：性价比之王

**来源：Anthropic / Hacker News（740 赞）**

Anthropic 发布了 Claude Sonnet 4.6，这是迄今为止最强的 Sonnet 模型。核心亮点：

- **编码能力大幅提升**：Claude Code 用户 70% 的场景更偏好 Sonnet 4.6 而非 Sonnet 4.5，甚至 59% 的场景优于 Opus 4.5
- **100 万 token 上下文窗口**（beta），可容纳整个代码库或数十篇论文
- **计算机使用能力**在 OSWorld 基准上持续进步，已接近人类水平
- **价格不变**：仍为 3/15 美元每百万 token，性价比极为惊人
- **安全评估**显示该模型「具有温暖、诚实、亲社会的特质，无重大对齐风险」

> 💬 这次发布的核心信息很明确：Sonnet 级别的模型已经能做到之前只有 Opus 才能做的事。对于日常开发者来说，这意味着不再需要为「够不够聪明」纠结——Sonnet 4.6 基本够用了。Anthropic 的模型迭代速度令人敬畏。

---

### Anthropic 与五角大楼的 AI 冲突升级

**来源：The Rundown AI / Axios**

五角大楼据报正考虑将 Anthropic 列为「供应链风险」——这一标签通常只留给外国对手。原因是 Anthropic 对 Claude 在军事用途上设置了限制。

- 若该标签落实，所有美国国防承包商将被迫切断与 Anthropic 的合作
- 国防官员要求 AI 用于「所有合法用途」，Anthropic 坚持不允许用于监控美国公民或自主武器
- Claude 目前是五角大楼保密系统中唯一部署的 AI，据报曾通过 Palantir 参与抓捕委内瑞拉前总统 Maduro

> 💬 这是 AI 治理领域的标志性事件。一边是前沿实验室坚持负责任 AI 的底线，另一边是军方要求不受限制地使用技术。Anthropic 的立场值得尊敬，但这场博弈的结果将深远影响整个行业的军事 AI 合作边界。

---

## 🤖 AI 与模型

### 阿里巴巴发布 Qwen 3.5：原生多模态 Agent 模型

**来源：TLDR AI / Qwen 官方博客**

Qwen3.5-397B-A17B 是 Qwen 3.5 系列首个模型：

- 采用**混合架构**：线性注意力 + 稀疏专家混合（MoE），总参数 3970 亿，每次推理仅激活 170 亿
- **原生视觉语言模型**，在推理、编码和 Agent 能力上表现出色
- 支持 **201 种语言和方言**

> 💬 开源模型的竞争越来越激烈。Qwen 3.5 的 MoE 架构在效率和能力之间取得了很好的平衡，170 亿激活参数意味着部署成本远低于同级别稠密模型。阿里在 AI 领域的投入正在加速回报。

### Manus 推出消息应用内的个人 Agent

**来源：TLDR AI**

Manus Agents 允许用户直接在 Telegram 等消息应用中使用 Manus 的多步骤任务执行能力。目前仅支持 Telegram，更多平台即将到来。

> 💬 Agent 的真正价值在于无处不在。把 AI Agent 嵌入到人们已有的通讯工具中，比让人们下载新 App 要聪明得多。

---

## 📐 战略与思考

### Stratechery：AI 时代，瘦客户端回来了

**来源：Stratechery（Ben Thompson）**

Ben Thompson 在新文章「Thin Is In」中论述了一个重要观点：

- 从大型机到 PC 再到手机，厚客户端一直占据主导地位
- 但 AI 时代的交互范式正在逆转——聊天界面几乎不需要本地算力，Agent 更是将瘦客户端推向极致
- 当界面是自然语言对话时，多年积累的肌肉记忆变得毫无价值，支撑高价软件的转换成本正在瓦解
- 计算资源不足 + 大模型需要大量内存 → 工作负载将流向大型数据中心

> 💬 这篇文章点出了 AI 时代一个深层结构性变化：当 UI 不再重要时，垂直 SaaS 的壁垒就瓦解了。对创业者和投资人而言，这是一个需要认真思考的信号。

### The Pragmatic Engineer 深度解析：Codex 是怎么造出来的

**来源：The Pragmatic Engineer（Gergely Orosz）**

Gergely 与 OpenAI Codex 团队深入对话，揭示了多个内幕：

- Codex 每周超过 **100 万开发者**使用，1 月以来使用量增长 5 倍
- Codex **90% 以上的代码由自己编写**
- 采用 Rust 编写，走云端异步路线（与 Claude Code 的本地 TypeScript 路线形成对比）
- OpenAI 内部工程实践：分级代码审查、Codex 自测试、通过结对编程帮助新人上手
- 数据基础设施团队用 Codex 在 **2 个月内**构建了原本需要超过 1 年的内部「数据 Agent」

> 💬 「模型自己写自己」不再是科幻。Codex 选择云端异步、Rust 实现的技术路线与 Claude Code 的本地 TypeScript 路线截然不同，但都在各自方向上走得很远。竞争催生了创新。

---

## 🛠️ 开发工具与工程

### Simon Willison 发布 Rodney v0.4.0：浏览器自动化 CLI

**来源：Simon Willison's Weblog**

Simon Willison 的浏览器自动化工具 Rodney 发布 v0.4.0，吸引了大量社区 PR：

- 新增 `rodney assert` 命令，支持 JavaScript 测试
- 新增目录级别的 session 管理（`--local`/`--global`）
- 支持连接已运行的 Chrome 实例、Windows 平台支持
- 可用于构建完整的 Web 应用端到端测试脚本

> 💬 Simon 总能把复杂的工具做得优雅简洁。Rodney 结合他的 Showboat 演示工具，正在构建一套 AI 时代的浏览器自动化基础设施。

### BarraCUDA：开源 CUDA 编译器，目标 AMD GPU

**来源：Hacker News（76 赞）**

BarraCUDA 是一个开源的 CUDA 编译器，目标是让 CUDA 代码直接在 AMD GPU 上运行。

> 💬 NVIDIA 的 CUDA 生态锁定一直是 GPU 市场竞争的最大壁垒。BarraCUDA 如果成熟，将显著降低 AMD GPU 在 AI 训练/推理场景中的迁移成本。值得关注。

### Go 团队推出 go fix：自动现代化 Go 代码

**来源：Hacker News（246 赞）/ Go 官方博客**

Go 官方发布 `go fix` 工具，可自动将旧式 Go 代码迁移到新的惯用写法。

> 💬 Go 团队一直在「让升级变得不痛苦」这件事上下功夫。go fix 延续了这个传统。

---

## 🏗️ 基础设施

### ByteByteGo：Cloudflare 如何将 Serverless 冷启动延迟降低 10 倍

**来源：ByteByteGo Newsletter**

Cloudflare 通过 **worker sharding** 技术将 Workers 平台的冷启动延迟降低了 10 倍：

- 使用一致性哈希环将同一应用的请求路由到同一服务器
- 99.99% 的请求现在直接命中已运行的代码实例
- 冷启动包含 4 个阶段：拉取源码 → 编译 → 执行初始化 → 处理请求

> 💬 Serverless 的「冷启动」一直是开发者的痛点。Cloudflare 的方案本质上是用空间换时间——通过减少代码分散来提高缓存命中率，思路简洁有效。

### Micron 投资 2000 亿美元解决 AI 内存瓶颈

**来源：TLDR AI / WSJ**

美国最大的内存芯片制造商 Micron 正在大规模扩产：

- **500 亿美元**扩建现有园区，新建两座芯片工厂
- **1000 亿美元**纽约制造综合体已动工
- 日本 96 亿美元工厂投资
- 首座新工厂预计 2027 年中开始量产 DRAM

> 💬 AI 的瓶颈正在从算力转向内存。Micron 的天量投资反映了一个事实：未来 AI 模型对 HBM 和 DRAM 的需求将远超当前产能。这也是为什么 Stratechery 文章中提到的「瘦客户端回归」逻辑成立——数据中心将吞噬一切。

---

## 📝 值得一读

### Lenny's Newsletter：如何做出可信赖的 AI 分析

**来源：Lenny's Newsletter**

用户研究专家 Caitlin Sullivan 分享了 4 个让 AI 分析结果可信赖的提示技巧，帮助避免 LLM 最常见的错误——虚构引用、错误结论和盲目自信的输出。

> 💬 AI 输出「看起来都很自信」这个问题被越来越多人意识到了。对于产品经理和研究人员来说，学会如何正确使用 AI 做分析可能比学会 prompt 更重要。

### Gentoo Linux 迁移至 Codeberg

**来源：Hacker News（218 赞）**

Gentoo Linux 正式将代码仓库迁移到 Codeberg，这是又一个主流开源项目从 GitHub 迁出的案例。

> 💬 从 GitHub 到 Codeberg 的迁移趋势值得关注。在 GitHub 日益 AI 化和商业化的背景下，部分开源社区选择了更加独立的平台。
