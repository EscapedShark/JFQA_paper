# JFQA / Management Science 选题建议

> 版本：2026-09-25
> 前提：有 WRDS、主流数据与大模型 API、约 1TB 本地存储
> 说明：新颖性判断基于 2026 年 9 月的网络检索（SSRN、NBER、期刊、arXiv 的搜索结果），不能保证穷尽。动手前请按 §3.3 末尾的关键词再检索一遍。

---

## 0. 结论速览

**主推方向 A：并购私下出售流程中的信息扩散与泄露交易。**
建议把它做成一个“合并委托书（merger proxy）数据平台”：同一套大模型抽取管线可以支撑 2–3 篇论文，方向 A2 就是第二篇。

**时间窗口型副线是方向 B：2026 年 11 月生效的半分钱最小报价单位。** 只建议有微观结构背景、并且能马上开工的人做。

**方向 C、D 作为储备。**

| 优先级 | 方向 | 一句话 | 新颖性 | 可行性 | 被抢先风险 | 首选期刊 |
|---|---|---|---|---|---|---|
| 1 | **A. 私下出售流程中的信息扩散与泄露交易** | 用大模型从约 5,000 份合并委托书的 “Background of the Merger” 章节抽取按日期排列的流程事件；检验每多一批签保密协议（NDA）的外部方，目标公司期权和股票是否出现知情交易 | 高 | 高 | 中 | JFQA（MS 次选） |
| 2 | **A2. 投行估值中的折现率：利率传导与黏性** | 同一批文件的 “Opinion of Financial Advisor” 章节；用 Kroll 离散调整推荐 ERP 和正常化无风险利率作为准自然实验 | 中 | 高 | 中 | JFQA / MS |
| 3 | **B. 半分钱最小报价单位（Rule 612）** | 以 1.5 美分价差为门槛的断点回归；有时间窗口，必须现在准备 | 高（仅窗口期内） | 中 | 高 | JFQA / MS |
| 4 | **C. 全机构投票数据（新版 N-PX）与代理顾问行业变局** | 2024 年起所有 13F 机构须披露 say-on-pay 投票；JPMorgan 改用内部 AI 投票等冲击 | 中 | 高 | 中 | MS |
| 5 | **D. Form 144 电子化：内部人卖出的“事前披露”自然实验** | 2023-04 起 Form 144 实时公开；检验门槛附近的聚集（bunching）和信息租金 | 中 | 高 | 中 | JFQA（题目偏窄） |

第 8 节列出了我检索后认为已经拥挤、不建议现在进入的方向，以及对应的竞争论文。

---

## 1. 我对你资源的理解与假设

- **WRDS**：假设至少有 CRSP、Compustat、IBES、OptionMetrics、TAQ（含 WRDS Intraday Indicators）、LSEG/Thomson 13F 与内部人交易数据、ISS Voting Analytics、SDC 或 Capital IQ 并购数据、RavenPack。各学校订阅差别很大，第一周先核对（§3.9）。
- **“订阅 API”**：我理解为主流大模型 API（用于大规模文本抽取）加上常见金融数据 API。方向 A、A2、C 的核心依赖大模型 API。如果你指的只是数据 API，方向 A 就只能靠人工或 RA 标注，周期会明显拉长。
- **1TB 存储**：足够容纳全量 EDGAR 相关文本，以及目标公司子样本的期权和高频数据。不要把全市场 TAQ 逐笔数据下载到本地；在 WRDS Cloud 上算好日度或分钟级指标，只下载结果（§9）。
- **不知道的信息**：你的领域偏好（公司金融、资产定价或微观结构）、编程熟练度、是否有合作者。下文排序按“一个人加大模型工具、12 个月内出工作论文”的情形给出。如果你是微观结构方向，方向 B 可以升为主线。

---

## 2. 选题逻辑：比较优势在哪里

1. **WRDS 本身不是优势，所有竞争者都有。** 真正的优势是：用大模型把过去只能手工收集几百个样本的监管文本变量，做成全样本、事件级的数据，再和 WRDS 的市场数据对接。
2. **优先用公开文本（EDGAR）。** 把 WRDS 授权数据（例如电话会议文本）发给外部大模型 API，可能违反数据许可。EDGAR 文件没有这个问题，也便于满足期刊的复现要求。
3. **回答一阶的经济问题，而不是“用大模型预测收益”。** 后者已有大量工作，审稿人普遍担心前视偏差（Glasserman & Lin 2023；Sarkar & Vafa 2024）。
4. **避开大牛正在抢的热点事件。** 私募信贷基金挤兑、官方统计可信度、预测市场等 2025–26 年的热点，已经有 Goldstein、Bloom 等人的 NBER 工作论文（见 §8）。
5. **从第一天就按可复现标准搭管线。** 据 JFQA 代码共享政策页面，作者需要把代码以及原始数据（受许可限制时用可运行的伪数据）存入 JFQA Dataverse；使用 AI 工具的论文需用开源模型或带时间戳的闭源模型版本，并记录提示词（投稿前以官网原文为准）。Management Science 自 2019 年起执行数据与代码披露政策，由数据编辑团队审查复现包。

