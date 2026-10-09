---
title: Bio Research Agent Platform：把异构科研工具装进同一套插件契约
date: '2026-10-04T20:00:00+08:00'
description: 一个可插拔的生物科研 Agent 平台：用统一插件契约、领域注册表和工作流运行器，把 CADD、组学、mRNA 序列设计与文献检索接进同一套 Agent 工具协议，并补上任务调度、插件沙箱与可观测性。
excerpt: 把分子对接、组学分析、序列设计和文献检索塞进一个平台，难点从来不是"接得多"，而是接得一致、跑得可追踪、结果可审计。这篇记录平台的分层方式，以及为"能上生产"补的那些工程件。
categories:
  - 科研
tags:
  - 科研平台
  - Agent
  - FastAPI
  - 生物信息
  - 平台工程
type: github
---

做科研工具集成，最容易走的弯路是"一个工具写一套接口"。分子对接要用 AutoDock Vina 和 RDKit，组学要做差异表达和通路富集，序列设计要算密码子和折叠结构，文献检索要连 PubMed、UniProt、KEGG——每个工具都有自己的输入格式、自己的运行时长、自己的失败方式。当它们散落在不同的脚本里，你得到的是**一堆能跑一次的代码**，而不是一个能反复交付结果的平台。

所以我做了 **Bio Research Agent Platform**：一个可插拔的生物科研 Agent 平台。它把"接什么工具"和"怎么跑、怎么追踪、怎么审计"拆开——工具通过统一插件契约接入，平台负责调度、隔离与可观测性。

