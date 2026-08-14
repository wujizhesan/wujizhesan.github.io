---
title: CADD 虚拟筛选流水线：从真实 EGFR 抑制剂到可复现筛选
date: '2026-08-14T17:30:00+08:00'
description: 用 AutoDock Vina、RDKit 与 scikit-learn 构建 EGFR 虚拟筛选流水线，并结合 ML 活性预测，完成一次"真实数据 + 对抗性审查"驱动的生物计算实践。
excerpt: 一个以 6 个已上市 EGFR 抑制剂为活性对照、AutoDock Vina 做分子对接、随机森林做活性预测的虚拟筛选流水线，附对抗性审查后的数据修复记录。
categories:
  - 项目
tags:
  - CADD
  - 生物信息
  - Python
  - AutoDock Vina
  - RDKit
  - AI
type: github
---

这是一条建立在 **真实药物数据** 基础上的 EGFR 靶点虚拟筛选流水线。它用 AutoDock Vina 做分子对接、RDKit 处理分子、scikit-learn 做活性预测，并把流程通过一条命令串起来，支持对已知活性/非活性分子做可复现的评估。

> 虚拟筛选的价值：不用做湿实验，用计算从大量分子里挑出最可能结合靶点的候选，减少实验成本。但要可靠，方法得先能区分"已知活性"与"已知非活性"。

**[在 GitHub 查看源码](https://github.com/wujizhesan/cadd-virtual-screening)**

## 项目概览

| 项目 | 说明 |
| --- | --- |
| 语言 | Python |
| 对接引擎 | AutoDock Vina v1.2.7（官方 Windows exe） |
| 化学库 | RDKit |
| 配体准备 | Meeko |
| 活性预测 | RDKit 描述符 + 随机森林 |
| 开源许可 | MIT |

## 数据：这次不用编的分子

最早的版本踩过一个坑：从 ChEMBL 拉回来的"EGFR 活性分子"实际上是一批**没有芳环的糖/多元醇**，根本不是激酶抑制剂。经对抗性审查后，我改用 **PubChem 官方 API** 拉取 6 个**已上市** EGFR 抑制剂的正准 SMILES 重建分子库：

- erlotinib / gefitinib / osimertinib / afatinib / lapatinib / icotinib
- 另有 AQ4（PDB 4HJO 的真实共晶配体）

非活性负对照既有常见小分子，也补充了有芳环的分子（苯胺、苯甲酰胺、异喹啉等），避免"分离太好是分子差异太大"的假象。

## 技术流水线

```text
PDB 受体准备（去配体/水）
  ↓
分子库（SMILES → 3D 构象 → PDBQT）
  ↓
AutoDock Vina 批量对接（取多构象最优打分）
  ↓
结合打分排序 + ML 活性预测交叉验证
  ↓
报告与可视化（report.md / 打分图 / Top hit 结构图）
```

### 关键设计

- **盒子中心真计算**：从原始 PDB 的 AQ4 共晶配体质心算结合口袋，而非硬编码魔数
- **对接打分稳定**：取 Vina 多个构象里的最优结合得分，而非固定第一个 mode
- **双引擎交叉**：对接（几何结合）+ ML（性质建模）互相对照

## 结果

对 17 个分子（7 活性 + 10 非活性）跑完整流水线：

```
rank  molecule      tag       score(kcal/mol)
 1    lapatinib     active    -9.67
 2    gefitinib     active    -8.68
 3    afatinib      active    -8.49
 4    osimertinib   active    -8.03
 5    AQ4           active    -7.77
 6    erlotinib     active    -7.64
 7    (非活性)        ...
```

- 6 个已上市 EGFR 药物全部排进前 6，最强 hit 是 **lapatinib（-9.67）**
- 富集倍数约 x2.4，QA 自动断言 5/5 通过
- 对接打分可稳定复现

> 诚实说明：这是**小样本验证演示**，ML accuracy=1.00 来自性质差异明显的负对照，不代表真实世界泛化。真实虚拟筛选需用 DUD-E 这类与活性性质匹配的 decoy 做更严格评估。

## 对抗性审查的教训

这次最值得写下来的是：**数据污染比算法 bug 更难发现、更致命**。一开始"15 个 ChEMBL 真实 EGFR 抑制剂"其实是糖分子，靠着"糖 vs 小疏水分子"的差异轻易分出 1.00——如果直接拿去做面试或产出，一句"这是糖分子怎么叫 EGFR 抑制剂？"就打穿。修复的关键是：

1. 用**官方、确定性**的来源（PubChem REST）而非"看起来是"的来源
2. 用 RDKit 验证每个分子的形态（芳环数、LogP、氢键数目）
3. 请**对抗性视角**独立审查，假设"我没做对"

## 快速开始

```bash
python -m venv .venv
pip install -r requirements.txt
python src/pipeline.py --exhaustiveness 8
```

查看离线报告与 QA：

```bash
python src/agent.py --report
python src/qa_verify.py
```

## 项目地址

项目源码、完整文档与讲解稿均保存在 GitHub：

**[打开 wujizhesan/cadd-virtual-screening](https://github.com/wujizhesan/cadd-virtual-screening)**
