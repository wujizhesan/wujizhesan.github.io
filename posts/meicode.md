---
title: MeiCode：终端里的多智能体 AI 助手
date: '2026-08-13T17:30:00+08:00'
description: 介绍 MeiCode 的核心能力、技术栈与快速开始方式。
excerpt: MeiCode 是一个运行在终端里的命令行 AI 助手，支持工具调用、Skill 扩展和多智能体团队编排。
categories:
  - 项目
tags:
  - MeiCode
  - TypeScript
  - AI
  - CLI
type: github
---

MeiCode 是一个运行在终端里的命令行 AI 助手。它可以直接操作文件系统、执行命令、搜索代码，通过 Skill 系统扩展能力，并内置多智能体团队编排。

## 核心能力

### Agent 主循环

- 支持模型请求、工具执行、结果回流和流式输出
- 支持中断取消、上下文压缩和会话恢复
- 检测重复工具调用，降低死循环风险

### 多智能体团队

- 主会话可以派生专职成员并行处理任务
- 每个成员拥有独立上下文
- 使用 Git worktree 隔离文件修改
- 通过邮箱和任务协议协作

### Workflow 与 Skill

- 使用声明式 Workflow 拆分复杂任务
- 支持项目级和用户级 Skill
- 提供权限模式、路径围栏、危险命令拦截和快照回滚

## 技术栈

MeiCode 使用 TypeScript、ESM 和 Node.js 构建，终端界面使用 Ink。

## 快速开始

\`\`\`bash
npm install
npm start
\`\`\`

无人值守运行：

\`\`\`bash
npm start -- --run "任务描述"
\`\`\`

运行测试：

\`\`\`bash
npm test
\`\`\`

## 项目地址

[在 GitHub 查看 MeiCode](https://github.com/wujizhesan/meicode)