---

## 3. 方向 A（主推）：并购私下出售流程中的信息扩散与泄露交易

**工作标题**：*Who Leaks? Information Diffusion in Private Takeover Negotiations*

### 3.1 研究问题

上市公司被收购前，通常要经历数月的私下出售流程：接触潜在买方，签署保密协议，开放数据室，进行多轮报价，给予排他期，最后签约并公告。每多一个签了 NDA 的外部方（战略买家、私募股权基金及其融资银行和共同投资人、各方顾问），知道这笔交易的人就更多。

- **RQ1（信息扩散与知情交易）**：新一批外部方签署 NDA 之后，目标公司的异常期权成交（尤其是短期、价外看涨期权）、股票订单不平衡和价格漂移，是否比签署之前显著上升？
- **RQ2（谁在泄露）**：效应是否随对手方类型（战略买方还是财务买方）、对手方数量、是否引入融资银行、顾问身份而变化？
- **RQ3（后果）**：泄露带来的股价上涨会不会反过来抬高最终收购价（重新检验 costly feedback）？会不会降低交易完成率？卖方如何在“扩大竞价范围带来的竞争收益”和“泄露成本”之间权衡？
- **RQ4（扩展：影子交易）**：与战略买方签 NDA 之后，目标公司的同业公司（例如 Hoberg–Phillips TNIC 同行）是否也出现异常期权交易？这一问题与 SEC v. Panuwat 案以及 Mehta, Reeb & Zhao (2021) 相呼应。

**示意摘要（结构示意，结果待验证）**：
> Using an LLM-constructed, event-level record of ~5,000 private sale processes of U.S. public targets (2001–2025), we show that informed trading in target options and stocks rises sharply after each new batch of outside parties is brought under confidentiality agreements, holding the deal fixed. Leakage is concentrated in processes involving financial sponsors and their financing banks. Bidders revise offers in response to fundamental news but not to leakage-driven run-ups, and targets with high leakage exposure run narrower auctions.

### 3.2 为什么是现在、为什么是你

- 合并委托书的 “Background of the Merger” 章节会按日期逐条披露整个出售流程，但过去只能手工收集，样本都不大：Boone & Mulherin (2007) 约 400 笔（1989–1999）；Schubert (2020) 780 笔；Eckbo, Norli & Thorburn (2026) 手工编码了 4,636 笔要约，公开摘要显示其重点是交易由谁发起。
- 大模型把全样本（2001–2025 年，约 5,000 笔）事件级抽取的费用降到了几千美元以内。数据全部来自 EDGAR，公开、可复现，没有许可风险。
- 市场端数据（OptionMetrics、TAQ、CRSP、RavenPack）都在 WRDS 上，正是你已有的资源。

### 3.3 最接近的文献与差异

| 文献 | 做了什么 | 本项目的差异 |
|---|---|---|
| Augustin, Brenner & Subrahmanyam (2019, *MS*) | 1996–2012 年 1,859 笔收购公告前的期权交易：约 25% 有正的异常成交，其中过半无法用投机、新闻传闻、内部人交易或股市泄露解释 | 把“谁、在什么时候知道”放进模型，解释这部分说不清的知情交易 |
| Betton, Eckbo, Thompson & Thorburn (2014, *JF*) | 公告前的上涨主要反映理性预期，拒绝 costly feedback | 用 NDA 时点区分泄露驱动和基本面驱动的上涨，再检验出价是否跟随 |
| Liu (AEA 2020)；Liu & Officer (SSRN 3383209)；Liu, Officer & Tu (SSRN 4422506) | 手工数据：私下谈判中的出价修正与股价变化相关；谈判与拍卖方式 | 关注信息扩散与交易；全样本、事件级 |
| Schubert (2020, SSRN 3640480) | 780 笔：私下竞争强度（报价数 / NDA 数）与溢价 | 同上，并加入泄露成本这一权衡 |
| Eckbo, Norli & Thorburn (2026, SSRN 7194259) | 4,636 笔要约的发起方 | 他们研究谁发起交易；本项目研究流程中的信息扩散 |
| Boone & Mulherin (2007, *JF*)；Masulis & Simsir (2018, *JFQA*)；Liu & Mulherin (2018, *JCF*) | 出售方式、交易发起、竞争的经典手工数据 | 事件级时间线加上高频市场数据 |
| Dai, Massoud, Nandy & Saunders (2017, *JCF*)；Bargeron, Clifford & Qiu (2025, *JFQA*)；Hasan 等 (2025, *JFR*) | 对冲基金、申报代理机构、社会关系等泄露渠道 | 用流程内部的时点识别（within-deal），不依赖横截面相关 |
| Mehta, Reeb & Zhao (2021, *TAR*)；Ahern (2017, *JFE*)；Kacperczyk & Pagnotta (2019, *RFS*) | 影子交易；内幕信息的传播网络；知情交易的市场特征 | RQ4 的扩展；知情交易的度量方法 |

