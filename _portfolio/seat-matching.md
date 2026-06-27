---
title: "电销坐席智能匹配"
excerpt: "Wide&Deep + 线性规划求解全局最优名单分配。拉格朗日松弛实现实时分配。"
header:
  image: /images/liw.png
  teaser: /images/liw2.png
---

**技术栈**: Wide&Deep, XGBoost, 线性规划 (scipy.linprog), 拉格朗日松弛, Python

**核心贡献**:
- Wide&Deep 建模 user×坐席 交互效应，解决 XGB 加性模型组合信号丢失
- Wide 侧手工交叉特征 + Deep 侧 embedding+MLP 隐式交叉
- scipy.linprog 在全局约束下求解最优分配
- 拉格朗日松弛实现实时序贯分配框架
