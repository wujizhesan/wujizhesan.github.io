---
title: 我的九个逆向工程项目：从拆解原版到可验证重建
date: '2026-08-14T00:40:00+08:00'
cover: /images/1.jpg
description: 汇总初华、睦子、小红、帕瓦、Dr.JC、站斧与战锤、奶绿、挖象、网店魔盒九组逆向工程项目，以及我逐渐形成的管线、验证和交付方法。
excerpt: 这些项目横跨 Chromium、Electron、WPF、WebView2、Vue、React、uni-app 与后端服务。我更关心的不是“拆出来了什么”，而是怎样把观察、提取、重建和验证变成可重复的工程流程。
categories:
  - 逆向工程
tags:
  - 逆向工程
  - 工程复盘
  - 自动化管线
  - 桌面应用
---

过去一段时间，我陆续做了九组逆向工程项目。它们有的是 Chromium 浏览器与扩展，有的是 Electron 桌面端，有的是 WPF 与 WebView2 混合应用，也有 uni-app 小程序和管理后台。

把它们放在一起看，最有价值的并不是项目数量，而是逐渐稳定下来的一套工作方式：先观察真实目标，再建立可重复的提取管线；先跑通独立运行，再谈品牌化与交付；最后用构建、接口、DOM 和像素对比证明结果。

## 项目地图

| 项目 | 对标方向 | 主要技术 | 当前阶段 |
| --- | --- | --- | --- |
| [初华](/posts/chuhua-reverse-engineering/) | 电商数据采集浏览器 | Chromium、Chrome Extension、CDP、Python | 采集链路与本地服务已验证 |
| [睦子](/posts/muzi-reverse-engineering/) | 客服知识库与聚合浏览器 | Vue 3、React、Electron、Flask | 双组件构建通过 |
| [小红](/posts/xiaohong-reverse-engineering/) | 电商运营桌面工具 | WPF、WebView2、Vue、FastAPI | 双桌面壳与主要业务路由完成 |
| [帕瓦](/posts/pawa-reverse-engineering/) | 多平台运营工具箱 | Electron、React、Flask、Extension | Phase 1 完成，进入深度验证 |
| [Dr.JC](/posts/drjc-reverse-engineering/) | 商城小程序迁移 | uni-app、Vue 2、PHP/Java | 品牌与 API 适配完成，待编译部署 |
| [站斧与战锤](/posts/zhanfu-zhanchui-reverse-engineering/) | 多店铺浏览器 | Electron、Vue 3、Node、SQLite | 净室重建与品牌分支并存 |
| [奶绿](/posts/naicha-reverse-engineering/) | AI 营销桌面应用 | Electron、Vue 2、Express、Flask | 218 个组件，管线进入收尾 |
| [挖象](/posts/waxiang-reverse-notes/) | Chromium 桌面应用 | Chromium、二进制分析 | 资产盘点与侦察阶段 |
| [网店魔盒](/posts/wangdianmohe-reverse-notes/) | WPF 电商工具 | .NET 8、WPF、WebView2、ClearScript | 安装包、程序集与界面资源已提取 |

## 我现在采用的四段式流程

### 1. 侦察

先确认目标到底由哪些部分组成。桌面窗口只是表面，背后可能同时存在主进程、渲染进程、浏览器内核、扩展、WebView、后端接口和本地数据库。最早的误判，往往比后面的代码错误更昂贵。

### 2. 提取

根据技术栈选择工具：Electron 先看 ASAR 与 bundle，WPF 关注程序集、BAML、资源和 WebView2，Chromium 项目则要把内核、扩展、配置和更新机制分开。这个阶段的目标不是立即重写，而是建立可追溯的组件地图。

### 3. 重建

能自动化的部分尽量进入管线。典型流程是：bundle 提取、AST 转换、组件生成、路由接入、依赖补齐、构建检查。手工修补只解决少量特例，同类错误出现多次，就应该回到生成器修源头。

### 4. 验证

“能编译”只是第一道门。完整验证通常包括：

```text
构建通过
  → API 端点与数据结构检查
  → 真实浏览器或桌面端运行
  → DOM 结构对比
  → 同视口截图与像素差异
```

## 最重要的几个教训

第一，不要把产品名当成架构。睦子项目直到拆清 QAPI、QChrome 和 QWIN 三个组件后，工作才真正顺畅。

第二，不要信单一指标。构建通过不代表页面正确，接口 200 不代表字段可用，像素差异很小也可能只是截到了相同的空态。

第三，原版、分析目录、当前源码、品牌分支和构建产物必须分开管理。它们看起来相似，承担的职责完全不同。

第四，公开复盘必须有边界。本系列只记录我在授权环境和本地样本上的工程方法，省略账号、密钥、第三方私有数据以及可直接用于绕过访问控制的细节。

这九组项目并非都已完成。它们分别处在侦察、重建、验证和交付的不同阶段。把真实状态写出来，比把每个项目都包装成“完成品”更有价值。