**动手前的最后检索**（SSRN、Google Scholar、NBER）：
`"background of the merger" leak` · `confidentiality agreement informed trading` · `private negotiations option volume` · `sale process insider trading` · `takeover auction information leakage` · `NDA merger trading`。
如果发现高度重合的工作，§3.10 列出了同一数据平台上的替代题目。

### 3.4 数据构建

1. **样本**：2001–2025 年美国上市目标公司的并购（来自 SDC，或 WRDS 上的 Capital IQ），要求能匹配 CRSP，交易额高于某个阈值。
2. **文件**：从 EDGAR 获取 DEFM14A、PREM14A、DEFM14C、SC 14D9、SC TO-T（含要约收购书附件）、S-4 / 424B3、SC 13E3。每笔交易取最终版本；公告后撤回的交易取初步版本。
3. **章节切分**：用正则表达式定位 “Background of the Merger / Offer / Transaction”；少数失败的文件交给大模型兜底。
4. **大模型抽取**：每个事件输出一条 JSON 记录，且必须附上原文证据句。这样既便于核查，也能防止模型编造。示意：

```json
{
  "deal_id": "...",
  "events": [
    {
      "date": "2019-03-04",
      "date_precision": "day | month | range",
      "type": "first_contact | nda_signed | mgmt_presentation | data_room | ioi | bid | exclusivity | financing_commitment | advisor_engaged | board_meeting | special_committee | go_shop | agreement_signed | leak_or_rumor",
      "counterparties": ["Party A"],
      "counterparty_type": "strategic | financial | unknown",
      "n_parties": 1,
      "price_per_share": null,
      "evidence": "On March 4, 2019, Party A executed a confidentiality agreement ..."
    }
  ],
  "deal_level": {
    "initiator": "target | bidder | third_party",
    "n_contacted": 25, "n_nda": 12, "n_ioi": 5, "n_final_bids": 2,
    "go_shop": false, "special_committee": true
  }
}
```

5. **验证**（审稿人一定会问）：
   - 人工编码 300 笔（随机抽样加分层抽样，最好两人独立编码），按字段报告精确率、召回率和日期误差；
   - 用两个不同的大模型分别抽取，报告一致率；
   - 用开源权重模型复现主要变量，以符合期刊对 AI 工具的披露要求；
   - 把汇总统计（NDA 数量分布、流程时长等）与既有手工样本对照。
6. **市场数据**：
   - CRSP 日度数据；
   - OptionMetrics：按期限和价内外程度拆分的成交量、未平仓量、隐含波动率与偏斜；
   - TAQ 或 WRDS Intraday Indicators：订单不平衡、交易规模分布；
   - RavenPack：新闻与传闻，用于剔除公开信息；
   - 13F：对冲基金持股变化；
   - SEC 诉讼公告：内幕交易案件，可用大模型匹配到具体交易；
   - Hoberg–Phillips TNIC：同业公司，用于 RQ4。

### 3.5 识别策略

核心思想是**在同一笔交易内部，用流程事件发生的时点做识别**，而不是比较不同的交易。

```
Y[d,t] = β · log(1 + Outsiders[d,t]) + α[d] + λ[τ(d,t)] + δ[t] + X[d,t]'γ + ε[d,t]
```

