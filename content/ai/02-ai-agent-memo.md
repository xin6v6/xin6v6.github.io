---
title: "AI时代第二章：制作Agent的前置准备"
date: 2026-05-07
tags: ["AI", "Agent"]
summary: "手把手带你完成模型选型、技术栈确认与架构设计等核心前置准备。"
---

在上一章节中，我们已经初步了解了 Agent（智能体）的工作原理。现在，是时候将理论转化为实践，开始设计属于你自己的第一个简易 Agent 了！

## 1. 寻找一个合适的 LLM API

大语言模型（LLM）目前分为开源与闭源两大阵营。在推理能力、专精技术、使用体验与调用价格上，它们各有千秋。

为了帮你避坑，这里基于笔者真实的业务使用体验，从四个维度对主流模型做了一个简单的打分与排名（仅代表个人真实体验）：

| 评估维度 | 综合排名 (从高到低) |
| :--- | :--- |
| 🧠 **推理能力** | Gemini > Claude > OpenAI = Qwen > GLM > MiniMax |
| ⚡ **响应速度** | OpenAI > Claude > MiniMax > Gemini > Qwen > GLM |
| 💻 **代码能力** | Claude = Gemini > 其他 |
| 🌟 **综合体验** | Gemini > Qwen > Claude > OpenAI > MiniMax > GLM |

**💡 核心推荐方案：**
* **海外/出海开发者首选**：`Gemini` (综合体验最佳，推理能力顶尖)
* **国内开发者首选**：`Qwen` (通义千问，国内体验与性价比的极佳平衡)

---

## 2. 选择 Agent 架构语言

明确了模型后，我们需要选择开发 Agent 的底层语言。综合笔者的实际工程体验，**TypeScript (TS) 搭配 Bun 引擎** 是目前在性能和综合能力上的最优解。

* **为什么不首选 Python？**
  虽然 Python 在 AI 算法层生态极佳，但在我们的架构设计中，每一个工具（Tool）都是一个**独立的脚本文件**，Agent 通过命令行调用它——这样做的好处是新增工具无需重启服务，直接添加新脚本即可热插拔。然而代价是：每次调用都要拉起一个新的 Python 解释器，冷启动开销在单次调用中看似微小，但这是一个工具数量和任务复杂度会持续增长的长期项目，随着单次任务调用的工具链变长，这个开销会线性累积，最终成为不可忽视的性能瓶颈。而同样是独立脚本的设计，TypeScript 搭配 Bun 引擎的启动速度远快于 Python，提前做出这个选择，是为规模化做准备。
* **为什么推荐 TypeScript + Bun？**
  Bun 是专为性能设计的现代 JS 运行时，启动速度远快于 Python 解释器，在独立脚本的调用模式下优势尤为明显。同时，这也是一个长期持续演进的项目，TS 的静态类型系统在工具数量和复杂度不断增长时，能有效降低维护成本、减少运行时错误。

因此，在接下来的实战环节中，我们将全程采用 **TypeScript** 来编写这个简易 Agent。

---

## 3. 实现架构图


{{< mermaid >}}
graph TD
    %% 外部输入输出
    Start([用户提出任务]) --> History[将任务存入对话历史 TS Memory Array]
    
    %% 核心 Agent 循环
    subgraph AgentLoop [Agent 核心循环 - 运行在 Bun 环境]
        History --> LLM[🧠 LLM 大脑 Qwen 国内 / Gemini 海外 API 调用]
        
        LLM -- 生成 Thought: 思考如何做 --> Decide{是否需要工具?}
        
        %% 分支1：执行工具
        Decide -- 是 调用脚本 --> ExecuteTool[🛠️ 执行工具 Action 调用本地 TS 工具脚本]
        ExecuteTool --> Observe[👀 获取 Observation 工具执行结果]
        Observe --> UpdateHistory[更新对话历史 Thought + Action + Observation]
        UpdateHistory --> LLM
        
        %% 分支2：回答用户
        Decide -- 否 得出结论 --> FinalResponse[✨ 生成最终回答]
    end
    
    %% 外部输出
    FinalResponse --> End([结束任务并输出])
{{< /mermaid >}}