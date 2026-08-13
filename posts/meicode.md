---
title: MeiCode：终端里的多智能体 AI 助手
date: '2026-08-13T17:30:00+08:00'
cover: /images/meicode-team.webp
description: 了解 MeiCode 的 Agent 主循环、多智能体团队、Workflow、权限机制与快速开始方式。
excerpt: MeiCode 是一个运行在终端里的多智能体 AI 助手，可操作文件、执行命令、搜索代码，并通过 Skill、Workflow 与团队编排完成复杂任务。
categories:
  - 项目
tags:
  - MeiCode
  - TypeScript
  - AI
  - CLI
type: github
---

MeiCode 是一个运行在终端里的多智能体 AI 助手。它可以直接操作文件系统、执行命令、搜索代码，通过 Skill 与 Workflow 扩展能力，并让多个专职成员并行协作。

> 从一个任务开始，让主 Agent 负责拆解，让专职成员并行执行，再由主会话汇总结果。

**[在 GitHub 查看源码](https://github.com/wujizhesan/meicode)** · **[浏览项目页](/projects/)**

## 项目概览

| 项目 | 说明 |
| --- | --- |
| 语言 | TypeScript / ESM |
| 运行环境 | Node.js |
| 终端界面 | Ink |
| 开源许可 | MIT |
| 协作方式 | 主 Agent、多智能体团队、Workflow |

## 核心能力

### Agent 主循环

- 完整处理模型请求、工具执行、结果回流和流式输出
- 支持中断取消、上下文压缩、溢出落盘和会话恢复
- 检测重复工具调用，降低无效循环风险

### 多智能体团队

- 主会话可以派生专职成员并行处理任务
- 每个成员拥有独立上下文，并可恢复历史记录
- 使用 Git worktree 隔离文件修改
- 通过邮箱、任务和状态协议协作
- 内置 team-lead、research、implement、qa 等角色

### Workflow 与 Skill

- 使用声明式 Workflow 拆分和追踪复杂任务
- 支持项目级和用户级 Skill
- Workflow 阶段支持子 Agent 或团队成员两种执行方式

### 权限与安全

- 提供 default、edits、plan、yolo 四级权限模式
- 使用路径围栏限制成员写入范围
- 拦截危险命令，并提供 Git 快照与回滚安全网

## 工作方式

```text
用户任务
  ↓
主 Agent 分析与拆解
  ↓
专职成员并行执行
  ↓
任务、邮箱与产物汇总
  ↓
主会话验证并交付
```

## 适用场景

- 跨多个文件的代码修改与重构
- 需要研究、实现、测试并行推进的复杂任务
- 可重复执行的工程流程
- 需要权限控制、路径隔离与结果回滚的自动化工作

## 快速开始

```bash
npm install
npm start
```

无人值守运行：

```bash
npm start -- --run "任务描述"
```

运行测试：

```bash
npm test
```

## 配置

复制示例配置到用户目录：

```bash
cp config.example.yaml ~/.mewcode/config.yaml
```

然后填写模型 Provider 与 API Key。项目支持在配置中切换不同模型目录。

## 常用命令

| 命令 | 用途 |
| --- | --- |
| `/mode default\|edits\|plan\|yolo` | 切换权限模式 |
| `/team create\|spawn\|assign\|tasks\|merge` | 管理团队与任务 |
| `/workflow create\|validate\|run` | 创建、校验和运行工作流 |
| `/session`、`/compact`、`/resume` | 管理会话与上下文 |

## 项目地址

项目源码、完整目录结构与最新使用说明均保存在 GitHub：

**[打开 wujizhesan/meicode](https://github.com/wujizhesan/meicode)**