- `Y`：异常期权成交（短期价外看涨期权的占比）、异常股票成交或订单不平衡、累计异常收益。
- `Outsiders[d,t]`：截至 t 日已签 NDA 的外部方累计数量（可以再加上融资银行和顾问）。
- `α[d]` 是交易固定效应。`λ[τ]` 是距公告日的事件时间固定效应，用来吸收“越接近公告交易越活跃”的一般趋势。`δ[t]` 是日历日期固定效应或市场因子。
- **堆叠事件研究**：以每一批 NDA 的签署日为事件，比较 [-10,-1] 和 [0,+10] 两个窗口，并检查事前趋势。
- **安慰剂检验**：只涉及内部人的事件（董事会会议、成立特别委员会）不应带来异常交易；签 NDA 后很快退出的对手方，效应应该更小。
- **排除公开信息**：剔除 RavenPack 中有传闻或“战略评估”公告的交易，或者把它们作为对照组。
- **后果检验（RQ3）**：在交易内部比较两类价格变化对后续出价修正的影响：一类发生在 NDA 事件之后，一类来自财报等公开新闻。如果买方只对后者提价，说明他们能识别并过滤掉泄露。

### 3.6 可能的卖点

1. 第一个覆盖 2001–2025 年全样本、事件级的私下并购流程数据集。论文接收后公开，可以带来长期引用。
2. 直接回答泄露从哪里来、在什么时候发生，解释 Augustin 等 (2019) 中说不清的那部分知情交易。
3. 重新检验 costly feedback，并量化卖方“扩大竞价范围”与“泄露”之间的权衡。
4. 对 SEC 执法（异常交易筛查、影子交易）和交易保密实务有直接的政策含义。

### 3.7 审稿人会问什么

| 质疑 | 应对 |
|---|---|
| 这是泄露，还是市场的理性预期？ | 交易内部的事件时点识别，加上事件时间固定效应、安慰剂检验、剔除有公开传闻的交易；知情交易的特征（短期价外看涨期权）也能帮助区分 |
| 大模型抽取有误差 | 证据句、人工验证集、双模型一致率；主回归只用日期精确到天的事件；测量误差通常导致衰减偏误，结果偏保守 |
| 只能看到最终公告的交易，看不到没有公告的失败流程 | 如实说明；纳入公告后撤回的交易；讨论偏误方向 |
| 披露中的对手方是匿名的（Party A、Party B） | RQ1–RQ3 只需要对手方类型和数量；RQ4 使用目标公司的同业，而不是具体买方 |
| NDA 的签署时点本身是内生的 | 用堆叠设计检查事前趋势；控制签署前的公开新闻；按流程阶段（第几轮）做异质性分析 |

### 3.8 成本与存储（粗略估计）

- **大模型费用**：约 5,000 笔交易，背景章节平均约 1 万 token，每跑一轮约 5,000 万输入 token。算上双模型、重跑和验证，费用在几百到两三千美元之间，取决于模型和是否使用批处理接口。
- **存储**：EDGAR 原始文件不到 50GB，抽取结果不到 5GB；期权数据先在 WRDS 上聚合到“日度×合约类别”，不到 20GB；TAQ 只下载日度指标，不到 10GB。合计远低于 1TB。

### 3.9 90 天路线图

| 周 | 任务 | 交付物 |
|---|---|---|
| 1–2 | 核对 WRDS 订阅（OptionMetrics、TAQ、RavenPack、SDC 或 Capital IQ）；按 §3.3 的关键词做最终文献检索；确定样本定义 | 一页研究设计和文献地图 |
| 3–5 | EDGAR 下载与章节切分；在 100 笔交易上试抽取，同时人工编码这 100 笔；迭代 schema 和提示词 | 抽取管线 v1 和准确率报告 |
| 6–8 | 全样本抽取；验证集扩大到 300 笔；构建“交易×日”面板 | 事件级数据集 v1 |
| 9–11 | 对接期权和股票数据；画出 NDA 事件前后的异常期权成交图（这是最关键的一张图） | 主图和主回归 |
| 12–13 | **决策点**：信号清晰就开始写工作论文；不清晰就转向 §3.10 的题目 | 继续或转向的决定 |

### 3.10 主假设不成立时：同一平台上的替代题目

- **A2：投行估值中的折现率**（见 §4，也建议作为第二篇）；
- 出售方式（拍卖、谈判、“谈判式拍卖”）与溢价的关系，交易保护条款（go-shop、终止费）是否有效；
- 特别委员会与控股股东交易的流程质量（可结合 2025 年特拉华 SB 21 修法；目前修法后的样本偏小）；
- 目标公司 CEO 在谈判中的私人利益（留任安排、雇佣协议出现的时点）与收购价格的关系。

---

## 4. 方向 A2：投行估值中的折现率，利率传导与黏性

