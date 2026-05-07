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
  虽然 Python 在 AI 算法层生态极佳，但由于 Agent 经常需要频繁调用各种外部工具（Tools），Python 脚本作为工具被反复拉起时，其冷启动速度较慢，极易成为系统的性能瓶颈。
* **为什么推荐 TypeScript + Bun？**
  得益于现代 JS 引擎（如 Bun）底层的 JIT（即时编译）结构，代码会在运行过程中进行预热优化，这意味着你编写的常用工具函数会**“越用越快”**。此外，TS 优秀的类型系统也能为 Agent 的复杂工程提供极佳的代码健壮性。

因此，在接下来的实战环节中，我们将全程采用 **TypeScript** 来编写这个简易 Agent。

---

## 3. 实现架构图


```mermaid
graph TD
    %% 定义节点样式
    classDef brain fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,rx:10,ry:10;
    classDef loop fill:#fff3e0,stroke:#ef6c00,stroke-width:2px;
    classDef io fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px,stroke-dasharray: 5 5;

    %% 外部输入输出
    Start([用户提出任务]) --> History[将任务存入对话历史`TS Memory Array`]
    
    %% 核心 Agent 循环
    subgraph AgentLoop [Agent 核心循环 - 运行在 Bun 环境]
        History --> LLM[🧠 LLM '大脑' `Qwen/Gemini API`调用]
        
        LLM -- 生成 Thought: 思考如何做 --> Decide{是否需要工具?}
        
        %% 分支1：执行工具
        Decide -- 是 (调用函数) --> ExecuteTool[🛠️ 执行工具 Action`调用本地 TS 工具函数`]
        ExecuteTool --> Observe[👀 获取 Observation`工具执行结果`]
        Observe --> UpdateHistory[更新对话历史`Thought + Action + Observation`]
        UpdateHistory --> LLM
        
        %% 分支2：回答用户
        Decide -- 否 (得出结论) --> FinalResponse[✨ 生成最终回答]
    end
    
    %% 外部输出
    FinalResponse --> End([结束任务并输出])

    %% 应用样式
    class LLM brain;
    class AgentLoop,Decide,UpdateHistory loop;
    class Start,End io;