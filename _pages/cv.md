---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

## Education

- **M.S.**, 北京航空航天大学, 仪器科学与光电工程学院, 2020 – 2023
  - 学科评估: **A+**（与清华大学并列全国第一）
  - 研究方向: 计算机视觉、深度学习、模型轻量化
  - 硕士论文: 基于轻量化卷积神经网络的快速目标检测技术研究
- **B.S.**, 吉林大学, 仪器科学与电气工程学院, 2015 – 2019
  - 学科评估: **B+**（2025 软科排名全国第 7）

## Work Experience

**高级算法工程师** — 奇富科技（原 360 数科）, AI技术中心-数据智能部, 2023.11 – 至今

校招身份入职，2 年内两次晋升至高级算法工程师。负责推荐系统、因果推断与 LLM Agent 方向的算法研发。

- **用户运营 Agent 系统**：设计并实现 Text-to-SQL 数据洞察 + 策略生成的 Agent 系统，包含五道 SQL 校验防线（语法/安全/白名单/逻辑/EXPLAIN）、ReAct 防幻觉框架、RAG 历史策略检索、线性规划切分点优化。支持多轮对话、槽位累积、跨 Skill 上下文携带。
- **资源位推荐系统**：基于 ESMM 多任务学习 + IPW 纠偏的全链路推荐。K-means 用户分群后独立建模，SMD 评估纠偏效果。200+ 维特征体系，线上 AUC 0.78。
- **电销坐席智能匹配**：Wide&Deep 模型 + 线性规划求解全局最优分配。拉格朗日松弛实现实时分配框架。

**算法工程师** — 滴滴（DiDi）, 网约车平台公司，MPT, 2023.04 – 2023.07
- 超值出租车呼返策略：Base_ECR (XGBoost) + Delta_ECR (因果森林) 双模型，RCT 实验 + CATE 个性化补贴
- 出租车预估价优化：MLP 回归模型，长/短单分组建模 + 锚定快车兜底方案

## Skills

- **Languages**: Python (primary), SQL, C/C++
- **Deep Learning**: PyTorch, TensorFlow
- **ML/Statistics**: XGBoost, LightGBM, scikit-learn, causal inference
- **Recommendation**: ESMM, Wide&Deep, multi-task learning, recall/ranking
- **LLM Agent**: Langflow, RAG, ReAct, Text-to-SQL, prompt engineering

## Publications

{% if site.publications %}
  {% for post in site.publications reversed %}
    {% include archive-single.html %}
  {% endfor %}
{% endif %}