同一批文件的 “Opinion of Financial Advisor” 章节会披露 DCF 的折现率区间、永续增长率和退出倍数，常常还有 WACC 的构成（无风险利率、beta、股权风险溢价 ERP、规模溢价）。

- **问题**：执业者使用的折现率是否随无风险利率一对一变动？是否像 Gormsen & Huber (2025, *AER*) 发现的企业门槛利率那样具有黏性？折现率变化是否影响估值区间和成交溢价？
- **准自然实验**：很多投行按 Kroll（原 Duff & Phelps）的推荐值设定 ERP 和“正常化”无风险利率，而 Kroll 会离散地调整推荐值：
  - 正常化无风险利率：2020-06-30 从 3.0% 降到 2.5%；2022-04-07 升到 3.0%；2022-06-16 升到 3.5%；
  - 推荐 ERP：2020-12-09 从 6.0% 降到 5.5%；2024-06-05 从 5.5% 降到 5.0%。

  比较“引用 Kroll 正常化利率的顾问”和“使用即期利率的顾问”在这些日期前后的变化，就能得到与目标公司基本面无关的折现率冲击。
- **最接近的文献**：
  - Gormsen & Huber (2025, *AER*)：企业的门槛利率及其黏性；
  - Shaffer (2023, *MS*)：第三方基本面估值是否影响收购结果；其博士论文发现估值方会事后调整折现率，去迎合谈判出来的价格；
  - Imperatore, Pündrich, Verdi & Yost (2024, *JAE*)：诉讼风险下的策略性估值；
  - Dessaint, Olivier, Otto & Thesmar (2021, *RFS*)：基于 CAPM 的误估；
  - Kisgen, Qian & Song (2009, *JFE*)；Cain & Denis (2013, *JLE*)。
- **差异**：采用宏观金融视角（利率传导与黏性），并利用 Kroll 推荐值的离散调整做识别。
- **风险**：Shaffer 已经有公平意见书数据，存在被抢先的可能。动手前先读完他的全部相关论文。

---

## 5. 方向 B（时间窗口型）：2026 年 11 月的半分钱最小报价单位

- **事实**：
  - SEC 在 2024 年修订了 Reg NMS Rule 612：股价不低于 1 美元、且评估期内时间加权平均报价价差不超过 1.5 美分的股票，最小报价单位由 1 美分降为 0.5 美分；同时访问费上限由每股 0.3 美分降到 0.1 美分。
  - 该规则原定 2025-11-03 生效。SEC 于 2025-10-31 发布豁免令，把生效日推迟到 **2026 年 11 月的第一个工作日**。D.C. 巡回法院已维持该规则。
  - 同批修订中，整手（round lot）的重新定义已于 2025-11-03 生效；SIP 发布零股（odd-lot）信息的要求于 2026 年 5 月生效。
- **设计**：以评估期内的价差为驱动变量，在 1.5 美分处做断点回归（RDD）。之后每半年重新评估一次，“换档”的股票可以做 DiD。结果变量包括报价价差、有效价差、实现价差、NBBO 深度、排队长度、零股和次便士成交、场外成交占比、价格效率。数据用 WRDS TAQ，在 WRDS Cloud 上计算。
- **现在就能做的事**：
  - 用 2026 年 1–10 月的数据搭好管线，模拟分组；
  - 在规则生效**之前**，把带时间戳的预分析计划（pre-analysis plan）挂到 OSF 或 SSRN。金融学很少有人对已知的政策冲击做预注册，这能增加可信度。
- **风险**：
  1. 规则仍可能再次推迟，或被 SEC 更大范围的 Reg NMS 改革取代；
  2. 竞争激烈，SEC 的经济研究部门（DERA）和微观结构领域的资深学者都会研究；
  3. TAQ 数据处理量大。

  文献基准：Werner, Rindi, Buti & Wen (2023, *MS*)；SEC DERA 对 Tick Size Pilot 的再评估。
- **备选**：如果再次推迟，可以研究已经发生的 2025-11 整手重新定义（250、1,000、10,000 美元的价格门槛构成断点）。不过 Nasdaq、Cboe、BMLL 已经发布过行业分析，受影响的股票只有约 250 只，样本偏小。

---

## 6. 方向 C：全机构投票数据与代理顾问行业变局

