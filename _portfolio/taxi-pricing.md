---
title: "出租车预估价优化"
excerpt: "MLP 回归模型，长/短单分组建模。多维特征工程 + 锚定快车兜底方案。"
header:
  image: /images/liw.png
  teaser: /images/liw2.png
---

**技术栈**: MLP, PyTorch, 特征工程, 回归分析, Python

**核心贡献**:
- 基于数据驱动的分组建模决策（长/短单分组建模、城市分组）
- 多维特征工程（ODT × POI × 天气 × 计价规则）
- 锚定快车预估价兜底方案（分时×分里程×最优调节系数）
- 极端 Badcase 率大幅下降
