---
title: 站点地图
description: 留沙碎念的内容、项目与公开入口
keywords:
  - 站点地图
  - 留沙碎念
  - MeiCode
siteLinks:
  - url: /
    avatar: /yun.svg
    name: 留沙碎念
    blog: 主站首页
    desc: 文章、项目与个人内容的起点。
    color: '#0078E7'
  - url: /posts/
    avatar: /yun.svg
    name: 博客文章
    blog: 长期内容
    desc: Agent、MeiCode 与工程实践文章。
    color: '#3B82F6'
  - url: /archives/
    avatar: /yun.svg
    name: 文章归档
    blog: 时间线
    desc: 按发布时间浏览全部公开文章。
    color: '#06B6D4'
  - url: /about/
    avatar: https://github.com/wujizhesan.png?size=256
    name: 关于我
    blog: 秋叶白
    desc: 当前方向、项目、目标与公开联系方式。
    color: '#8B5CF6'
codeLinks:
  - url: https://github.com/wujizhesan/meicode
    avatar: https://github.com/wujizhesan.png?size=256
    name: MeiCode
    blog: 多智能体 AI 助手
    desc: 终端里的工具调用、Skill、Workflow 与团队编排实践。
    color: '#0078E7'
  - url: /projects/
    avatar: /yun.svg
    name: 项目橱窗
    blog: 作品总览
    desc: 已公开项目、实验仓库与待整理内容。
    color: '#10B981'
  - url: https://github.com/wujizhesan
    avatar: https://github.com/wujizhesan.png?size=256
    name: GitHub
    blog: 代码与协作
    desc: 公开仓库和项目更新入口。
    color: '#6e5494'
communityLinks:
  - url: /sponsors/
    avatar: /yun.svg
    name: 赞助与支持
    blog: 支持本站
    desc: 支持方式、公开名单与后续说明。
    color: '#F87171'
random: false
---

散落在不同页面里的文章、项目与公开入口，都从这里出发。

## 博客

<YunLinks :links="frontmatter.siteLinks" :random="frontmatter.random" errorImg="/yun.png" />

## 项目与代码

<YunLinks :links="frontmatter.codeLinks" :random="frontmatter.random" errorImg="/yun.png" />

## 交流与支持

<YunLinks :links="frontmatter.communityLinks" :random="frontmatter.random" errorImg="/yun.png" />

## 以后会加入

- 学习笔记与短记录入口
- 更多项目演示和项目文档
- 经授权公开的社交平台与联系方式
- 更完整的站点历史