- **新数据**：修订后的 Form N-PX 自 2024-07-01 生效。所有 13F 机构须以 XML 结构化格式披露 say-on-pay 投票；SEC 估计涉及 8,000 多家管理人，包括对冲基金、银行、保险公司、养老金和家族办公室。基金则按系列披露全部投票。截至 2026-08-31，已有 2024、2025、2026 三个投票季的数据。
- **冲击**：
  - JPMorgan 资产管理在 2026 年一季度改用内部 AI 工具（Proxy IQ），不再使用代理顾问；
  - Glass Lewis 宣布到 2027 年停止发布基准政策建议；
  - 2025 年 12 月的行政令要求审查代理顾问；
  - SEC 从 2025-11-17 起，对大多数 Rule 14a-8 no-action 请求不再作实质回应。
- **问题**：
  - 公募基金以外的机构如何投票？
  - 几乎完全跟随代理顾问的“机器人投票”有多普遍，代理顾问的真实影响有多大？
  - 用 AI 替代代理顾问之后，投票是否更偏向管理层、与 ISS 建议的差异是否变大、是否影响投票结果和薪酬设计？
- **文献**：Shu (2024, *JFE*，识别基金的代理顾问客户关系，数据已公开)；Malenko & Malenko (NBER w31636, *Voting Choice*)；Barry（工作论文，利用 2025 年 13G 指引冲击研究股东沟通）。
- **风险**：部分冲击只涉及单一机构，识别偏弱，容易写成描述性论文。更适合用“AI 与公司治理”的叙事投 MS。

---

## 7. 方向 D：Form 144 电子化，内部人卖出的“事前披露”自然实验

- **事实**：2023-04-13 起，Form 144 必须通过 EDGAR 电子提交；此前大多是纸质文件，公众很难及时获取。关联人在三个月内出售超过 5,000 股或超过 5 万美元时，必须提交。
- **问题**：
  - 实时公开的事前披露是否削弱了内部人的信息租金？
  - 市场是在 144 表提交时就作出反应，还是要等到 Form 4 披露成交之后？
  - 内部人是否开始把交易压在门槛以下（bunching），或者转向 10b5-1 计划？

  这直接检验了 Fried (1998) 提出的“交易前披露”政策建议。
- **识别**：政策前后对比，结合门槛附近的比较（刚好高于和低于 5,000 股或 5 万美元）；用 Form 4 上的 10b5-1 勾选项区分计划内交易。
- **混杂与竞争**：10b5-1 修订（2023-02-27 生效）几乎同时发生；arXiv 上已有一篇 2026 年 2 月的相关论文，研究 Form 144 与 Form 4 之间的“报告倒挂”。适合作为一篇较快完成的第二篇论文，但对 JFQA 来说题目可能偏窄。

---

## 8. 查过但不建议现在进入的方向

| 方向 | 已有的强竞争工作 | 判断 |
|---|---|---|
| 半流动私募信贷基金（非交易型 BDC）的挤兑与赎回限制 | Fang, Goldstein & Zeng (2026, NBER w35385) | 核心问题已被占 |
| 同一笔私募贷款在不同 BDC 之间的估值差异 | Jang & Kim (2025, SSRN 5126860) | 已被占 |
| 官方统计的可信度（2025 年 BLS 局长被解雇） | Bloom, Groshen, Hobbs & Strain (2026, NBER w35135) | 已被占；2025 年政府停摆造成的数据空窗很可能也会被覆盖 |
| 用 Kalshi 等预测市场衡量宏观预期 | NBER w34702，以及美联储的相关研究 | 由美联储和宏观金融研究者主导 |
| 单股杠杆 ETF | Huang (2025)；Kim, Han & Won (2026)；Bessembinder (2025)；Zhao (2026) | 2025–26 年迅速拥挤 |
| 加密资产财库公司（MSTR 等）的 NAV 溢价 | Andrade, Coomes & Duarte (2026) 等 | 竞争中等偏高，审稿人可能视为短期现象 |
| 2025 年 13G 指引冲击与股东沟通 | Barry（工作论文） | 已有人使用同一冲击 |
| 基金致股东信的文本分析 | Hillert, Niessen-Ruenzi & Ruenzi (2025, *MS*)；Cao, Yang & Zhang | 已拥挤 |
| 体育博彩合法化与散户投机 | *Journal of Financial Markets* 2026 年的相关论文等 | 已被占 |
| 用大模型情绪或摘要预测股票收益 | 大量已有工作；前视偏差争议（Glasserman & Lin 2023；Sarkar & Vafa 2024） | 除非有方法论贡献，否则不建议 |
| 2025 年关税冲击的事件研究、泛 ESG/气候、SPAC、COVID | 未逐一核实，属于明显的热点或已过时的方向 | 不建议作为第一篇 |

