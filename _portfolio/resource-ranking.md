---
title: "资源位推荐系统"
excerpt: "APP 内多资源位个性化推荐。ESMM 多任务学习 + IPW 纠偏 + K-means 分群建模。"
header:
  image: /images/liw.png
  teaser: /images/liw2.png
---

**技术栈**: ESMM, IPW, K-means, XGBoost, SMD, Python

**核心贡献**:
- 基于 ESMM 的多任务学习（CTR + CVR 联合建模）
- IPW 逆概率加权纠偏 + K-means 用户分群后独立建模
- SMD 评估纠偏效果
- 200+ 维特征体系，线上 AUC 0.78
