---
title: mRNA-Forge：一个可审计的 mRNA 序列优化 Agent
date: '2026-08-21T10:30:00+08:00'
description: 将蛋白序列优化、确定性质量控制、VaxPress 和 LLM 工具编排组合成一个可复现的 mRNA 设计工作流。
excerpt: mRNA-Forge 把“序列设计 → 质量评估 → 反翻译验证 → 报告交付”封装成可审计 Agent，LLM 负责编排，数值结果来自本地工具。
categories:
  - 项目
tags:
  - mRNA
  - 生物信息
  - AI
  - Agent
  - Python
type: github
---

mRNA-Forge 是一个面向生物序列设计的 Agent demo。它把蛋白序列转成候选 mRNA，再用确定性工具完成质量评估、反翻译验证和报告生成。

**[在 GitHub 查看源码](https://github.com/wujizhesan/mrna-forge)**

## 它解决什么问题

序列优化不能只看一个“模型分数”。一个可交付的工作流至少要回答四个问题：

1. 生成的 mRNA 是否忠实编码原始蛋白；
2. 密码子、GC、GC3、UpA、UpU 和同聚物指标如何；
3. 不同优化策略的差异是什么；
4. 每个结论能否回溯到具体工具和输入。

因此项目采用两层结构：

~~~text
LLM Agent：选择工具、组织步骤、解释结果
              ↓
确定性工具层：优化、评分、反翻译验证、benchmark
              ↓
结构化结果：指标、规则判定、序列和 HTML 报告
~~~

LLM 不直接生成指标，也不负责“编造”实验结论；所有数值来自本地计算。

## 已验证的 demo

项目使用真实荧光素酶蛋白序列作为 demo，长度为 550 aa。

当前 greedy 优化路径可以完成：

- 生成 1650 nt mRNA；
- 反翻译结果与原蛋白一致；
- CAI：1.000；
- GC：65.2%；
- 表达评分：0.602；
- 规则系统给出 **REVIEW**，原因是 GC 高于当前线性 mRNA 阈值。

这里保留 **REVIEW** 很重要：优化不是单指标越高越好，而是在表达、稳定性和结构之间权衡。

项目还保留了 VaxPress 进化优化路径，并通过 naive / greedy / VaxPress 对照实验观察不同策略的取舍。

## Agent 和工具

Agent 可以通过 function-calling 编排：

- **optimize**：生成候选 mRNA；
- **score**：计算 CAI、GC、GC3、UpA、UpU 和规则判定；
- **verify**：反翻译并检查蛋白序列一致性；
- **benchmark**：比较不同编码策略；
- Streamlit UI：展示指标、雷达图、滑窗 GC、报告和解释。

项目同时提供 rule 模式。没有 API Key 时，核心工作流仍然可以离线运行。

## 当前边界

这不是临床或生产级序列设计系统。目前的表达评分是可解释的启发式评分，还没有用实验表达数据校准；circRNA 和 saRNA 也仍处于规则扩展阶段。后续可以接入真实表达数据、UTR/Kozak 设计、RNA 结构约束和更严格的外部 benchmark。

## 快速开始

~~~bash
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt
.venv\Scripts\python agent.py --demo
.venv\Scripts\streamlit run app_streamlit.py
~~~

浏览器打开 http://localhost:8501。

> 公开仓库不包含 API Key。需要 Agent 编排时，请通过环境变量配置 DEEPSEEK_API_KEY。