---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

## Education

- **M.S. Electronic Information Engineering**, Beihang University, 2020 – 2023
  - Thesis: Fast Object Detection Based on Lightweight Convolutional Neural Networks
  - Focus: Computer Vision, Deep Learning, Model Compression
- **B.S.**, Jilin University, 2016 – 2020

## Work Experience

**Algorithm Engineer** — QiFu Technology (360 Shuke), 2023 – Present

- **用户运营 Agent 系统**：设计并实现 Text-to-SQL 数据洞察 + 策略生成的 Agent 系统，包含五道 SQL 校验防线（语法/安全/白名单/逻辑/EXPLAIN）、ReAct 防幻觉框架、RAG 历史策略检索、线性规划切分点优化。支持多轮对话、槽位累积、跨 Skill 上下文携带。
- **资源位推荐系统**：基于 ESMM 多任务学习 + IPW 纠偏的全链路推荐。K-means 用户分群后独立建模，SMD 评估纠偏效果。200+ 维特征体系，线上 AUC 0.78。
- **电销坐席智能匹配**：Wide&Deep 模型 + 线性规划求解全局最优分配。拉格朗日松弛实现实时分配框架。
- **超值出租车呼返策略优化**：Base_ECR (XGBoost) + Delta_ECR (因果森林) 双模型，RCT 实验获取多 treatment 弹性数据，CATE 个性化补贴。
- **出租车预估价优化**：MLP 回归模型，长/短单分组建模。多维特征工程 + 锚定快车兜底方案，极端 Badcase 率大幅下降。

**Algorithm Intern** — Honor (荣耀), 2022

- 手机影像算法：图像去噪、自动调色、AI 摄影模式优化

**Algorithm Intern** — DiDi (滴滴), 2021

- 资源位推荐系统：ESMM 多任务学习 + IPW 纠偏 + K-means 分群

## Skills

- **Languages**: Python (primary), SQL, C/C++
- **Deep Learning**: PyTorch, TensorFlow
- **ML/Statistics**: XGBoost, LightGBM, scikit-learn, causal inference
- **Recommendation**: ESMM, Wide&Deep, multi-task learning, recall/ranking
- **LLM Agent**: LangChain, RAG, ReAct, Text-to-SQL, prompt engineering

## Publications

{% if site.publications %}
  {% for post in site.publications reversed %}
    {% include archive-single.html %}
  {% endfor %}
{% endif %}
