---
title: "超值出租车呼返策略"
excerpt: "Base_ECR (XGBoost) + Delta_ECR (因果森林) 双模型。RCT 实验 + CATE 个性化弹性系数。"
header:
  image: /images/liw.png
  teaser: /images/liw2.png
---

**技术栈**: 因果森林, XGBoost, RCT, CATE, AUUC, Python

**核心贡献**:
- 拆解为 Base + Delta 双模型，各自用最适合的数据训练
- RCT 随机实验获取多 treatment 弹性数据
- 因果森林输出 CATE 个性化弹性系数
- 单勾/多勾场景分组建模，AUUC 评估补贴排序性
