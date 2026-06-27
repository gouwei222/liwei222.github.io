---
title: "用户运营 Agent"
excerpt: "面向运营人员的智能 Agent 系统——对话路由、Text-to-SQL 数据洞察、策略生成，五道 SQL 校验防线 + ReAct 防幻觉。"
header:
  image: /images/liw.png
  teaser: /images/liw2.png
---

**技术栈**: LangChain, RAG, ReAct, Text-to-SQL, Python

**核心贡献**:
- 设计对话路由层：意图分类 + 多轮槽位累积 + 跨 Skill 上下文携带
- 五道 SQL 校验防线（语法/安全/白名单/逻辑/EXPLAIN），确保生成稳定性
- ReAct 框架防幻觉：跨人群经验验证、复杂修复自检
- RAG 历史策略检索（槽位重叠度匹配，非语义检索）
- 维度穷举搜索 + 加权评分 + 切分点优化算法
