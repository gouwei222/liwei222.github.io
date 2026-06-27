---
permalink: /interview-qa/
title: "面试题库"
author_profile: true
toc: true
toc_sticky: true
---

<style>
.qa-item { margin-bottom: 18px; }
.qa-item summary { font-weight: 700; font-size: 15px; cursor: pointer; padding: 8px 0; color: #1a1a2e; }
.qa-item summary:hover { color: #2563eb; }
.qa-item details { background: #fafbfc; border-radius: 8px; padding: 10px 16px; border: 1px solid #e5e7eb; }
.qa-answer { padding: 10px 0 6px 16px; font-size: 13.5px; color: #374151; line-height: 1.8; border-left: 3px solid #e5e7eb; margin-top: 6px; }
.badge { font-size: 11px; padding: 2px 8px; border-radius: 10px; font-weight: 600; }
.badge-easy { background: #dcfce7; color: #15803d; }
.badge-mid { background: #fef9c3; color: #a16207; }
.badge-hard { background: #fee2e2; color: #b91c1c; }
</style>

> 32 道面试题 · 推荐系统 & LLM Agent · 附带参考答案 · 点击题目展开

---

## 一、资源位推荐

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; ESMM 为什么不能直接训一个 CVR 模型？</summary>
<div class="qa-answer">

两个致命问题无法回避：

① **选择偏差（Selection Bias）**：CVR 模型只能用「点击过」的样本训练，因为没点击就没有转化标签。但线上 inference 是在曝光全集上打分。训练空间 ≠ 推理空间，模型在曝光全集上的预测是有偏的。

② **样本稀疏（Data Sparsity）**：曝光量 >> 点击量 >> 转化量。CVR 可用的训练样本极少，极易过拟合。

**ESMM 的解法**：pCTR 和 pCTCVR 都可以在「全量曝光样本」上训练——曝光→是否点击（CTR label），曝光→是否点击且转化（CTCVR label）。然后用 `pCVR = pCTCVR / pCTR` 间接在全量空间上学习 CVR。CVR tower 共享 CTR tower 的 embedding 参数，用 CTR 丰富的梯度信号驱动 CVR 的特征学习。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; IPW 中用了 min(log(1/p), 20) 截断，每一步是为什么？不截断会怎样？</summary>
<div class="qa-answer">

逐层拆解：

• **1/p（逆概率权重）**：p 是某样本被选中的 propensity。如果某个活动因为业务权重低几乎没曝光（p=0.0001），它的权重 = 10000。不处理的话，这一个样本就主导了整个 loss，模型拼命拟合它。

• **log(1/p)**：把权重从 [1, ∞) 压缩到 [0, ∞)。log 的压缩效应——10000 → 9.2，100 → 4.6，大幅降低极端值的影响力。

• **min(..., 20)**：进一步硬截断。即使 log 压缩后，极端样本权重仍有放大效应。截断到 20 后，所有样本权重 ≤ 20。

本质是 **bias-variance tradeoff**：截断会引入一点 bias（极端样本的纠偏不充分），但 variance 大幅下降（训练更稳定）。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 纠偏模型 AUC 只有 0.7，文档说「不是越高越好」——你同意吗？</summary>
<div class="qa-answer">

完全同意。关键要理解这个模型预测的是什么——不是用户的真实偏好，而是业务方的「分配规则」（propensity）。

业务方给活动 A 高权重、活动 B 低权重 → 活动 A 曝光多 → propensity model 学到「活动 A 的 p 更高」。如果 AUC = 0.95，说明模型几乎完美复现了业务规则。但业务规则的产物（有偏的曝光分布）**恰恰是我们要纠偏的对象**。用 AUC 0.95 的模型做 IPW，等于用「完美的偏差描述」去纠正偏差——逻辑上就矛盾了。

正确的评估维度：**SMD 是否下降**（纠偏后 treatment 组和 control 组的特征分布是否更相似），而不是 propensity model 的 AUC。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; K-means 聚 3 类后分别 IPW，为什么不能直接在全体样本上做？聚类解决什么根因？</summary>
<div class="qa-answer">

直接在全体样本上做 IPW 会掩盖「组间异质性」：

假设有两类用户——A 类天然倾向活动 X（转化率 10%），B 类天然倾向活动 Y（转化率 2%）。在全体样本上，活动 X 和 Y 的「平均转化率差」可能被归因为曝光偏差，但实际上大部分是用户构成的差异。

更致命的是，如果 A 类用户恰好占了 80% 的样本，全量 IPW 就会用 A 类的行为模式去「纠偏」B 类——B 类中活动 Y 的低转化被错误地归因于「曝光不足」，从而被赋予不合理的高权重。

**分簇 IPW 的核心思路**：在每个簇内部，用户的行为模式相对同质，此时曝光量的差异更可能反映的是真实的 exposure bias，而不是用户构成的 confounding。每簇内部独立估计 propensity，各簇之间互不污染。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; X→T→Y 因果路径中 confounder 用 IPW 处理，隐含核心假设是什么？如果假设不成立怎么办？</summary>
<div class="qa-answer">

核心假设：**Unconfoundedness（强可忽略性）**——给定观测特征 X，treatment T 与潜在结果 Y 独立。翻译成人话：所有同时影响「业务方给活动多少权重」和「用户转化可能」的变量，都已经在 X 里了。

这个假设几乎一定不完美成立——业务方的「主观偏好」、竞品的「外部冲击」都可能在 X 之外。

应对策略（由易到难）：
1. **敏感性分析**：假设存在一个未观测 confounder U，量化「U 要多强才能推翻结论」。如果阈值很高，结论相对可信。
2. **工具变量 IV**：找只影响 T 但不直接影响 Y 的变量，如活动排期的自然轮换（不是基于用户特征决定的）。
3. **DID + 自然实验**：利用规则变更（如果某天系统改了权重逻辑），比较前后差异。
4. **双稳健估计 DR**：propensity score + outcome regression 组合，只要至少一个模型正确就一致。
5. **在线 A/B 验证**作为最终裁判——离线纠偏只是「猜测」，线上数据是「答案」。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; 离线训练只取每天第一条曝光，线上同一天内多次请求怎么处理？</summary>
<div class="qa-answer">

离线的约束是为了避免「穿越」——如果某用户一天内有 3 次曝光，第 2 次曝光前的特征已被第 1 次曝光后的行为「污染」，模型会学到「看了活动 A 就会点活动 B」这种虚假因果，上线后效果大打折扣。

线上的解决方案是一个组合拳：
1. **实时特征更新（Kafka/Flink）**：每次请求前消费实时行为日志，更新「今日已曝光次数」「今日已点击次数」「距上次曝光时间」等 session 级特征
2. **频控与多样性**：排序层强约束——同一活动不连续曝光、单日曝光上限，避免信息茧房
3. **探索-利用（Thompson Sampling / ε-greedy）**：以一定概率随机出活动，收集 debiased 反馈
4. **概率校准（Platt Scaling / Isotonic Regression）**：训练分布「每天第一条曝光」≠ 线上分布「所有曝光」，对预估 CTR/CVR 做分布校准后才用来排序

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; 冷启动活动零曝光怎么分配资源位？至少两种方案并对比。</summary>
<div class="qa-answer">

**方案一：基于属性的相似度冷启动。** 提取新活动的结构化属性（活动类型、权益类型、目标人群标签），做 embedding → 与历史活动 embedding 做 cosine 相似度 → 取 Top-N 最相似活动的 CTR/CVR 均值作为先验。优点：不需要任何曝光数据即可启动。缺点：假设同类活动表现相似，全新品类会失效。

**方案二：Thompson Sampling。** 新活动给一个宽先验 Beta(1,1)。每次决策时从每个活动的后验分布采样 → 选采样值最高的曝光。新活动因为先验方差大，有一定概率被采样到高值而获得曝光机会；随时间推移，后验收敛到真实值。优点：自动平衡探索与利用，不需要人工调参。缺点：总流量小则收敛慢。

**方案三（补充）：UCB。** `score = CTR_hat + c × sqrt(log(t) / n)`。n 越小（曝光越少），不确定性项越大 → 自动获得更多曝光。理论 regret bound 更好。

实际系统通常是**方案一 + 方案二的混合**——先验来自相似活动，再加探索 bonus。

</div>
</details></div>

---

## 二、电销坐席匹配

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; XGB 建模 user×坐席 为什么不行？文档的根因分析是什么？</summary>
<div class="qa-answer">

三层根因：

1. **XGB 是加性模型。** user 和坐席的效应被近似成「用户贡献 + 坐席贡献」两个独立部分相加。真实的 user×坐席交互效应（如高额度用户×高能力坐席的协同增益）无法被这种加法结构捕捉。

2. **样本稀疏。** 每个 user×seat 组合在训练数据中就几十条样本。树模型要专门为这个组合建一条分裂路径 → 分裂收益不够 → 树根本不会为它分裂。

3. **叶子是区域均值。** 同一 user 喂不同 seat → 因为叶子划分主要靠 user 特征（user 特征重要性远高于 seat），大概率落到同一叶子 → 输出接近 → 打分差异度小（同用户不同坐席打分接近）。

直观理解：XGB 学到的本质是「这个用户有多大概率动支」，**坐席的差异被平均掉了**。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 为什么不直接建动支金额的回归模型，而是建二分类？</summary>
<div class="qa-answer">

金额数据是典型的长尾分布——5% 的用户贡献 80% 的金额。MSE loss 有对称惩罚特性：

预测 100 实际 10000 → error² = 98,010,000，模型被这个样本的梯度「拽着跑」，拼命往高里猜。但如果为了学好这类长尾样本调权重，中间部分的普通用户又变得不准了——两拨人互相干扰。

MSE 惩罚错误但不奖励正确，极大值被严格惩罚后预估值往中间靠拢——这就是回归模型在长尾数据上「预测全在均值附近」的根本原因。

业界标准做法：建几个二分类模型（如「是否动支」「是否大额动支」），在分类模型排的序上看金额的排序性。电销场景样本更少，回归更难做。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; Wide&Deep 为什么在 user×坐席 这个场景比 XGB 效果好？</summary>
<div class="qa-answer">

三个层面的优势：

**① Wide 侧——显式交叉直接喂入。** 「用户可用额度分桶 × 坐席质量」「用户活跃时段 × 坐席拨打时段」「用户年龄 × 坐席话术类型」……这些人工设计的组合特征直接作为 Wide 侧的输入。Wide 是稀疏高维的线性模型，对这些离散交叉天然友好——命中了就激活，不命中的权重为 0，输出离散跳变，差异化天然更大。

**② Deep 侧——隐式交叉自动学习。** user_id embedding + 坐席 embedding + 坐席音色 embedding，MLP 自动学非线性交互。

**③ XGB 的根本局限：对高基数类别交叉特征效果差。** user×seat 的笛卡尔积基数极高（几万 × 几百），树分裂时每个分支只有极少样本 → 增益不够 → 不分裂。

为什么不用 DeepFM？因为 FM 侧的自动特征交叉无法学到精细的业务逻辑（如「年龄大的用户 × 话术标签含操作引导多的坐席」这种组合）。Wide 侧手动设计给了模型一个「正确的起点」。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 线性规划求解匹配时为什么先校准模型分？校准逻辑是什么？</summary>
<div class="qa-answer">

因为模型在加权样本上训练（IPW 加权 + 金额分桶加权），改变了数据分布——模型学到的分布 ≠ 真实数据分布。模型打分与真实动支率存在系统性偏差。

校准流程：
1. 按模型分做十分桶
2. 每个桶内计算「真实 label 均值 / 模型打分均值 = 比例系数」
3. 用这个系数调整模型分，让校准后的分数 ≈ 真实概率

**为什么要校准才能进线性规划？** 线性规划的目标函数是 `max Σ(p_i,j × X_i,j)`，即最大化动支率得分。如果 p_i,j 是系统偏差的（比如整体高估了 20%），那么规划求出来的分配方案在实际中就会偏离预期。校准确保 p 接近真实概率，规划才是以真实期望动支为目标。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; 拉格朗日松弛在实时分配场景解决什么核心难题？价格信号如何自然收敛？</summary>
<div class="qa-answer">

**核心难题：序贯决策 vs 全局约束的耦合。**

原始问题：
```
分配 user_1 给坐席 A 还是 B？
→ 要看 user_2, user_3... user_1000 怎么分
→ 才能保证每个坐席拿到的名单数量均衡
→ 但后面的人还没来，你不知道
```

在线分配时用户一个接一个到达，你在分配第 1 个用户时不知道后面还有多少用户、什么质量、什么可用额度。但约束是全局的。这就是耦合——需要让前面的决策「感知到」后面会发生什么。

**拉格朗日松弛的解法**：把"每个坐席名单量均衡"这条硬约束，从"必须满足"改成"违反就罚款"。

原始目标函数：
```
max  Σ(score_i,j × X_i,j)
s.t. Σ_i X_i,j = 33   (每个坐席拿 33 个)
```

松弛后：
```
max  Σ(score_i,j × X_i,j) - λ_j × (|Σ_i X_i,j - 33|)
```

λ_j 就是坐席 j 的"价格"——拿超了，价格就涨，后面的人自然不选它。

**价格信号的收敛过程：**
```
user_35 到达：
  → A：已分配 35 个 ← 拿超了！λ × (35-33) = 正惩罚
  → B：已分配 32 个 ← 还没超，惩罚 = 0
  → C：已分配 33 个 ← 刚好，惩罚 = 0
  → A 的原始分 0.7 - 惩罚 0.3 = 修正分 0.4
  → B 的原始分 0.7 - 惩罚 0   = 修正分 0.7
  → 选 B ✓
```

类比：高速收费站——不需要一个中央调度器告诉每辆车走哪个口，每个司机看到哪个口排队短就往哪走，整体自然均衡。拉格朗日乘子 λ 就是"排队长度"。

</div>
</details></div>

---

## 三、用户运营 Agent

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 为什么需要「对话路由」这一层？不用会出什么问题？</summary>
<div class="qa-answer">

三个具体失败 case：

**Case 1**：用户输入「帮我写邮件」→ 正则槽位全空 → LLM 硬凑一个不存在的「维度」→ 下游全部跑偏，生成无意义的结果。

**Case 2**：用户查完数据说「加个渠道过滤」→ 系统不认识这是对上一轮的补充 → 当成新请求重新提取全量槽位 → 上一个查询的条件全丢了。

**Case 3**：两个 Skill（数据洞察 vs 策略生成）各做各的意图判断 → 互不知道对方在聊什么 → 无法跨 Skill 复用上下文（用户在数据洞察里选好的人群和时间范围，切到策略生成时要重新说一遍）。

对话路由做三件事：①意图分类——非策略相关的输入在入口就被拦住；②增量补充自动合并——「加个渠道」不会丢失上一轮的条件；③跨槽位携带——通用槽位（人群、时间、过滤条件）自动带过去。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 判断「新话题 vs 补充」为什么用 Python 硬规则而不用 LLM？</summary>
<div class="qa-answer">

三个理由：

1. **规律足够固定。** 「帮我/我想/查一下/找一个」开头 → 新话题；「加个/再加上/换成/不要/只看」开头 → 补充；输入太短且有历史槽位 → 倾向补充。这些规则覆盖了 90%+ 的场景，不需要 NLP 能力。

2. **确定性。** Python 规则的结果是 100% 可预测的。LLM 可能把「加个渠道过滤」误判为新话题，导致上下文丢失——这个错误的代价远大于延迟。

3. **延迟和成本。** 正则匹配是微秒级的，LLM 是秒级的。在对话场景中多一秒延迟用户体验就差很多。

但文档也做了兜底——当正则没命中时（如用户表述复杂或打了错别字），走 LLM 分支。这是「规则覆盖确定性场景，LLM 覆盖边缘场景」的经典模式。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; Text-to-SQL 五道校验防线，只能留三道你留哪三道？</summary>
<div class="qa-answer">

五道防线：语法 / 安全 / 白名单 / 逻辑 / EXPLAIN

**只能留三道：①安全 ②白名单 ③逻辑。**

- **安全**：底线，DROP/INSERT 绝不执行，触发即终止不给修正机会。
- **白名单**：幻觉是 Text-to-SQL 最大问题——语法正确但字段不存在 = 业务错误，必须拦住。
- **逻辑**：GROUP BY 完整性 + 分区过滤——这两类错误最常见且影响最大。

语法错误修起来成本低（一次 self-correction 基本能修好），EXPLAIN 预检可以通过默认缩小日期范围来绕过。安全和白名单是无法妥协的。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 修正节点为什么强调「只修改有问题的部分，不要重写整个 SQL 结构」？</summary>
<div class="qa-answer">

文档给出了实际遇到过的问题：大模型在修正时容易矫枉过正。

具体例子：修正节点被告知「字段名不对」→ 它替换 willingness_score_v3 → willingness_v3_bucket，但在替换时不小心把同一行后面的 GROUP BY 子句也删掉了 → 引入新错误 → 第二次校验失败 → 再次修正 → 又引入新问题 → 死循环。

**「只改错误点」把修正空间从「重写整条 SQL」缩小到「修具体的 bug」，大幅提升稳定性。** 文档后面用 ReAct 的 diff 思路进一步强化了这点——先看清楚自己改了什么，再判断新问题是不是自己引入的。这跟人类 debug 的逻辑是一样的：先 git diff 看改动，再判断是不是自己的改动引入的 bug。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 为什么用槽位重叠度匹配 few-shot 模板，而不是语义检索？</summary>
<div class="qa-answer">

核心理由：数据是结构化的，不需要语义理解。

每条 few-shot 模板前面挂了一组槽位标签：`{analysis_type, required_slots, optional_slots}`。用户输入经过槽位提取后也是一组 key-value 对。用 Python 算两组 key-value 的重叠度——是确定性匹配，毫秒级，零成本。

**语义检索的坑（文档明确指出）：** 两条经验在自然语言描述上可能极其相似但结论完全相反——「老客动支金额按意愿分切效果好」vs「新客动支金额按意愿分切效果差」。Embedding 向量很可能非常近（都关于「动支金额」「意愿分切」），但一个该优先试，一个该跳过。语义检索分不出好坏，结构化匹配一秒分清楚。

还有一个实际考量：语义检索需要维护 embedding + 向量数据库，多了两个可能出错的组件。结构化匹配只需要 Python dict 比对，运维成本为零。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 切分点评分公式：Lift × log(人群) × 覆盖度惩罚——为什么不能只看 Lift？</summary>
<div class="qa-answer">

只看 Lift 会陷入「极高但极窄」的陷阱——某个切分点圈出 50 个人，Lift=10，p<0.001，看起来非常漂亮。但这 50 个人对于运营策略来说毫无意义——发券成本高于收益，而且无法规模化。

三个因子的设计逻辑：
- **Lift**：效果差异（必须 > 1.5 且 p<0.05，否则没资格参与评分）
- **log(高潜人数)**：保证规模。log 避免人数主导得分（50→3.9 vs 5000→8.5，差距是 2.2 倍而不是 100 倍）
- **覆盖度惩罚**：`1 - |(coverage - 0.3)| × 2`，让高潜占比倾向于 30% 左右。太小（5%）扣分，太大（80%）也扣分——超过 50% 等于「几乎没切」，没有圈人价值

运营要的不是「Lift 最高的那几十个人」，而是「Lift 足够好且规模足够支撑一个发券活动的那群人」。30% 左右是常见的最佳运营人群占比。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; RAG「经验只定搜索顺序不替代验证」——架构上如何强制执行？</summary>
<div class="qa-answer">

三层设计强制不绕过：

**① RAG 输出是 priority_dims + skip_dims + examples，不是结论。** 经验以表格形式进入 Prompt，标注「仅供参考，不替代验证」。

**② Python 引擎强制拉数据。** 无论 RAG 说了什么，引擎必须实际调 test_dimension → 拿真实分桶数据 → 跑 find_best_cut_point → 算出实际 Lift 和 p 值。这个步骤是纯代码，不经过 LLM。

**③ ReAct 拦截跨人群经验迁移。** 如果 LLM 引用来自不同人群的经验（如新客上意愿分 Lift=2.8，建议老客也用），ReAct 强制先验证再输出。验证失败则标记「无效」，不输出错误策略。

如果 LLM 绕过 → 老客按意愿分切分无效但策略被发布 → AB 实验浪费时间 → 浪费预算。ReAct 的价值就在于把 LLM 的跨越联想从 bug 变成了可控的 feature。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; Offer 选择用规则引擎、数字用 Python 模板——背后的架构原则是什么？</summary>
<div class="qa-answer">

同一个架构原则：**能做硬约束的地方，绝对不让 LLM 自由发挥。**

**Offer 选择**：LLM 看到「额度使用率高」会联想「提额 offer」→ 但它可能忽略「这个用户还是新客，尚未授信」→ 推荐了不该推荐的 offer。用 Python 硬编码映射表：新客 → 池子里永远没有提额选项，老客 → 根据额度使用率在提额和降息之间选。LLM 只负责把选好的 Offer 包装进文案，不参与决策。

**数字展示**：LLM 在生成文案时不是从数据结构里取值，而是「凭记忆」写 Prompt 里看到的数字——记错了就编一个。Python 用 `str.replace({{ lift }}, 2.75)` 填模板占位符，纯字符串操作。LLM 负责写 qualitative interpretation（「差异显著，建议触达」），但不能包含数字。如果 LLM 文案里出现了数字，Python 后处理校验一致性。

**这条原则的边界在哪？** 当规则的维护成本（每新增一种人群 × Offer 组合就要加代码）超过 LLM 的调用成本 + 幻觉风险成本时，才该放权给 LLM。但在此之前，代码硬约束 > Prompt 软约束。

</div>
</details></div>

---

## 四、超值出租车呼返策略

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 为什么拆 Base_ECR + Delta_ECR 两个模型而不是一个含 treatment 的模型？</summary>
<div class="qa-answer">

最终工程决策公式需要这两个参数独立判断：

- Base_ECR（自然发单率）低但 Delta_ECR（发券增量）高 → 最终发单概率还是低（基数太小）
- Base_ECR 高但 Delta_ECR 低 → 发券没意义（不发券他也会发单，纯浪费预算）

分开建模的工程价值：Base_ECR 不需要 RCT 数据，可以用全量数据（200w+ vs RCT 30w），模型更准确；Delta_ECR 用 RCT 数据对「发券带来的增量」做无偏估计，不被混杂因素干扰。两个模型各自用最适合的数据量和模型结构，互不妥协。

这跟优惠券/营销场景的决策逻辑完全一致：`ROI = uplift × value - cost`。Base 帮你算 value，Delta 帮你算 uplift，分开算才能各自优化。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 因果森林 vs 随机森林的本质区别？「诚实分割」解决什么问题？</summary>
<div class="qa-answer">

本质上问的问题不同：
- **随机森林**：这个 trace 在 treatment=T 下发单概率是多少？（预测 Y）
- **因果森林**：这个 trace 在 treatment=T 下，相对于 control 组，发单概率提升了多少？（预测 τ = Y_T - Y_C）

分裂标准完全不同：
- 随机森林：把 Y 接近的样本分到一起（同叶子内 Y 方差最小化）
- 因果森林：把 treatment effect 接近的样本分到一起（同叶子内 τ 差异小，叶子间 τ 差异大）

**诚实分割（Honest Splitting）**：训练每棵树时，数据先分成两半——一半决定怎么分裂（选特征和切分点），另一半计算叶子里的 treatment effect。目的：防止用同一批数据既决定分裂又估计效应，导致 overfitting bias（叶子的 treatment effect 被系统性地往极端方向偏）。

因果森林的输出是 **CATE（条件平均处理效应）**——不同特征的人得到不同的弹性系数。而只看整体 C 组 vs T 组的差异是 **ATE（平均处理效应）**。

</div>
</details></div>

---

## 五、出租车预估价优化

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 为什么选 MLP 而不是 XGBoost 做回归？</summary>
<div class="qa-answer">

三个关键理由：

**① 预估价是平滑连续函数。** MLP 输出天然平滑；XGBoost 是树模型，输出是阶梯状的（每个叶子节点一个常数值，叶子之间是离散跳变）。

**② 特征交互灵活。** 120+ 特征（ODT × 统计 × POI × 天气），特征间的交互关系复杂。MLP 的隐层自动学非线性交互，不需要人工做特征交叉。XGBoost 只能学到分裂路径上的低阶交互。

**③ ReLU 不怕大误差。** 预估价场景中极端 badcase（预测 50 实际 200）的梯度不会消失——ReLU 正半区梯度恒为 1。如果用 tanh/sigmoid，大误差时梯度趋近 0（饱和区），模型无法从 badcase 中学习。

代价：XGBoost 可解释性更强（可以画出特征重要性、分裂路径），MLP 是黑盒。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-easy">基础</span> &nbsp; 长单和短单为什么要分开建模？MSE loss 下不分开会怎样？</summary>
<div class="qa-answer">

MSE = (预测值 - 真实值)²，对误差绝对值天然敏感。

- 长单（33km+）：实付价 100-300 元，误差 20 元 → MSE = 400
- 短单（3km）：实付价 10-20 元，误差 5 元 → MSE = 25

如果统一建模，一个长单样本的 loss 贡献是短单的 16 倍。模型自然更「关注」长单——梯度更新主要被长单驱动。

后果：短单被牺牲（模型注意力都在长单上，短单被高估）；长单也受影响（短单的低误差信号干扰了长单的学习方向，长单倾向被低估）。

分开建模后，各自价格区间内样本公平竞争，模型不再用长单的标准来衡量短单。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 数据分析阶段如何指导建模方向？这个流程体现了什么方法论？</summary>
<div class="qa-answer">

多维分桶探索，每个洞察直接对应一个建模决策：

- **按里程分桶** → 15km+ 长单和 3km- 短单 badcase 率高且方向相反 → 长/短单分开建模
- **按时间分桶**（10分钟×144桶/天）→ 早晚高峰期 badcase 率显著高 → 新增时间分桶统计特征
- **按里程/时长 diff 分桶** → diff 越大 badcase 率越高 → 新增 diff 相关特征（预估vs实际 MAE）
- **按城市分桶** → 小城市订单稀疏，历史 1 天特征波动大 → 统计窗口拉长到 7 天

方法论：**问题驱动建模**——先搞清楚「什么 case 最差、在哪差、为什么差」，特征工程直接针对 badcase 根因。而不是「把所有能想到的特征全扔进去让模型自己学」。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; 锚定快车预估价兜底——为什么选快车？怎么实现？</summary>
<div class="qa-answer">

**为什么选快车**：快车是最大品类、预估价经过长期优化、准确度和稳定性最强。在极端 badcase 场景（出租车模型彻底失效），快车预估价提供了一个「可靠的锚点」。

**具体实现**：分时（早晚高峰/夜间/平峰）× 分里程（短单/中单/长单）做分桶。在每个桶内部：
1. 用出租车实付价作为 label，快车预估价作为预测值，计算当前桶的 badcase 率
2. 用快车预估价 × 调节系数（0.95, 0.96, ..., 1.04, 1.05），对每个系数重新算 badcase 率
3. 选每个桶 badcase 率最低的系数
4. 线上兜底时：如果出租车模型预估价触发了预警条件，就用「快车预估价 × 该桶的最优系数」替代

核心思路：**不追求完美预估，但保证不出现极端 badcase**。兜底的目标不是降低 badcase 率，是让最差的那部分 case 至少不差到用户投诉的程度。

</div>
</details></div>

---

## 综合交叉

<div class="qa-item"><details>
<summary><span class="badge badge-mid">进阶</span> &nbsp; Agent vs 传统 ML 项目，面试评估侧重点有什么不同？</summary>
<div class="qa-answer">

| 维度 | 传统 ML | Agent |
|------|---------|-------|
| 正确性保障 | AUC / A/B 实验 | 校验防线 × ReAct × 人工兜底率 |
| 失败代价 | 推荐不准→转化低一点 | SQL 写错→策略失误→预算浪费 |
| 可维护性 | 模型定期重训 | few-shot 模板沉淀 + 失败日志驱动迭代 |
| 核心权衡 | bias-variance | 规则覆盖 vs LLM 覆盖 / 延迟 vs 质量 / 灵活性 vs 可控性 |

一个优秀的 Agent 工程师需要知道 **LLM 的边界**——能写规则的地方不调 LLM，能校验的地方不留隐患，能把每次失败转化为系统的成长。

</div>
</details></div>

<div class="qa-item"><details>
<summary><span class="badge badge-hard">高阶</span> &nbsp; 从五个项目中选一个最有区分度的深挖，追问链怎么设计？</summary>
<div class="qa-answer">

选**用户运营 Agent**——它在有限篇幅内展示了最完整的问题解决框架。

**追问链设计：**

**第一层（基础理解）**：「五道防线中，选了错的转化口径（发起 vs 成功）哪道能拦住？」
→ 正则精准匹配的歧义检测（「动支」无其他限定词 → 标记 AMBIGUOUS，反问用户）。考对每道防线能力边界的理解。

**第二层（架构权衡）**：「100 个运营同时用，最大瓶颈在哪？」
→ 不是 LLM 吞吐（可以水平扩展），而是只读从库的并发承载。EXPLAIN 预检阈值收紧，因为多个大查询同时跑会把从库打挂。

**第三层（辩证思维）**：「什么时候该反过来用 LLM 优先？」
→ 规则维护成本 > LLM 调用成本 + 幻觉风险成本时。用户问法无限多变，正则永远追不上——此时 LLM 优先 + 结构校验兜底才是最优解。

**第四层（系统演进）**：「运行一年后最大技术债务？」
→ few-shot 模板库膨胀劣化——很多模板匆匆沉淀、互相重叠甚至矛盾。需要定期「模板清理」：去重、合并相似模板、淘汰长期未命中的、更新 schema 变更后过时的。

</div>
</details></div>