**如果你同时有 Wind 或 CSMAR**：以下两个都是干净的自然实验：
- 北向资金数据停止披露（2024 年 5 月停发实时数据，8 月停发每日数据）；
- 证监会的市值管理指引（2024 年 11 月发布，市净率连续 12 个月低于 1 的公司须披露估值提升计划）。

但这两个事件都已经出现了一批相关论文，日本东证的 PBR 改革也已有断点回归研究。如果要做，需要尽快行动，并找到差异化的角度。

---

## 9. 数据工程与大模型使用规范（1TB 约束下）

- **存储布局**：`raw/`（EDGAR 原文，只读）→ `interim/`（切分后的章节、大模型原始输出）→ `clean/`（Parquet）→ `analysis/`。用 DuckDB 或 Polars 直接读 Parquet，不必建数据库。
- **TAQ 与期权**：在 WRDS Cloud 上用 SAS 或 Python 聚合，只下载聚合后的结果。
- **EDGAR**：遵守 SEC 的公平访问规则（声明 User-Agent，每秒不超过 10 次请求），建立本地索引和缓存。
- **大模型抽取规范**：
  1. 固定模型版本号并记录调用日期，temperature 设为 0，缓存所有请求与响应；
  2. 每个抽取字段都附原文证据，并自动校验证据是否确实出现在原文中；
  3. 人工验证集、双模型一致率、开源模型复现，三者都要有；
  4. 提示词和 schema 纳入版本控制，与代码一起放进复现包；
  5. 不要把 WRDS 授权数据发给外部 API，除非许可明确允许。
- **Git**：只提交代码和文档，不提交数据。从第一天起就让别人能用一条命令复现整个管线（Makefile 或 Snakemake）。

---

## 10. 投稿策略

- **方向 A**：首选 JFQA。并购和知情交易是 JFQA 的常规主题，JFQA 最近也发表过申报代理与信息泄露的论文。如果主要卖点是“大模型构建的新型流程数据”和组织设计上的含义，MS 也合适。
- **方向 B**：JFQA、MS 都可以（MS 在 2023 年发表过 tick size 论文）。
- **方向 C**：更适合 MS。
- **方向 D**：JFQA，或更专门的期刊。
- **节奏**：6–9 个月内出工作论文，挂到 SSRN，并投主要会议（AFA、WFA、EFA、NFA、FMA、SFS Cavalcade 等）；根据反馈修改后再投期刊。第一次冲 JFQA 或 MS 时，找一位发表过并购或知情交易论文的合作者，会明显提高成功率。

---

## 11. 下一步

需要的话，我可以继续：
1. 写 EDGAR 合并委托书的下载代码和 “Background” 章节的切分代码；
2. 设计大模型抽取的提示词和 schema，在 50 笔交易上试跑，并出一份准确率报告；
3. 按 §3.3 的关键词做一次更系统的文献地图，确认方向 A 没有被抢先。

---

## 参考资料（本次检索用到的主要来源）

**期刊政策**
- JFQA 代码共享政策：https://jfqa.org/submissions/jfqa-code-sharing-policy/
- Management Science 数据与代码披露政策：https://pubsonline.informs.org/page/mnsc/code-and-data-disclosure-policy

**方向 A / A2**
- Augustin, Brenner & Subrahmanyam (2019, *MS*)：https://pubsonline.informs.org/doi/10.1287/mnsc.2018.3122
- Betton, Eckbo, Thompson & Thorburn (2014, *JF*)：https://onlinelibrary.wiley.com/doi/abs/10.1111/jofi.12151
- Eckbo, Norli & Thorburn (2026), *Who Initiates Takeovers?*：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7194259
- Eckbo, Malenko & Thorburn (2026), *Corporate Takeovers: Theory and Evidence*：https://www.ecgi.global/sites/default/files/2026-01/corporate-takeovers.pdf
- Schubert (2020)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3640480
- Liu & Officer：https://ssrn.com/abstract=3383209 ；Liu, Officer & Tu：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4422506
- Masulis & Simsir (2018, *JFQA*)：https://ideas.repec.org/a/cup/jfinqa/v53y2018i06p2389-2430_00.html
- Boone & Mulherin (2007, *JF*)：https://www.ssrn.com/abstract=642306
- Liu & Mulherin (2018, *JCF*)：https://www.sciencedirect.com/science/article/abs/pii/S0929119917306922
- Bargeron, Clifford & Qiu (2025, *JFQA*)：https://www.cambridge.org/core/journals/journal-of-financial-and-quantitative-analysis/article/filing-agents-and-information-leakage/2DC0ECC2930E8E19EFF353F1FFDA31A1
- Dai, Massoud, Nandy & Saunders (2017, *JCF*)：https://www.sciencedirect.com/science/article/abs/pii/S0929119917301165
- Hasan 等 (2025, *JFR*)：https://onlinelibrary.wiley.com/doi/10.1111/jfir.12420
- Mehta, Reeb & Zhao (2021, *TAR*)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3689154
- Ahern (2017, *JFE*)：https://www.sciencedirect.com/science/article/abs/pii/S0304405X17300570
- Kacperczyk & Pagnotta (2019, *RFS*)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2695197
- Ahern & Sosyura (2015, *RFS*)：https://academic.oup.com/rfs/article-abstract/28/7/2050/1592275
- Gormsen & Huber (2025, *AER*)：https://www.aeaweb.org/articles?id=10.1257%2Faer.20231246
- Shaffer（估值研究与博士论文）：https://matthewshaffer.online/research.html
- Imperatore, Pündrich, Verdi & Yost (2024, *JAE*)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3869109
- Dessaint, Olivier, Otto & Thesmar (2021, *RFS*)：https://academic.oup.com/rfs/article/34/1/1/5836312
- Kroll 推荐 ERP 与无风险利率的历史：https://www.kroll.com/en/reports/cost-of-capital/recommended-us-equity-risk-premium-and-corresponding-risk-free-rates