**[在 GitHub 查看源码](https://github.com/wujizhesan/bio-research-agent-platform)**

## 平台的分层方式

```text
React 研究工作台
      │ HTTP / SSE
      ▼
FastAPI 服务层 ───── PostgreSQL（任务读模型 / 最终状态）
      │                    ▲
      ├──────── Redis ─────┘（队列 + 执行缓存）
      │
      └──── 插件契约 / 领域注册表 / 工作流运行器
                    ├── CADD        RDKit · AutoDock Vina · 虚拟筛选 · 活性预测
                    ├── Omics       RNA-seq 差异表达 · 通路富集 · 报告生成
                    ├── Sequence    mRNA-Forge 优化 · 评分 · 翻译验证
                    ├── Literature  本地证据 · UniProt · PubMed · NCBI Gene · KEGG
                    ├── Knowledge   本地 Markdown / 文本 / HTML 科研资料索引检索
                    └── Research    领域选择 · 证据源推断 · 可追踪的跨领域工作流
```

关键设计只有三条：

1. **统一插件契约**：领域只负责"这个工具吃什么、吐什么"，不关心排队、隔离、重试和日志。
2. **领域注册表 + 工作流运行器**：`Research` 域根据任务推断需要哪些领域和证据源，再把跨领域步骤串成一条可追踪的工作流，而不是让调用方自己拼命令。
3. **服务层只做编排**：提交任务、订阅状态、查询结果，科学计算全部下沉到插件执行层。

## 任务这一层，才是真正的工程量

科研任务的特点是**长、重、会失败**。一个对接任务可能跑十几分钟，一次组学分析要吃几十 GB 内存。这类负载塞进 HTTP 请求里就是灾难，所以平台把它做成了后台任务 + 事件流：

- **双执行后端**：本地调度器（开发）与 Redis Worker（生产），通过 `JOB_BACKEND` 切换；Redis Worker 使用高/普通/低三级队列，并跳过能力不匹配的任务。
- **进程隔离**：生产任务默认在独立子进程中执行，故障、超时和取消不会拖垮 API 或 Worker；`JOB_TIMEOUT_SECONDS`、`JOB_MEMORY_LIMIT_MB`、`JOB_CPU_TIME_SECONDS`、`JOB_RESULT_MAX_BYTES` 控制资源上限。
- **容量与优先级**：任务提交时可声明 CPU、内存、GPU、GPU 显存和 Worker 标签需求；Worker 容量由 `JOB_TOTAL_*` 声明，并发任务原子预留资源，调度器做容量预留和优先级回填。`GET /api/v1/scheduler/resources` 能看到剩余额度。
- **至少一次投递**：Redis 侧用 processing 列表 + 租约自动续租，进程崩溃后回收过期任务；达到最大尝试次数或执行失败的任务进 dead-letter 队列，不会静默消失。
- **结果缓存与幂等**：每个任务缓存成功结果，恢复同一任务优先复用；提交支持 `Idempotency-Key`，同工具同参数重复提交复用原任务，参数不一致直接 `400`。
- **进度可订阅**：`GET /api/v1/jobs/{job_id}/events` 走 SSE，推送 `queued`、`running` 和终态事件。

## 一个我比较满意的交互：两阶段研究模式

科研里最怕的是"按钮一点，跑了一小时，结果不是你要的"。所以研究工作台把研究任务拆成两阶段：

1. 先提交 `research_plan`，平台展示**这次会用哪些领域、哪些证据源、输入门槛是什么、实际工具链长什么样**；
2. 用户确认后，再提交 `research_execute` 真正执行。

两阶段任务复用同一套 Job/SSE 生命周期。这个设计把"不可见的自动化"变成"可审阅的计划"，我认为是科研场景里比单次问答更重要的东西。

## 可观测性与安全：不是加分项，是及格线

**可观测性**

- `/metrics` 统一暴露 Prometheus 指标：HTTP 请求量/延迟/并发、任务排队与执行耗时、状态迁移、插件工具调用、工作流与步骤执行、插件健康状态。指标标签**只使用后端、路由模板、已注册工具和有限状态**，不用 job ID 或用户输入，避免高基数把监控打爆。
- Worker 独立在 `:9000/metrics` 暴露队列深度、processing 深度、重试、缓存命中和当前执行数。
- API 接受 `X-Request-ID`、`X-Trace-ID` 和 W3C `traceparent`，同一 `trace_id` 贯穿 API、任务记录、调度器/Worker、隔离子进程、插件工具与工作流清单。
- 日志默认 JSON 结构化，令牌、密码、Cookie、API Key 等敏感字段自动脱敏。

**高安全档**

科研数据和计算环境往往不能随便放。高安全档把所有科研工具执行转发到独立的插件沙箱容器：

- 沙箱**不接收**数据库、Redis、JWT 或对象存储凭据，只通过带 32 字符以上随机令牌的内部网络接受请求，令牌由 Compose Secret 以只读文件分别授予 Worker 和沙箱，不进入环境变量；
- 非 root 用户、只读根文件系统、`no-new-privileges`、移除全部 Linux capabilities、限制 PID/CPU/内存/临时目录；不挂载 Docker Socket，也访问不到数据库网络；
- 沙箱**默认无外网**，需要联网的插件走单独的白名单出口代理。

上传链路同样按"关闭式拒绝"设计：文件在写入正式元数据或上传 S3 前，依次经过 ClamAV `INSTREAM` 扫描、CDR 内容重构、重构后复检。CDR 会规范化科研文本和 VCF gzip、重新序列化 JSON/YAML、把 HTML 转成无活动内容的纯文本；ClamAV 不可用、发现恶意内容或 CDR 失败时，高安全档直接拒绝上传并清理隔离目录。

## 迁移这件事也认真做了

老版本有一个 `python -m src.api_server` 的 HTTP 服务。它现在是兼容期状态：**2026-12-31 00:00 UTC 起拒绝启动**，随后从发行包移除。兼容期内它只允许开发环境使用，启动时打废弃警告，所有 HTTP 响应带 `Deprecation`、`Sunset`、`Link` 和 `X-API-Successor` 四个头，明确告诉调用方新接口在哪。新部署一律走 `bio-agent-api` 和 `/api/v1/*`。

我自己的看法是：**能删掉的接口，比"永远兼容"更有诚意。**

## 快速开始

```bash
uv sync --locked --no-dev
bio-agent-api --port 8000
# 打开 http://127.0.0.1:8000/docs
```

Docker Compose 会同时起 FastAPI、PostgreSQL 与 React 研究工作台（`:5173`）；高安全档另外叠一层：

```bash
export PLUGIN_SANDBOX_TOKEN="$(python -c 'import secrets; print(secrets.token_urlsafe(48))')"
docker compose -f docker-compose.yml -f docker-compose.secure.yml up --build
```

## 诚实说明

- **CADD 是目前最完整的科学计算领域实现**，其他领域通过同一套插件和工作流接口扩展，并非同等成熟。
- 平台解决的是"工具怎么被可靠地跑起来、追踪到、审计到"，**不替代具体科研工具的科学验证**。领域接入后仍需要针对该领域补输入校验、结果模型、资源调度和科学质量评估。
- 科研计算依赖的大型受体、分子库、Vina 可执行文件需要按项目说明另行挂载，Compose 不会替你准备好。
- 高安全档的 ClamAV 首次启动要下载特征库，建议预留至少 4 GB 内存。

## 项目地址

**[打开 wujizhesan/bio-research-agent-platform](https://github.com/wujizhesan/bio-research-agent-platform)** · MIT 许可

CI 会跑 Python 3.11/3.12 测试、Alembic 迁移、领域/工作流冒烟和 Docker 镜像构建。相关文章：[CADD 虚拟筛选流水线](/posts/cadd-virtual-screening/) · [发酵软测量与数据驱动选株](/posts/fermentation-soft-sensor/)