**方向 B**
- SEC 2024 年规则新闻稿：https://www.sec.gov/newsroom/press-releases/2024-137
- SEC 关于 Reg NMS 合规日期的豁免令：https://www.sec.gov/newsroom/press-releases/2025-130-sec-issues-exemptive-order-regarding-compliance-certain-rules-under-regulation-nms
- Werner, Rindi, Buti & Wen (2023, *MS*)：https://pubsonline.informs.org/doi/abs/10.1287/mnsc.2022.4502
- SEC DERA, *Tick Sizes and Market Quality: Revisiting the Tick Size Pilot*：https://www.sec.gov/files/dera_wp_ticksize-pilot-revisit.pdf
- 整手改革的行业分析：https://www.nasdaq.com/articles/new-round-lots-helped-decrease-spreads ；https://www.cboe.com/insights/posts/the-impact-round-lot-reform-had-on-u-s-equities-market-quality

**方向 C**
- N-PX 修订（SEC 新闻稿）：https://www.sec.gov/newsroom/press-releases/2022-198
- Shu (2024, *JFE*) 公开数据：https://github.com/chongshu/proxy-advisor-customers
- Malenko & Malenko, *Voting Choice*：https://www.nber.org/system/files/working_papers/w31636/w31636.pdf
- JPMorgan 改用 AI 投票：https://www.esgdive.com/news/jpmorgan-drops-proxy-advisers-for-internal-ai-tool-u-s-proxy-voting-decisions/808993/
- SEC 关于 14a-8 流程的声明：https://www.sec.gov/newsroom/speeches-statements/statement-regarding-division-corporation-finances-role-exchange-act-rule-14a-8-process-current-proxy-season
- Barry, *Beyond the Ballot*：https://johnwbarry.info/files/papers/engagement_voting.pdf

**方向 D**
- Form 144 电子化（Cooley）：https://www.cooley.com/news/insight/2023/2023-04-12-form-144-goes-digital
- arXiv 2602.17890（Form 144 与 Form 4）：https://arxiv.org/html/2602.17890

**第 8 节中的竞争论文**
- Fang, Goldstein & Zeng (2026)：https://www.nber.org/papers/w35385
- Jang & Kim (2025)：https://papers.ssrn.com/sol3/Delivery.cfm/5126860.pdf?abstractid=5126860&mirid=1
- Bloom, Groshen, Hobbs & Strain (2026)：https://www.nber.org/papers/w35135
- *Kalshi and the Rise of Macro Markets*：https://www.nber.org/system/files/working_papers/w34702/w34702.pdf
- 单股杠杆 ETF：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5691524 ；https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7030278 ；https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5369417 ；https://arxiv.org/pdf/2608.03703
- Andrade, Coomes & Duarte (2026)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5434457
- Hillert, Niessen-Ruenzi & Ruenzi (2025, *MS*)：https://pubsonline.informs.org/doi/10.1287/mnsc.2021.03417
- Cao, Yang & Zhang：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3713966
- *Do retail traders gamble on stock options?* (2026)：https://www.sciencedirect.com/science/article/abs/pii/S1386418126000170
- Glasserman & Lin (2023)：https://arxiv.org/abs/2309.17322 ；Sarkar & Vafa (2024)：https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4754678
