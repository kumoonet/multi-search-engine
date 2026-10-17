# Multi Search Engine — 多元搜索方法论手册

**版本**：v3.1.0
**一句话**：意图驱动的四层深度搜索方法论（侦查→解构→策略→执行→证据板综合），综合前三道质检（覆盖度/引用抽查/查询日志），层 3 结果按档位默认经**子代理加工层**（四免费傀儡 agnes-2.5/glm-4-flash/agnes-2.0/lite，难度路由 route + 双交叉 + 主模型仲裁 + 三层兜底）粗加工后进证据板（增强不削弱，证据标准不变），**Exa 语义搜索通道**（MCP 神经语义匹配）补三处：层 0 侦查语义探路、⑤.8 语义雪球、⑤.5 描述性查询重构，**v3.0.0 新增 BrowserAct 浏览器自动化第二执行范式**（JS渲染/登录态/验证码/交互四大死区，层 3 降级链扩为四级：重试→回声→浏览器兜底→fallback），**v3.1.0 吸收 tanzi 采集 Skill 三项设计**：证据板「取回状态」列（全文/仅摘要/失败，把证据诚实从纪律变成字段）、关 1 信源类别覆盖核对（防转载池假交叉）、F.5 解析锚点 fixture 回归（测"渠道活着但解析已坏"），渠道数据外置 `engines.yaml`，状态可用 `selftest.py` 一键刷新。零 API Key（语义通道免费层无 Key，深度功能需 Key）。

> ⚠️ **网络出口前提**：渠道可达性是（渠道 × 网络出口）的函数，不是渠道固有属性。执行前按 `engines.yaml` 的 `status_cn_direct` 列（大陆直连实测）或层 3 预检确认；状态过期就双击 `selftest.bat` 刷新。

## ⓪ 三件套结构（本文件是唯一入口）

| 文件 | 角色 | 谁读 |
|------|------|------|
| **本手册**（Multi-Search-Engine_v3.1.0.md） | 方法论：怎么理解意图、怎么拆、怎么搜、怎么验证、怎么管理证据 | Agent，每次搜索任务 |
| **engines.yaml**（同目录） | 渠道注册表：45 条目（43 web_fetch 渠道 + 1 Exa 语义 MCP 通道 + 1 BrowserAct 浏览器 CLI 通道）的 URL/状态/直连/降级/意图/探活标记 + 路由组 + 分级 + **executor 执行器字段（v3.0.0）**，机器可读单一事实源 | Agent，层 3 执行时按需加载 |
| **selftest.py**（同目录，双击 selftest.bat 运行） | 状态刷新器：逐渠道探活并把 `status_cn_direct` 写回 yaml。与解析锚点 fixture 回归互补（F.5）——探活测"渠道活着没"，fixture 测"解析锚点对不对" | 人，怀疑状态过期时手动触发 |

**引用规则**：手册中所有渠道 id（如 `baidu`、`bing_intl`、`openalex`、`exa`、`browser_act`）与路由组名（如 `default_zh`、`restricted_fallback_en`）均指向 engines.yaml 中的定义。URL/状态/降级**只在 yaml 维护**，本手册不重复。

---

## ① 执行声明（Agent 必读）

### 四层架构（同一 Agent 内顺序执行 + 回退循环）

| 层 | 职责 | 依据 |
|----|------|------|
| **层 0 侦查搜索**（条件触发） | 对陌生领域先做 1-3 次廉价探路查询，摸清概念地图/争议点/行话术语 → 产出 research brief（含**语义探路**快捷通道，见下） | 本节「层 0 详解」 |
| **层 1 意图解构** | 识别 query_type（技术/学术/新闻/微信/投研场景）+ 检测语言 + 区分两类歧义 → 查 ② 路由表 → 确定渠道组 | ② 意图路由表 + engines.yaml `route_groups` |
| **层 2 策略构造** | 基于意图 + 方法论，产出搜索词（拆解/定向过滤/逆向构造/**跨语言适配**/**描述性查询**）与信源策略；`deep` 档额外产出**实体追踪表**（锁定关键人/机构/概念跨平台追踪，见 ⑤.9） | ⑦ 投研方法论 + ⑤.5 查询构造建议 + ⑤.9 实体追踪 |
| **层 3 策略执行** | 加载 engines.yaml → **可达性预检**（候选渠道 ≤3s 轻探活；深度探活用 selftest.py 的 10s 标准，见 F.5，或直接采信 `status_cn_direct` 列）→ 逐渠道 `web_fetch`（1-2s 延迟），失败先重试 1 次，仍失败则**回声搜索**（提取标题/关键句投其他渠道找同内容镜像，见 ⑤.7），回声仍失败走**浏览器兜底**（browser-act 重放同一 URL，见 ⑤.10，v3.0.0 新增），浏览器兜底仍失败才走 `fallback`；找到 L1/L2 高价值深度文章时触发**滚雪球搜索**（沿引用链/外链扩展发现 + **语义雪球**，见 ⑤.8）；**关键事实写入证据板** | engines.yaml + ④ 调用分级 + 证据板 + ⑤.8 滚雪球 + ⑤.10 浏览器兜底 |

> **回退循环**：层 3 结果不足 → 回层 2 重构搜索词；意图漂移 → 回层 1 重解构；发现自己根本不懂这个领域 → 回层 0 补侦查。搜索是迭代不是流水线，禁止线性走完即止。

> **反思三问（每轮必答，v2.3.1 新增；v2.5.0 增补 learnings 追踪）**：每执行完一轮查询、进入下一轮前，必须回答——① 本轮学到了什么**新**事实？（写入证据板）② 距离回答用户问题还缺什么关键信息？③ 下一轮查询为什么这样构造？（必须用新行话/新角度，禁止同义重复上一轮查询）。三问答不出实质内容 → 触发强制规则 7 饱和即停。
>
> **Learnings 查询树（v2.5.0，`deep` 档启用）**：反思三问的①产出不仅是证据板行，还应提炼为**learnings 列表**——每轮搜索后记录：新发现的行话/术语、新出现的实体名、新暴露的子问题角度。下一轮查询**必须**用 learnings 列表中的新词构造（禁止同义重复），形成"搜索→学习→用新知识重构查询→再搜索"的递归扩展树。这与 IterResearch 的 learnings 机制和 DeepRAG 的二叉树递归搜索同构。成本门控：learnings 列表上限 10 条，超出时淘汰最旧的；`standard`/`quick` 档不强制维护 learnings 列表，反思三问仍必答。

### 层 0 详解：侦查搜索（v2.3.0 新增）

**两类歧义，两种处理**——不要混淆：

| 歧义类型 | 判别 | 处理 |
|---------|------|------|
| **用户意图不明**（"帮我查查这个"到底要什么） | 缺的是用户脑子里的信息 | **问用户**，不要搜。搜索解决不了需求歧义 |
| **领域陌生**（意图清楚但不懂领域行话/结构） | 缺的是世界知识 | **侦查搜索**：1-3 次泛查询摸地图，再进层 1 |

> **语义探路（v2.9.0 新增）**：侦查搜索新增一条快捷通道——把**用户原话（大白话）**直接投 Exa 语义通道（`web_search_exa`，1 次），按意义匹配返回语义相近页面，从结果标题/摘要中提取行话/实体/权威信源，**替代或补充** baidu/bing_intl 泛搜。关键词搜索的死穴是"不知道关键词就搜不到"（用户原话往往不含领域行话，如"先进封装"≠"CoWoS"），语义探路绕过这一环：描述性查询按意义命中，行话从结果里自然暴露。**成本门控**：计入侦查查询次数（deep 标配 2-3 次侦查内含此 1 次；standard 领域陌生时 1 次侦查可用；`quick` 档仍禁用层 0）。语义通道经 MCP 工具调用，非 web_fetch，见 engines.yaml `semantic` 组。

**侦查搜索产出 = research brief**（进入层 1 的输入）：

```
research brief
├─ 核心概念与同义词/行话（如 "CoWoS" = 台积电先进封装）
├─ 关键争议点/分歧叙事（后续 D.2 对立族的素材）
├─ 权威信源清单（哪些站点/机构在这个领域是 L1/L2）
└─ 修正后的查询词表（用行话替换用户原话）
```

**合格样例**（问题「半导体库存周期到哪了？」）：
```
research brief（合格样例）
├─ 核心概念/行话：库存周期 = 库存销售比；产能利用率 = utilization；先进封装 = CoWoS/HBM
├─ 关键争议点：看多「AI 需求吸收产能」 vs 看空「消费电子去库未完成」（→ D.2 对立族素材）
├─ 权威信源：TrendForce/集微网（L2）、工信部/海关数据（L1）、雪球半导体板块（L3）
└─ 查询词表：「晶圆厂 产能利用率 Q2」「CoWoS 扩产 进度」「芯片 渠道 涨价」「消费电子 库存销售比」
```

**合格线**：四项均非空；查询词表 ≥3 条且 ≥一半用行话替换了用户原话；信源清单至少含 1 个 L1/L2。不满足 = 侦查不足，补 1 次侦查查询。

**成本门控（防滥用）**：
- `quick` 档：禁用层 0，直接层 1
- `standard` 档：仅当领域完全陌生时用 1 次侦查查询
- `deep` 档：标配 2-3 次侦查查询，brief 必须落纸（写进最终报告的方法论部分）
- 侦查用最泛渠道（`baidu`/`bing_intl` 各 1 次，可选 + Exa 语义探路 1 次）即可，禁止在层 0 展开全渠道

> 设计出处：本层综合 STORM 的 Perspective-Guided Question Asking（先发现视角再提问）与 LangChain open_deep_research 的 Scope 阶段（正式研究前先产出 research brief），实证笔记见同目录《深度搜索架构调研笔记.md》。

### 证据板（层 3 与最终综合之间的工作记忆，v2.3.1 新增）

**为什么需要**：搜索结果直接堆进上下文会互相污染，最终综合时找不回关键事实与出处。2026 前沿共识：深度研究的性能关键不在搜得多，而在证据管理与何时停止（IterResearch 每轮重构工作区、dzhng/deep-research 的 learnings 机制均为此设计）。

**格式（每轮搜索后更新；「取回状态」列 v3.1.0 新增，deep 档必填、standard 档可选）**：

| 事实/数据点 | 来源（渠道+URL） | 信源层级 | 独立信源数 | 置信度 | 取回状态 |
|------------|----------------|:------:|:--------:|:------:|:------:|
| 例：CoWoS 月产能 2026 底达 8 万片 | bing_intl → trendforce.com/... | L2 | 2 | 中 | 全文 |

**规则**：

- 只录**关键事实**（带量纲数据/结论/反例），不录背景噪音
- **取回状态（v3.1.0 新增，吸收 tanzi「正文状态」设计）**：每行标注该事实的取得方式——`全文`（读到来源页正文）/ `仅摘要`（只来自 SERP 摘要、snippet 或加工层提炼）/ `失败`（取回被拒）。**「仅摘要」行不得直接作带量纲结论**——须升级为全文或经强制规则 5 降级链/关 2 核验后再用。此字段把"不把摘要写成全文结论"从审计环节（关 2）前移到写入环节；09-24 实测中 so360 SERP 常态只出标题+摘要，该列即为日常风险打标
- 信源层级按 D.4 的 L1-L4 判定；同一事实每新增一个**独立**信源计数 +1（同一媒体转载不算独立）
- 独立信源数 ≥3 → 置信度「高」，可作结论；=2 →「中」，谨慎引用；=1 →「低」，只标注不定论（这是强制规则 6 交叉验证的落地载体）
- **加工层一致性可作置信度参考（v2.7.0）**：分类经加工层双模型交叉一致（agnes-2.5 + glm-4-flash 同判）→ 置信度在独立信源数基础上**升一档**（低→中、中→高）；不一致且仲裁判出胜者 → 按原档，标注「仲裁采纳」；**独立性判断不受影响**——加工一致只增强置信度，不增加信源数（规则 6 计数仍按原始渠道）
- **升档适用范围（2026-08-23 实测补充）**：置信度升档**仅适用于 classify 类别一致性**（判据完整、省 94% 但信息无损）；**summarize 摘要 / tags 打标结果不触发升档**——摘要丢信息，升档依据被压缩过，只能作参考不作升档理由
- **最终报告只从证据板生成**，禁止从原始搜索结果里现捞
- **语义通道产出的结构化提取（v2.9.0 可选增强，需 API Key）**：Exa `web_search_advanced_exa`（需 Key 解锁）的 `contents.summary` 支持 JSON schema 结构化抽取（如 `{fact, metric, value, date}`），可用作证据板行的**预制草稿**——但独立信源数/置信度赋值仍是 Agent 职责，语义通道单源不增加交叉验证计数（同规则 6）。免费层无此能力，跳过即可
- 成本门控：`quick` 免建；`standard` 简版（事实+来源两列，取回状态可选）；`deep` 完整六列（含取回状态）并随报告落纸

### 子代理加工层（v2.8.0 默认启用：难度路由 + 双交叉 + 主模型仲裁 + 档位成本门控）

**定位**：层 3 原始结果 → 证据板之间的瘦身环节（**默认启用**，见下「默认启用与豁免」）。用**四个免费子代理傀儡**（脚本 `<PuppetMaster>/free_models.py`，池见 PuppetMaster_Ver1.8.4）按难度路由、并行/交叉加工，再写入证据板。

> ⚠️ **增强不削弱原则（硬约束）**：加工层是默认增强，**不是替代**。证据板各列、信源层级（D.4）、交叉验证计数（规则 6）、三道质检全部保持不变。任何情况下不得为了"省钱"降低档位、跳过验证或稀释证据标准——**削弱整体能力是红线**。

**为什么需要**：层 3 原始结果直接堆进上下文会互相污染（见证据板引言）；多引擎对同一内容重复返回时，10 条噪音换 1 条判断，省窗口、省注意力、提前发现跑题。实测（2026-08-22）：搜「Alibaba 财报」首轮 10 条全是官网导航，summarize 一句点破"全是 B2B 官网页面" → 触发 ⑤.5 失败后重构 → 重构后 classify 从「科技」变「财经」，跑题一眼可辨。**省 token 量化锚点（2026-08-23 真实计费口径实测，见 PuppetMaster_Ver1.8.2 二、核心思考与实践）**：加工层输入密集操作全类型 total 均值省 64%——classify 省 94%、tags 92%、route 90%、summarize 82%。加工不是玄学，是可量化的账：输入密集任务（分类/打标/路由/摘要）真省 token；短文本（<100 字）不加工。

**六维增量价值（2026-08-23 实测，见 PuppetMaster_Ver1.8.2 二.2）**：省 token（全类型 total 省 64%）只是第 1 维，后 5 维更根本——**① 省 token**（防限流、防窗口不够）；**② 并行干活**（5 傀儡 = 5 子代理同时跑，主模型不排队，批量时间减半）；**③ 主模型专注度**（只读加工后精华，不被原文噪音稀释）；**④ 防限流**（各免费 API 独立窗口互不挤占，连续批量不触发单一模型 429）；**⑤ 窗口容量**（原文堆进上下文互相污染，加工后省窗口，长会话不爆）；**⑥ 异构视角**（多模型交叉 + 仲裁，防单模型偏见）。**加工层的本质是"注意力解放器 + 并行手脚"，不是省钱工具**——明白为什么值得，才不会被"省钱"误导降档（与下方红线"增强不削弱"互补：红线是"不许"，此节是"为什么值得"）。

**四个傀儡的分工**（稳定性优先，agnes-2.5-flash 实测可扛主力）：

| 傀儡 | 加工层角色 | 特性 | 使用档位 |
|------|----------|------|---------|
| `agnes-2.5-flash` | **主判**（相关性分类） | 推理模型，分类最准，跑题识别最强 | deep |
| `glm-4-flash` | **交叉验证**（分类/打标） | 128K 长文不截断，与 agnes 结果比对 | deep |
| `agnes-2.0-flash` | **打标 + 去重摘要** | 非推理、快，批量瘦身不拖速 | standard / deep |
| `lite` | **难度评估 + 兜底**（短文本高频） | 8K 窗口、2QPS，仅简单文本评估/最后兜底 | route / 兜底 |

**难度感知前置路由（v2.7.0 新增，依据 Dynamic Model Routing 综述）**：加工前先 `route` 评估层 3 结果整体难度——简单文本直接 lite/agnes-2.0 快跑（省成本），困难文本才动用 agnes-2.5 + glm 双交叉。成本降 60% 质量仅差 1 分（实测范式收益，见 Puppeteer范式研究简报）。

**三层兜底逻辑（免费模型会不稳定，必须兜底）**：

```
层 1（主链路）：agnes-2.5-flash 主判 + glm-4-flash 交叉验证，并行跑
        │ 两者一致 → 高置信，直接采用
        │ 两者不一致 → 主模型实质仲裁（v2.8.0 修订：原 judge/agnes-2.0 裁判同族偏差已废，依据 PuppetMaster_Ver1.8.2 §七）
        │     主模型读两个候选 + 必要原文，直接判哪个分类正确（实质判断，规避 position bias）
        │     确认跑题 → 触发 ⑤.5 信号；低风险批量分歧可用 judge 命令可选档（裁判须与候选跨家族 + 双序跑）
        ▼ 任一侧超时 / 429 / 报错
层 2（补位）：agnes-2.0-flash 顶上（非推理、快，不依赖推理模型稳定性）
        │ agnes-2.0 也挂且文本 ≤8K → lite 二次补位（2 QPS 限速）
        ▼ 仍失败
层 3（终极兜底）：当前对话主模型亲自执行该子代理任务
        （不依赖任何外部 API，绝不空转、绝不跳过加工）
```

> 层 3 是安全网：免费傀儡全挂时，主模型自己干分类/打标/摘要，保证加工层**永远不空转、绝不削弱证据标准**。**不一致仲裁直接由主模型实质裁决（v2.8.0 修订，原「judge（层2）→ 主模型（层3）」级联取消）**——傀儡裁判撞 LLM-as-Judge 两大偏差：self-preference 同族溢价 5-25 点（原 agnes-2.0 裁判与候选 agnes-2.5 同族）、position bias A 位占优 10-15 点；且仲裁本身即判断，判断权归主模型（依据 PuppetMaster_Ver1.8.2 §七，2026-08-28 范式扫描）。

**五个操作**（均经实测验证）：

| 操作 | 命令 | 作用 | 实测效果 |
|------|------|------|---------|
| 难度前置（v2.7.0） | `free_models.py route "<结果>"` | 评估加工难度，决定档位 | 简单→lite；困难→双交叉 |
| 相关性分类 | `free_models.py classify "<结果>" --model agnes-2.5-flash` | 判断结果类别 / 跑题信号（deep 档并行 glm 交叉） | 跑题"科技"→ 重构后"财经" |
| 不一致仲裁（v2.8.0 修订） | **主模型实质仲裁**（`free_models.py judge` 为可选省资源档） | 双交叉不一致时主模型直接判哪个正确 | 实质判断规避 position bias；judge 可选档须跨家族裁判 + 双序 |
| 主题打标 | `free_models.py tags "<结果>" --model agnes-2.0-flash` | 快速看出结果覆盖哪些主题 | 阿里巴巴,财报,香港,电商,京东 |
| 去重摘要 | `free_models.py summarize < 文件 --model agnes-2.0-flash` | 多引擎重复 → 单条精简 | 10 条官网导航 → 1 句判断 |
| 兜底（层2 二次补位） | `classify/tags/summarize --model lite` | 文本 ≤8K 时 lite 补位 | 2 QPS 限速，不参与关键判断 |

**deep 档分类双模型交叉**（提准不提弱）：并行跑 `agnes-2.5-flash` + `glm-4-flash`，一致→高置信直接采用；不一致→**主模型实质仲裁（v2.8.0 修订）**——读两个候选 + 必要原文直接判哪个正确，确认跑题即触发 ⑤.5 跑题信号。比单模型"说了算"更准，且证据板/三道质检零削弱。

**默认启用与豁免（v2.8.0）**：凡调用多元搜索（进入 ① 层 1-3），加工层按档位成本门控**默认执行**，**无需用户显式指示"调度傀儡模型"**——deep 档 route+双交叉+主模型仲裁+打标+摘要全开；standard 档难度前置+去重摘要；quick 档禁用（见下「成本门控」）。豁免仅两类：① 用户显式说"不用傀儡/跳过加工"；② 短文本（单条结果 <100 字，主模型直读，加工成本 > 收益，见上「短文本不加工」）。原「触发条件」转正为**必执行情形**（不因豁免条款跳过）：层 3 结果 ≥15 条且重复度高（多引擎同一内容）、原始结果总量超上下文预算、deep 档证据板写入前需结构化整理。

**短文本不加工（2026-08-23 实测）**：单条结果 <100 字 → 主模型直接读，**不进加工层**——加工成本 > 收益（extract/explain 等输出密集任务 prompt 倒挂 -104%/-115%，短文本加工是负收益）。

**成本门控**：`quick` 禁用；`standard` 难度前置 + 去重摘要（route + agnes-2.0 单模型，控成本）；`deep` 难度前置 + 分类双模型交叉 + 主模型仲裁 + 打标 + 去重摘要全开。注意 lite 限 2 QPS（批量加 `sleep 1`），glm-4-flash 单条 ≤10 万字（防 429 限流）。

**批量调用节流纪律（2026-08-23 实测踩坑）**：连续批量加工 >5 轮会触发 glm-4-flash 限流（429），脚本 `fail()` 抛 `SystemExit` 被 `except Exception` 漏捕 → 静默退出。**批量加工 >5 轮：循环间 sleep 2-4s；对 429/5xx 加重试；用 `except BaseException` 捕获傀儡异常**（PuppetMaster 注意事项第 13 条）。

**与证据板的关系**：加工只瘦身、不新增信源——来源列仍标原始渠道，信源层级按 D.4 判定不变，交叉验证计数（规则 6）不受影响。加工产物只是"读起来更快的原始结果"，判断权永远在 Agent。

**红线（不削弱原则落地）**：
- 子代理只做粗加工；相关性判定、信源层级、置信度赋值永远是 Agent 职责
- 加工不等于信源：转载/镜像经加工后仍是转载，D.4 判定不变
- 不得为省成本降低档位、跳过三道质检或规则 6
- 子代理输出仅供 Agent 参考，Agent 有最终否决权
- **输出密集型产出（translate/proscons/explain/extract 等，2026-08-23 新增）**：prompt 倒挂实测 -38%~-115%（傀儡输出比输入长，可能脑补/错译），**一律主模型校验后采纳，不直接引用**——与输入密集（classify/tags/summarize）不同，输出密集省的是主模型生成，质量风险更高
- **三层兜底缺一不可**：层 3 主模型兜底必须可用，禁止"傀儡全挂就跳过加工"

### 三道质检（综合成报告前，v2.4.0 新增）

搜索做完 ≠ 研究做完。写报告前过三关（`deep` 档三关全过，`standard` 至少过关 1，`quick` 免）：

**关 1 · 覆盖度检查表（防偏科）**——饱和只说明"当前角度挖到底了"，不说明"别的角度挖过没有"：

1. 把用户问题拆成 3-6 个子维度（取自层 0 brief 或层 2 拆解结果；例："X 行业前景" → 供给/需求/政策/竞争/风险）
2. 逐维度对照证据板：每个子维度有 ≥1 行证据吗？
3. 有维度 0 行 → 对该维度补搜 1 轮；仍 0 行 → 报告中显式声明「XX 维度未覆盖（无相关公开信息/渠道不可达）」，禁止静默遗漏
4. **信源类别覆盖核对（v3.1.0 新增，吸收 tanzi「最小够用平台类别组合」的多样性维度）**——五类信源：①官方/政务（gov.cn、监管、交易所）②新闻门户/垂直媒体 ③社区 UGC（知乎/雪球/V2EX/HN 等）④学术（openalex/crossref/arxiv）⑤海外独立索引。**deep 档要求 ≥4 类各有 ≥1 行证据**，缺类 → 对该类渠道补搜 1 轮或声明「该类不可达/无相关证据」。防的是渠道数配额下的"假交叉"：12 个渠道跑成同质 SERP、≥3 独立信源全出自同一转载池（域不同≠话语体系独立，D.4 转载计数的类别侧补充）
5. 所有维度 covered-or-declared 且类别达标后才能进综合

**关 2 · 引用抽查协议（防张冠李戴）**——证据板"有来源列" ≠ "来源真的支持该事实"：

- L1 来源的事实 + 置信度「高」的事实：**全部**重开来源 URL，确认原文真的支持该事实（数字、结论、上下文一致）
- 置信度「中」的事实：抽查 ≥30% 重开核验
- 置信度「低」或抽查不通过：不进结论，确需提及标注「单源/未核验」
- 抽查是打开页面核对，不是新搜索——通常 3-8 个 URL，成本远低于多搜一轮（依据：From Fluent to Verifiable，claim-level auditability）
- **取回状态联动（v3.1.0）**：抽查前先过滤——「仅摘要/失败」行若被报告引用，**必须全部重开取全文核验**（不适用 30% 抽样折扣）；「全文」行按上述 L1/高/中档常规抽查。抽查通过 → 该行取回状态升级为「全文」；不通过 → 按原规则降级标注

**关 3 · 查询日志（可审计 + 经验沉淀）**——报告末尾必附：

```
查询日志
| 轮次 | 查询 | 渠道 | 降级轨迹 | 证据板新增行 |
|------|------|------|---------|------------|
| 1 | ... | bing_intl | L1 直取 | 3 |
| 2 | ... | baidu | L1 blocked→L2 回声(搜狐镜像) | 2 |
| 3 | ... | bing_intl | L1 ok | 0 |
饱和声明：连续 2 轮零新增行，覆盖 N 渠道（档位达标/未达标原因）
```

**降级轨迹列（v3.1.0 新增）**：记「用了第几级 + 前一级为何失败」，格式 `L{n} {结果}｜失败原因简述`。四级的判据对应强制规则 5：`L1 web_fetch 直取` / `L2 重试` / `L3 回声搜索(镜像渠道)` / `L4 浏览器兜底(browser_act)` / `L5 fallback 渠道`。

**为什么加这一列**：渠道状态只能说明"现在能不能用"，降级轨迹说明"**这次为什么没直接拿到**"。deep 档出问题时，两列一对照就能分辨三种根因——① 渠道真挂了（连回声都失败）② 渠道活着但本次 blocked（回声成功）③ 查询构造有问题（全程 L1 拿回但内容跑题）。没有这一列，这三种情况在日志里长得一样。

作用：一眼看出是否偷懒（重复查询、渠道数不足、过早停止、**降级次数异常**）；也是经验沉淀的原料（"ecosia 状态漂移快""baidu 高频触发安全验证"这类教训记在这，跨任务复用）。

### 读取路径

| 场景 | 必读 | 可跳 |
|------|------|------|
| 日常搜索 | ① ② ④ ⑤ + engines.yaml | ⑦ ⑧ |
| 投研任务 | ① ② ④ ⑤ ⑦ + engines.yaml | ⑧ |
| JS页/登录态/验证码任务 | ① ② ④ ⑤.10 F.4.2 + engines.yaml browser_act 条目 | ⑦ ⑧ |
| 新设备接入 / 维护 | ① ⑧ F.4.2 + selftest.py | ⑦ |

### 强制规则（防偷懒，硬约束）

1. **分级绑定**：按 ④ 的渠道数执行，`deep` 必须 ≥12 渠道**或当前出口可达渠道上限**（以层 3 预检为准；凑不满 12 时按上限执行并在结果中注明），禁止少搜。
2. **降级必走**：engines.yaml 中状态非 `ok` 的渠道必须走 `fallback` 字段指向的降级渠道，禁止跳过或直接放弃。
3. **来源标注**：聚合结果必须标注来源渠道 + 置信度，无法标注 = 不合格。
4. **投研必读**：投研场景深度任务必须读 ⑦，缺读 = 流程违规。
5. **重试→回声→浏览器兜底→降级**：因 cookie/session/瞬时网络失败时，重试 1 次（间隔 2s）；仍失败则**回声搜索**——提取被 blocked 页面的标题或关键句，投到其他渠道搜索同一内容的镜像版本（见 ⑤.7）；回声仍失败 → **浏览器兜底**（v3.0.0：browser-act stealth-extract 重放同一 URL，见 ⑤.10；web_fetch 因 JS 渲染丢内容/需登录态/有验证码时也可直接启用）；浏览器兜底仍失败才走 engines.yaml 的 `fallback` 降级渠道。禁止跳级（回声与浏览器兜底视目标站点特性可二选一，但禁止双双跳过直接降级）。
6. **交叉验证**：非共识关键数据 ≥3 个不同信源（分层规则见 D.4）才可作结论；孤证只标注不定论（计数由证据板承载）。
7. **饱和即停**：连续 2 轮搜索证据板**零新增行**（客观口径：表里加不进新行，不是「感觉没新东西」）→ 立即停止搜索转入综合，禁止为凑轮次继续搜。**与规则 1 冲突时规则 7 优先**：饱和先到就停，但必须在报告注明「覆盖 N 个渠道（低于档位要求），饱和即停」，让偷懒与理性停止可区分。与规则 1 互补：规则 1 防少搜（下限），规则 7 防过搜（上限）——深度 ≠ 轮次多，关键是何时停、如何综合（S1-DeepResearch 2026 实证：强模型平均仅 9.7 次工具调用）。

---

## ② 意图路由表

> 渠道组按 engines.yaml `route_groups` 定义，此处只写路由决策逻辑。
> **语言适配**：路由确定渠道组后，投递前按目标渠道 `langs`（支持语言 TOP3）匹配查询词语言——支持就用原文，不支持才翻译（跨语言规则见 ⑤.5 第 5 条）。
> **语义通道不参与常规路由（v2.9.0）**：Exa 语义搜索（`semantic` 组）经 MCP 工具调用，非 web_fetch，不属下表任何路由组——仅在三处按需调用：层 0 侦查语义探路（见 ①）、⑤.8 语义雪球（URL 相似页）、⑤.5 描述性查询重构。不计入层 3 渠道数与 ④ 调用分级。
> **浏览器通道不参与常规路由（v3.0.0）**：`browser_act`（`browser` 组）是 CLI 第二执行范式，非路由渠道——仅两类触发：① 强制规则 5 的降级链第四级（web_fetch 被反爬/JS壳/需登录态/有验证码时兜底重放）；② 渠道条目显式标注 `executor: browser_act` 时按浏览器范式执行。不计入层 3 渠道数与 ④ 调用分级。

| 查询类型 | 路由组（yaml 组名） | 原因 |
|---------|--------|------|
| 技术/编程 | `tech` | 技术文档索引全、GitHub/SO 直达 |
| 学术/研报 | `academic`（`bing_academic` 直连首选 + OpenAlex API；`google_scholar` 仅出墙低频；`baidu_xueshu` 已弃用 ❌403） | 学术索引专精，兜底链按直连可用性排序 |
| 公众号/国内舆情 | `wechat` | `weixin` 是唯一公众号渠道 |
| 实时新闻/价格 | `news`（`google_news_rss` 主力 + `tbs=qdr:w` 时间过滤） | 新闻时效 |
| 社区信号/舆情押注 | `community`（触发时机与信号分级见 D.6） | 真实参与度 > SEO 排序 |
| JS渲染页/登录态/验证码 | **不走路由组 → `browser_act` 执行器**（v3.0.0，经强制规则 5 第四级或渠道条目 `executor` 字段触发） | web_fetch 范式覆盖不了的站点 |
| 默认·中文 | `default_zh` | 全面覆盖 |
| 默认·非中文 | `default_en`（出墙出口可上 `google_hk`；直连出口 google 系返 JS 壳，主力 `bing_intl`；`ecosia` 直连 301 重定向不可用，`yandex` 直连实测可用但不稳定，见 yaml） | 全面覆盖 |
| 受限/直连环境兜底 | `restricted_fallback_zh` / `restricted_fallback_en` | 大陆直连出口下国际渠道大面积不可达（2026-08-13 实测），此组全部直连可用（en 组含 `yandex`/`naver`/`mojeek`/`yahoo_jp`/`marginalia`/`seznam` 补充出口，2026-08-15 实测） |

---

## ③ 渠道清单 → engines.yaml

本章已抽取为机器可读注册表 **`engines.yaml`**（同目录），含 **45 条目**：国内 7 + 国际 15 + 特化端点 11 + 学术集群 6 + 社区信号 4 + **语义通道 1**（`exa`，Exa 神经语义搜索 MCP——**非 web_fetch 渠道**，经 MCP 工具调用，定位：层 0 语义探路 / ⑤.8 语义雪球 / ⑤.5 描述性查询重构，不参与层 3 常规执行）+ **浏览器自动化通道 1**（`browser_act`，BrowserAct CLI——**v3.0.0 新增，非 web_fetch 渠道**，第二执行范式，定位：层 3 降级链第四级兜底 + `executor: browser_act` 渠道的执行器，见 ⑤.10 与 F.4.2）。特化端点 11 含 4 个 **fact 裸接口**（`hko` 中国香港天文台 / `open_meteo` 全球天气 / `frankfurter` + `currency_api` 汇率双渠道，2026-09-05 public-apis 交叉评估入池，`intents: [fact]` 按需直调，不进路由组不计渠道数）。国际 15 含 `yandex`/`naver`/`mojeek`/`yahoo_jp`/`marginalia`/`seznam` 六个直连补充出口（2026-08-15 实测）。

**执行器维度（v3.0.0 新增）**：渠道条目新增可选字段 `executor`——`web_fetch`（默认，可省略）| `browser_act`（浏览器 CLI）。现有 43 个 web_fetch 渠道零改动；`browser_act` 通道与任何标注 `executor: browser_act` 的新渠道按 ⑤.10 浏览器范式执行。

每渠道字段：`id / name / group / executor（v3.0.0，可选）/ url / probe / ok_marker / bad_marker / status_proxy（出墙）/ status_cn_direct（直连，selftest 自动刷新）/ fallback / intents / langs（支持语言 TOP3，支持就用原文，不支持才翻译，见手册 ⑤.5 跨语言适配）/ notes / manual（true = selftest 只探测不写回，见 F.5）`。

**Agent 使用方式**：层 3 执行前 Read engines.yaml → 按 ② 选中的组名取渠道列表 → 过滤 `status_cn_direct: ok`（直连环境）或 `status_proxy: ok`（出墙环境）→ 不可用渠道走 `fallback`。语义通道（`exa`）按需直接经 MCP 工具调用，浏览器通道（`browser_act`）按 ⑤.10 经 CLI 调用，均不走 web_fetch 流程。

**人维护方式**：怀疑状态过期 → 双击 `selftest.bat` → 探活结果自动写回 yaml 并生成 `selftest_report.txt`。

---

## ④ 调用分级

| 模式 | 渠道数 | 层 0 侦查 | 适用场景 |
|:----:|:----:|:----:|---------|
| `quick` | 3（`baidu` + `bing_intl` + `ddg`；`ddg` 不可达 → 换 `so360`，禁止降双引擎——双引擎凑不齐交叉验证最低人数） | 禁用 | 日常事实核查、单点数据确认 |
| `standard` | 6-8（按意图与语言路由） | 领域陌生时 1 次 | 产业动态获取、多源交叉验证 |
| `deep` | 12-16 或可达上限（全量+学术/垂直） | 标配 2-3 次 + brief 落纸 | 深度研报反查、认知校准对立族/想象族搜寻 |

> **语义通道与浏览器通道不计入渠道数（v2.9.0 / v3.0.0）**：`exa` 是 MCP 工具调用、`browser_act` 是 CLI 兜底执行器，均非 web_fetch 渠道，上表渠道数不包含它们；语义探路计入层 0 侦查查询次数，语义雪球计入 ⑤.8 雪球配额，浏览器兜底计入 ⑤.10 配额（单任务 ≤3 次）。

---

## ⑤ 操作符与查询构造（层 2 的武器库）

### 高级操作符

| 操作符 | 示例 | 说明 |
|--------|------|------|
| `site:` | `site:github.com python` | 站内搜索 |
| `filetype:` | `filetype:pdf report` | 指定文件类型 |
| `""` | `"machine learning"` | 精确匹配 |
| `-` | `python -snake` | 排除关键词 |
| `OR` | `cat OR dog` | 或运算 |
| `*` | `machine * algorithms` | 通配符（Google） |
| `()` | `(apple OR microsoft) phones` | 分组（Google） |
| `..` | `laptop $500..$1000` | 数字范围（Google） |
| `inurl:` / `intitle:` | `intitle:"index of" mp3` | URL/标题包含（Google） |
| ~~`cache:`~~ | — | ❌ Google 2024 已下线；替代：`wayback` 渠道（见 yaml） |
| `related:` | `related:github.com` | 相关网站（Google） |

### Google 时间/语言过滤

`tbs=qdr:h/d/w/m/y`（过去 1 小时/天/周/月/年）；`tbs=cdr:1,cd_min:1/1/2024,cd_max:12/31/2024`（自定义）；`hl=zh-CN` 界面语言；`lr=lang_zh-CN` 结果语言；`gl=us` 地理。特殊垂直：`&tbm=isch/nws/vid`（图片/新闻/视频）。

### DuckDuckGo Bangs

搜索引擎 `!g` `!b` `!y` `!sp` `!brave`；编程 `!gh` GitHub / `!so` StackOverflow / `!npm` / `!pypi` / `!mdn` / `!docker`；百科 `!w` Wikipedia / `!imdb`；购物 `!a` Amazon / `!e` eBay / `!ali`；地图 `!m`。

### 其他引擎参数

DDG：`kp=1` 严格安全 / `kl=us-en` 区域 / `ia=web|images|news|videos`。Brave：`tf=pw/pm/py` 时间 / Goggles `$boost,site=github.com`（`source=news` 已收费勿用）。Startpage：`cat=web|images|video|news` / `time=day|week|month|year`。WolframAlpha 适合数学/单位/货币/股票/化学查询（`integrate x^2 dx`、`100 USD to CNY`），需 API 或 `web_url_read`，裸 fetch 无效。

### ⑤.5 查询构造建议（层 2 策略构造的依据）

- **复杂问题先拆解**：将"半导体库存周期到哪了？"拆为 `["半导体 库存水位 2026", "晶圆厂 产能利用率 Q2", "芯片 渠道涨价 新闻"]` 分批调用
- **用 brief 里的行话替换用户原话**：层 0 产出的同义词/行话表优先于用户措辞（"先进封装产能" → "CoWoS 产能"）
- **定向过滤**：优先使用 `site:` / `filetype:` / `tbs=qdr:m` 缩小范围
- **迭代搜索**：首轮结果置信度低时，提取新关键词进行第 2 轮补搜（配合 ① 回退循环）
- **失败后重构（v2.4.0 补）**：首查精度差（跑题结果多）时，不翻页、不换渠道，先**重构查询**——加精确短语引号、换字段限定语法（见 ⑤.6）。实例（2026-08-13 实测）：OpenAlex `search=deep research agent` 召回大量无关文献 → 重构为 `filter=title.search:"deep research"` 即干净。
- **描述性查询（v2.9.0 新增）**：关键词搜不到/重构后仍跑题时，把查询改写为"给同事描述理想页面"的语义语句，投语义通道（Exa `web_search_exa`，免费层可用）——例："半导体库存周期到哪了？" → `recent analysis of semiconductor inventory cycle and capacity utilization`。关键词引擎搜不到不代表内容不存在，可能只是措辞不同；语义匹配按意义命中。也可反向用：从语义结果提取实体/行话后，再构造精确查询投关键词引擎。
- **跨语言适配（v2.4.0 补）**：**先匹配渠道支持语言，支持就用原文，不支持才翻译**——按引擎 `langs` 字段（TOP3）判断：查询词语言在 `langs` 内 → **原词直投**（yandex 支持中文就直接用中文，不用翻译成俄语）；不在 `langs` 内 → 翻译成 `langs[0]`（首选语言）再投（中文查韩系 → 译韩语投 `naver`，韩文不在 naver 的 langs 支持里？在——naver langs 是 [ko, en] 不含 zh，所以中文要译成韩语或英文）。`any` 渠道（openalex/crossref/arxiv 等学术 API）原词直投。翻译用 Agent 自身能力 + 层 0 brief 行话表，不依赖外部翻译 API（保持零 Key）。

### ⑤.6 API 型渠道查询语法（v2.4.0 新增）

搜索引擎操作符（`site:`/`filetype:`）只对 SERP 型渠道生效；API 型渠道是另一套体系，且默认常是模糊匹配——不学语法，召回全是噪音：

| 渠道 | 模糊用法（噪音大） | 精确用法 |
|------|----------------|---------|
| OpenAlex | `?search=deep research agent`（宽松匹配，混入无关文献） | `?filter=title.search:"deep research",from_publication_date:2025-06-01&sort=cited_by_count:desc`（标题精确短语 + 时间窗 + 被引排序） |
| GitHub Search | `?q=deep research`（全文模糊） | `?q=deep-research+in:name,description+created:>2025-09-01&sort=stars`（`in:name/description` 限定 + 创建时间 + star 排序） |
| HN Algolia | `?query=xxx`（帖子/评论混杂） | `?query=xxx&tags=story&numericFilters=points>50`（`tags` 过滤帖子/评论，`numericFilters` 过滤分数/时间） |
| Crossref | `?query=xxx` | `?query.bibliographic=xxx&filter=from-pub-date:2025`（bibliographic 字段 + 时间过滤） |

> 实测教训（2026-08-13）：OpenAlex `search=` 宽松匹配曾召回松根真菌、心肌炎指南等完全无关文献，换 `title.search:` 精确短语才干净。**API 渠道首查后先看头几条结果的相关性，跑题先改查询语法，别急着翻页或换渠道。**

### ⑤.7 回声搜索（Echo Search）降级策略（v2.4.1 新增）

**触发时机**：层 3 执行中，目标渠道因反爬（blocked/JS 壳/验证码）重试后仍无法获取全文，但搜索结果页已暴露文章标题或关键句。

**核心思路**：同一内容往往在多个平台分发——头条被反爬挡住，搜狐号/百家号/新浪可能转载了同一篇；百度/必应/Yandex 的索引可能已缓存了全文。用标题作为精确查询词投到其他渠道，找到同一内容的可读镜像。

**查询构造**：
```
// 精确标题 + 定向内容分发平台
"文章完整标题" site:sohu.com OR site:163.com OR site:baijiahao.baidu.com OR site:sina.com.cn

// 精确标题 + 通用搜索引擎（不限制平台）
"文章完整标题"

// 标题过长时，取最具辨识度的 10-15 字片段
"标题中最具辨识度的关键短语"
```

**回声渠道优先级**：

| 优先级 | 渠道 | 原因 |
|:---:|------|------|
| 1 | `baidu` | 中文索引最全，对头条/百家号/搜狐号等内容分发平台覆盖最好 |
| 2 | `bing_cn` | 补充百度未收录的镜像，对新浪/网易等覆盖好 |
| 3 | `yandex` | 独立索引体系，对国内内容平台有额外覆盖，直连相对稳定 |
| 4 | `so360` / `sogou` | 兜底补充 |

**降级体系四级递进（v3.0.0 扩展）**：

```
渠道 blocked → 重试 1 次（间隔 2s）→ 仍 blocked → 回声搜索（1-2 渠道）→ 回声失败
  → 浏览器兜底（browser-act stealth-extract 重放同一 URL，v3.0.0 新增，见 ⑤.10）
  → 浏览器兜底仍失败 → fallback 降级
```

**适用边界与限制**：

| 场景 | 回声有效？ | 原因 |
|------|:---:|------|
| 多平台分发的文章（头条号/百家号/搜狐号互转） | ✅ 高 | 同一内容多平台镜像，标题精确匹配即可命中 |
| 新闻机构原创文章（澎湃/财新/36氪） | ✅ 高 | 新闻稿被大量转载 |
| 平台独家原创（仅在该平台发布） | ⚠️ 低 | 无镜像可找，但搜索引擎摘要仍可能提取关键信息 |
| 微头条/短内容 | ❌ 无 | 不被外部搜索引擎索引 |
| 刚发布 < 2 小时的内容 | ⚠️ 低 | 其他引擎可能尚未索引，回声有时间延迟 |

> **v3.0.0 边界补**：独家原创（⚠️ 低）与平台内内容（❌ 无）两类场景，正是浏览器兜底的主场——回声找不到镜像时，`browser_act` 直接进原站抓。

**成本门控**：回声搜索计入当轮渠道数（不额外增加渠道配额）；每次回声限 1-2 个渠道，禁止全渠道回声（防止成本失控）。回声搜索命中镜像后，镜像来源标注为 `echo:{原渠道}→{镜像渠道}`，信源层级按 D.4 判定（转载不等于原始信源）。

### ⑤.8 滚雪球搜索（Snowball / Citation Chaining）（v2.5.0 新增）

**触发时机**：层 3 执行中，找到一篇 L1/L2 信源的高价值深度文章（长文/研报/政策原文），且文章内含外链、引用列表或作者署名。

**核心思路**：来自学术检索的"滚雪球法"——从一篇已知文献出发，**向后追**（它引用了谁→原始数据源）和**向前追**（谁引用了它→后续验证/反驳）。在 Web 场景里：提取文中的外链找到原始数据、提取作者名追踪同作者其他文章、用文章标题在学术 API 追引用链。这是**纵向发现**（沿引用链扩展新信息），与证据板的**横向验证**（同一事实多源交叉）互补。

**四种雪球路径**（v2.9.0 增补路径 C）：

| 路径 | 操作 | 查询构造 |
|------|------|---------|
| **向后追（数据溯源）** | 提取文章中的外链/引用，打开原始数据源 | 直接访问文内 URL；或提取被引作者+年份投 `openalex`/`bing_academic` |
| **向前追（引用追踪）** | 找哪些后续文章引用了这篇 | `"文章标题"` + `site:scholar.google.com`（出墙）或 `"文章标题"` + `site:openalex.org` |
| **作者追踪** | 找同一作者/研究组的其他文章 | `"作者名"` + `site:平台.com` 或 `"作者名"` + `openalex` API `?filter=author.search:"xxx"` |
| **路径 C：语义雪球（v2.9.0 新增）** | 把高价值文章 **URL 本身**投语义通道，找**内容同构但无引用关系**的页面（不同媒体对同一事件、不同语种的同一研究、措辞不同的同主题深文） | `web_search_exa {"query": "{文章URL}", "numResults": 5-10}`——免费层可用，Exa 语义搜索原生支持 URL 相似查询，按语义相似度返回，不依赖引用/外链关系（advanced 定制需 Key） |

**执行规则**：
- 每篇高价值文章限 **1 轮雪球扩展**（追一层），不递归展开（防止搜索爆炸）
- 雪球搜索结果写入证据板，来源标注：引用雪球为 `snowball:{原文章URL}→{新发现URL}`；语义雪球为 `snowball-sem:{原文章URL}→{新URL}`
- 雪球发现的新事实同样受强制规则 6 交叉验证约束（雪球来源不等于独立信源；**语义相似 ≠ 同一信源**——语义雪球命中的页面即使内容同构，信源层级仍按 D.4 独立判定，交叉验证计数不变）

**成本门控**：`deep` 档每轮搜索最多触发 3 次雪球（选最有价值的 3 篇文章展开，引用雪球与语义雪球合计）；`standard` 档限 1 次；`quick` 档禁用雪球。

### ⑤.9 实体追踪（Entity Tracing）（v2.5.0 新增）

**触发时机**：层 2 策略构造阶段，`deep` 档投研任务中识别出**关键实体**（核心人物/机构/公司/概念），需要追踪其在不同平台的踪迹。

**核心思路**：从"查询驱动"（用户给关键词→搜）切换到"实体驱动"（锁定一个实体→跨平台追踪它的所有踪迹）。来自 OSINT 的实体关系映射方法——不是搜"话题"，而是搜"谁/哪里/关联什么"。

**实体追踪表（层 2 产出物之一）**：

```
实体追踪表（deep 档专用）
├─ 实体1：{人名/机构名}
│  ├─ 目标平台：site:xueqiu.com / site:zhihu.com / site:gov.cn
│  ├─ 查询构造："{实体名}" site:{平台}
│  └─ 追踪目标：发言/任职/关联公司/政策提及
├─ 实体2：{概念/技术名}
│  ├─ 目标平台：学术 API + 行业垂直
│  └─ 追踪目标：产业链不同环节的提及情况
└─ ...
```

**查询构造**：
```
// 人物跨平台追踪
"{人名}" site:xueqiu.com OR site:zhihu.com OR site:36kr.com

// 机构政策追踪
"{机构名}" site:gov.cn OR filetype:pdf

// 概念产业链追踪
"{概念名}" ("上游" OR "中游" OR "下游" OR "供应链")
```

**与现有机制的关系**：实体追踪不是独立搜索轮次，而是**在层 2 构造查询词时，额外为每个关键实体生成 2-3 条定向查询**，并入层 3 的正常渠道执行流程。实体追踪产生的查询计入当轮渠道数。

**成本门控**：`deep` 档实体数上限 5 个，每实体限 2-3 渠道；`standard`/`quick` 档不启用实体追踪（但可在层 0 brief 中记录关键实体供后续参考）。

> **未来增强方向（待实证）**：**多跳推理链（Multi-hop Retrieval）**——A 源说 X→Y，B 源说 Y→Z，因此推出 X→Z。当前证据板的综合阶段已隐式执行此操作，但未显式标注跳数关系。若未来引入，可在证据板增加"推理链"列（标注事实间的隐含推导路径），增强可审计性。代价是增加证据板复杂度，需先评估实战收益是否值得。

### ⑤.10 浏览器兜底（BrowserAct 第二执行范式）（v3.0.0 新增）

**定位**：`browser_act`（engines.yaml `browser` 组，CLI 通道）是 web_fetch 之外的第二执行范式。web_fetch 覆盖不了的四大死区，全部由它兜底：

| 死区 | web_fetch 现状 | browser-act 解法 |
|------|---------------|----------------|
| ① JS 渲染页 | DOM/img 丢失（实测景区官网与政务 CMS 的 img src 全丢） | 真浏览器渲染后 `get html` 全量 DOM；`eval` 页面上下文直接提取结构化数据 |
| ② 登录态 | ❌ 无法携带 cookie/session | 浏览器会话保持登录态；多浏览器账号隔离（`browser create`） |
| ③ 验证码 | ❌ 遇到即 blocked | `solve-captcha` 自动解；强验证码 `remote-assist` 人工接管（生成远程控制链接） |
| ④ 交互操作 | ❌ 只能 GET | 点击/输入/翻页/滚动（state→interact 循环）；无限滚动页 `scroll` |

**第五能力（数据层）**：`network requests --filter` 直接捕获 XHR/fetch JSON——很多站点的数据根本不在 HTML 里而在接口里，抓接口比解析 DOM 更可靠。

**触发条件**（强制规则 5 第四级，或渠道条目 `executor: browser_act`）：
1. web_fetch 被 blocked/返 JS 壳，重试 + 回声搜索均失败 → `stealth-extract <url> --content-type html` 重放
2. 目标内容需要登录态/交互/验证码 → 走完整 open→state→interact→extract→close 循环
3. 目标数据在 XHR 接口里 → `network requests` 捕获

**操作速查**（完整命令见 browse-act skill）：

```bash
# 一次性提取（最简路径，自动反爬+JS渲染+指纹伪装）
browser-act stealth-extract <url> --content-type html

# 完整循环（需交互/登录态时）
browser-act --session s1 browser open <id> <url>
browser-act --session s1 state          # 可交互元素索引
browser-act --session s1 network requests --filter api.example.com   # XHR 捕获
browser-act --session s1 eval "JSON.stringify(Array.from(document.images).map(i=>i.src))"
browser-act session close s1            # 用后必关
```

**成本门控（硬约束）**：
- **单任务浏览器调用 ≤3 次**（云服务免费配额有限，单次 10-30s 延迟）；用超需向用户说明
- **`quick` 档禁用**；`standard`/`deep` 档仅在强制规则 5 第四级触发，web_fetch 通的情况下禁止动用
- 用后必 `session close`；创建/删除浏览器、导入 Profile、修改代理等敏感操作需用户显式确认
- 登录态涉及用户真实账号：**涉隐私站点的登录操作必须先征得用户显式授权**（见 F.6）

**证据板标注**：浏览器兜底取回的内容，来源标注 `browser:{原渠道}→{URL}`；信源层级按 D.4 独立判定（浏览器只是取回手段，不改变信源性质；截图可作引用抽查的视觉凭证）。

**实测战果（2026-09-19 打通当日）**：① 雁荡山官网（wzyds.com）——WebFetch 拿不到任何 img src，stealth-extract 全量提取（3 张带署名作品 + 5 张官方图）；② 温州文广旅局政务 CMS——两篇文章 63 张正文图直链全量提取（`site/picture/-1/` 路径），批量下载后目检精选 13 张嵌入公众号文章。

### ⑤.11 运行形态速查表（v3.1.0 新增）

**为什么需要**：同样的"渠道不可用"，不同运行形态的失败原因和排查入口完全不同——MCP 超时、CLI 起不来、HTTP 403，压根不是一类问题。原排查表散在 F.4.1/F.4.2/F.5 三处，用的时候要跳着查。本表一次列全。

| 运行形态 | 渠道 | 典型失败症状 | 第一排查动作 | 详细排查 |
|---------|------|------------|------------|---------|
| **HTTP 直连**（43 条 web_fetch） | cn 7 / intl 15 / endpoint 11 / academic 6 / community 4 | `timeout` 超时 | 看 engines.yaml `status_cn_direct` 列 | F.5 |
| | | `shell` 通但 0 结果（JS 壳/反爬壳） | 看 ok_marker 是否改版（站改版了）→ 改 marker 或走 L4 | F.5 |
| | | `blocked` 403 | 该渠道进 restricted 兜底组 | F.5 |
| **MCP 工具** | `exa` | 工具列表里没有它 | 没配 mcp.json 或没 Trust | F.4.1 |
| | | HTTP 403（脚本直调） | python-urllib 默认 UA 被 Cloudflare 拦，加浏览器 UA | F.4.1 |
| | | 405 | 用 GET 访问 MCP 端点（正常现象，非故障） | F.4.1 |
| **CLI 代理**（executor 字段） | `browser_act` | 配额耗尽 | 单任务调用已用满 3 次，报告声明 | ⑤.10 成本门控 |
| | | 会话未关导致后续失败 | 检查有无遗留 session，`session close` | F.4.2 |
| | | 起浏览器失败 | 代理/网络环境 | F.4.2 |
| **人工维护**（manual: true） | `baidu` / `bing_academic` | selftest 报的状态与实测不符 | **别信探针结果**——这些渠道对 urllib 裸客户端返反爬页但 web_fetch 可用，以实测为准 | F.5 注意段 |
| | | 频率过快触发风控 | 请求间隔 >2s | 该渠道 notes 字段 |

**集中度体检（v3.1.0 随 selftest.py 提供）**：探活只回答"这个渠道活着吗"，不回答"**它挂了谁跟着断**"。selftest.py 每次探活后自动输出四段体检（集中度枢纽 / 单点故障 / 二级单点 / 薄弱路由组），写入 `selftest_report.txt`。

2026-10-09 首跑结论（45 渠道）：
- **双枢纽**：`bing_intl` 被 10 个渠道当 fallback、`baidu` 被 8 个依赖——任一挂掉将同时断多条降级路径，扩渠道时优先给它们配反向 fallback
- **单点 7 个**：`weixin`（唯一公众号渠道，无降级属设计）、`exa`/`browser_act`（增强通道非必需）、`github_search`/`polymarket`/`wolfram`/`wayback`
- **二级单点**：`hackernews → github_search`，而后者自身无降级 → **社区链路实际只有一层**，HackerNews 挂了直接断
- **薄弱路由组**：`semantic` 组仅 `exa` 一条（设计如此，语义通道是增强不是必需）

> 维护纪律：**新增渠道时必答一句「它断了还剩什么」**——写不出 fallback 的，要么补一条，要么在 notes 里写明"单点可接受"及理由。

---

## ⑥ 快速示例

```javascript
// 渠道 id 与 URL 以 engines.yaml 为准；直连环境英文主力 bing_intl（google 系裸 fetch 返 JS 壳）
web_fetch({"url": "https://cn.bing.com/search?q=python+tutorial&ensearch=1"})

// 站内 / 文件类型 / 时间过滤
web_fetch({"url": "https://cn.bing.com/search?q=site:github.com+react&ensearch=1"})
web_fetch({"url": "https://www.baidu.com/s?wd=机器学习+filetype:pdf"})
web_fetch({"url": "https://www.google.com.hk/search?q=ai+news&tbs=qdr:w"})  // 仅出墙出口

// 学术 API（直连可用）
web_fetch({"url": "https://api.openalex.org/works?search=transformer&per-page=5"})

// 新闻 RSS（直连可用）
web_fetch({"url": "https://news.google.com/rss/search?q=半导体&hl=zh-CN&gl=CN&ceid=CN:zh-Hans"})

// 回声搜索（⑤.7）：头条被反爬 → 提取标题投百度找镜像
web_fetch({"url": "https://www.baidu.com/s?wd=%22半导体库存周期拐点确认+CoWoS产能扩张%22+site:sohu.com+OR+site:163.com"})
// 回声搜索：标题投 Yandex 找独立索引覆盖
web_fetch({"url": "https://yandex.com/search/?text=%22半导体库存周期拐点确认+CoWoS产能扩张%22"})

// 语义探路（层 0，v2.9.0）：用户原话直投语义通道，MCP 工具调用（非 web_fetch）；免费层参数仅 query/numResults/type/livecrawl/contextMaxCharacters
web_search_exa({"query": "半导体库存周期到哪了 产能利用率 先进封装", "numResults": 5, "contextMaxCharacters": 2000})

// 语义雪球（⑤.8 路径 C，v2.9.0）：URL 作 query 找内容同构无引用关系的页面（免费层可用，原生支持 URL 相似查询）
web_search_exa({"query": "https://trendforce.com/xxx/2026/semiconductor-inventory.html", "numResults": 8})

// 结构化提取（证据板预制草稿，v2.9.0 可选，需 API Key 解锁 advanced）：summary + JSON schema 抽取量纲数据
web_search_advanced_exa({"query": "CoWoS 月产能 2026 扩产进度", "numResults": 5, "contents": {"summary": {"query": "产能数字、时间、来源机构", "schema": {"type": "object", "properties": {"fact": {"type": "string"}, "metric": {"type": "string"}, "value": {"type": "string"}, "date": {"type": "string"}}}}}})

// 浏览器兜底（⑤.10，v3.0.0）：web_fetch 被 JS 壳/反爬挡住 → CLI 重放同一 URL（Bash 执行，非 MCP）
// browser-act stealth-extract "https://wzyds.com/" --content-type html
// 登录态/交互场景走完整循环（见 ⑤.10 操作速查）
```

---

## ⑦ 投研方法论 Playbook（Panorama 专用 · 日常搜索跳过，投研任务必读）

> **阅读范围声明**：本段专为 KuMoo Panorama 方法论设计，仅投研任务（产业穿透/认知校准/政策溯源）时阅读。
>
> **编号说明**：文中 A.1.4 / A.1.5 等 A.x 编号指 **Panorama 主方法论文档**的章节，非本手册章节。

### D.1 产业穿透层：高频/另类数据

**触发时机**：穿透层需要验证库存/订单/排产/渠道涨价，且结构化数据源无覆盖时。

**Query 模板**：
```
{行业/产品名} ("开工率" OR "库存水位" OR "排产" OR "现货价" OR "渠道" OR "提货")
```

**时间过滤**：`tbs=qdr:w` 或 `tbs=qdr:m`

**定向信源**：

| 行业 | 优先站点 |
|------|---------|
| 大宗/化工/制造 | `site:mysteel.com`（我的钢铁）、`site:sci99.com`（隆众）、`site:baiinfo.com`（百川盈孚） |
| 电子/半导体 | `site:jiweinet.com`（集微网）、`site:trendforce.com` |
| 消费/渠道 | `site:smzdm.com`（终端折扣深度）、论坛/贴吧（经销商吐槽） |

### D.2 认知校准·对立族：反向观点挖掘

**触发时机**：认知校准进入【对立族】，需要打破确认偏误，寻找看空逻辑、潜在风险和分歧叙事。

**核心原则**：查询词不硬编码——从当前论点取反构造，而非预设关键词穷举。

**工作流（3步）**：

**Step 1 — 提取论点与关键假设**：从路由声明·对立族输出中提取当前核心论点与支撑假设（例：①居民收入预期改善 ②消费意愿回升 ③政策持续加码）。

**Step 2 — 构造逆向查询**：对每个关键假设取反/寻找证伪证据：

| 假设 | 逆向查询构造 |
|------|------------|
| 居民收入预期改善 | `{标的/行业} "收入" ("下降" OR "不及预期" OR "萎缩" OR "承压")` |
| 消费意愿回升 | `{标的/行业} "消费" ("降级" OR "萎缩" OR "储蓄" OR "观望")` |
| 政策持续加码 | `{政策名} ("退出" OR "收紧" OR "力度减弱" OR "效果不及预期")` |

> 查询词随论点变化，上表仅为示例，实际由 Step 1 的假设动态生成。

**Step 3 — 定向信源投放**：

| 信源类型 | 投放策略 |
|---------|----------|
| 深度长文 | `site:zhihu.com/question/`、`site:xueqiu.com` + 查询 |
| 独立研究/研报风险提示 | `filetype:pdf` + `{标的}` + `风险` |
| 海外质疑视角 | `{标的英文名}` + 查询（英文） |

> 噪音词排除由 D.4 信噪比控制统一处理，不在此处硬编码。

### D.3 防火墙/验真层：政策溯源 + 时间考古（v2.5.0 增强）

**触发时机**：评估政策冲击（Panorama 主文档 A.1.5）或定位事件阶段（A.1.4）时，区分"情绪性冲击"与"实质性影响"。

```
# 找原文
{政策名/会议名} site:gov.cn OR filetype:pdf
# 找历史复盘
{历史相似事件} 复盘 影响 股市
# 找代价承担者
{政策名} ("冲击" OR "代价" OR "阵痛" OR "洗牌")
```

#### D.3.1 时间考古（Temporal Archaeology）（v2.5.0 新增）

**触发时机**：搜索结果暗示内容近期被修改/删除（404、内容突然变短、关键段落消失、发布时间被更新），或需要追踪政策/企业表述的**历史演变**。

**核心思路**：`wayback` 渠道不只是"死链还原"工具——它是一种**带时间维度的搜索策略**。主动搜索内容的历史版本来追踪变化：政策文件修改前后对比、企业官网战略表述演变、被删除的报道还原、数据/价格的时序变化。

**执行流程**：
```
当前版本搜索 → 发现修改/删除信号 → wayback 追溯历史版本 → 对比差异 → 提取关键变化
```

**查询构造**：
```
// Wayback 直接查询（wayback 渠道）
web_fetch({"url": "https://archive.org/wayback/available?url={目标URL}"})

// 被删除内容的还原
{文章标题} site:web.archive.org
{文章标题} "cache" OR "存档" OR "快照"

// 政策变更追踪
{政策名} ("修订" OR "废止" OR "替代" OR "更新") site:gov.cn
```

**适用场景**：

| 场景 | 操作 |
|------|------|
| 政策文件被修改 | Wayback 对比修改前后版本，提取关键变化段落 |
| 报道被删除/404 | Wayback 还原全文，标注删除时间 |
| 企业战略表述变化 | 对比官网不同时间点的"关于我们"/"投资者关系"页面 |
| 数据/指标修订 | 对比统计局/央行不同发布版本的同一数据 |

**成本门控**：每次时间考古限 1-2 次 Wayback 查询；仅投研 `deep` 档和 D.3 政策溯源场景启用。

### D.4 投研信噪比控制

按**信源权威性优先于关键词匹配**过滤：

| 层级 | 信源类型 | 处理方式 |
|:---:|---------|---------|
| **L1** | 政府/监管/交易所官方（site:gov.cn、csrc、sse） | 高优先级，直接提取 |
| **L2** | 行业垂直媒体、券商研究所 | 正常读取，提取带量纲段落 |
| **L3** | 财经门户、综合媒体 | 不盲信，交叉验证后引用 |
| **L4** | 匿名自媒体、标题连续感叹号/emoji | 降权，除非被 L1/L2 引用 |

**信号增强**：L1-L2 来源的带量纲/产业链专有名词段落（%、同比、bps、万吨、CoWoS、渗透率等）→ 直接引用；L3 同类段落 → 标注来源进交叉验证池。

**交叉验证**：≥3 个不同层级信源同时指向同一非共识数据 → 高置信度写入报告；单一信源无论多权威 → 只标注不作孤证结论。

### D.5 认知校准·想象族：替代叙事构建

**触发时机**：认知校准进入【想象族】，寻找能解释同样数据的替代叙事。

**核心区别**：D.2 = "我可能错在哪"（证伪）；D.5 = "同样的数据能否讲另一个故事"（替代解释）。

**工作流（3步）**：

**Step 1** — 提取当前叙事的核心数据与逻辑链（例：叙事"消费复苏政策驱动"，数据点"社零同比+5%、以旧换新带动家电"）。

**Step 2** — 围绕同样的数据点搜索替代解读框架：

| 搜索方向 | 查询构造思路 |
|---------|------------|
| 历史类比 | 数据特征 + "类似" + 历史时期（"社零回升 类似 2009 补贴退出后"） |
| 跨框架解读 | 同一数据 + 不同框架（"社零+5%" + "资产负债表" OR "预防性储蓄" OR "结构性分化"） |
| 结构性替代叙事 | 当前共识 + 结构性挑战（"消费复苏" + "人口" OR "杠杆" OR "收入分配"） |

**Step 3** — 定向信源投放：深度分析（知乎问题页/雪球）、海外视角（FT/WSJ/Economist 英文查询）、历史复盘（查询 + "复盘" OR "历史对比"）。

### D.6 社区信号源：参与度优先的另类搜索

**触发时机**：需要了解"真实用户在讨论什么、在押注什么"——舆情研判、竞品动态、市场情绪。

**核心思路**：真实参与度（upvote/点赞/真金白银）比 SEO 排序更可信。渠道清单与直连状态见 engines.yaml `community` 组（`polymarket` / `hackernews` / `github_search` / `reddit`）。

**信号分级**：Polymarket 真金白银 > HN/Reddit 高票 > 社媒点赞，按可靠性递减采信。

**实现说明**：Reddit 直连与脚本均被 block，用 `site:reddit.com` 经 `bing_intl` 定向抓取（`ecosia` 直连 301 重定向不可用）；有 last30days 类 skill 时可自动聚合（`mvanhorn/last30days-skill`，MIT）。

---

## ⑧ 参考区（仅新接入/维护时读）

> 含 SearXNG 部署、答案层、MCP 接入、维护流程、合规与版本历史。日常搜索与投研任务无需阅读。

### F.1 SearXNG 零 Key 元搜索（与渠道清单互补，非替代）

**定位**：SearXNG 是元搜索快速层——一个调用覆盖 70+ 引擎拿结构化 JSON；engines.yaml 清单适合深度定制（`site:`/`filetype:` 定向、单渠道行为可预期）。需要快、泛、结构化 → SearXNG；需要深、精、可控 → 渠道清单。

| 项目 | 内容 |
|---|---|
| 公共实例 | `searx.be`、`searx.tiekoetter.com`（实时列表 `searx.space`）——**仅浏览器手动兜底** |
| API 格式（仅自建） | `GET {实例}/search?q={query}&format=json&language=zh&categories=news&time_range=day` |
| 自托管（推荐） | `docker run -d -p 8080:8080 searxng/searxng:latest` |

> **⚠️ 关键实测（2026-08）**：公共实例普遍禁用 JSON API（priv.au 403、searx.be Cloudflare JS 验证）。脚本/MCP 调 JSON 必须自建实例且开启：
> ```yaml
> server:
>   limiter: false        # Agent/脚本调用必关，否则 botdetection 403
>   public_instance: false
> search:
>   formats: [html, json]  # 默认只开 html
> ```
> 自建实例需部署在能出墙的 VPS 上（否则拉不到 Google/Bing 上游）。

**接入**：原生 JSON API 或 MCP 桥 `ihor-sokoliuk/mcp-searxng`（MIT），`SEARXNG_URL` 指向实例。

### F.2 学术使用建议

端点清单与状态见 engines.yaml `academic` 组。研报/论文溯源流程：OpenAlex + Crossref 定位文献与 DOI → 视领域进 PubMed（医）/ arXiv（理工）→ Unpaywall 找开放全文。直连环境学术首选 `openalex` + `bing_academic`。

需要语义近义召回（关键词漏掉同义表达，如"强化学习"↔"RL"）时叠加中科大 `ustc-ai4science/academic-search`（MIT）：SQLite 300 万论文 + FTS5 + Qdrant + BGE-M3 + RRF 混合排序；**Claude Code Skill 形态非 MCP Server**，`git clone` 到 `~/.claude/skills/` + `check-deps.sh`，依赖 Chrome CDP 自动化与数 GB 索引数据；非 Claude Code 环境需评估兼容性或改用 yaml 学术端点。

### F.3 AI 答案层（检索→合成答案，与搜索层互补）

**定位**：yaml 渠道/SearXNG 是**搜索层**（返回原始结果，Agent 自己读）；答案层输入研究问题，自动多源检索+多轮迭代+带引用合成报告。

**使用决策**：带引用结论报告/跨领域综述 → 答案层；定制搜索词/逐源验证/控制信源（D.4）→ 搜索层。互补用法：答案层产初稿 → 搜索层对关键结论定向复核（D.2/D.3）。

**参考实现**：`LearningCircuit/local-deep-research`（~8.9k stars，2026-08 实测）
- 引擎层全零 Key：arXiv/PubMed/Semantic Scholar/Wikipedia/SearXNG/GitHub/Wayback，与 yaml 端点完全复用
- LLM 层：本地 Ollama（零 Key）或云端（需 Key，非必需）
- **⚠️ 重型依赖（实测）**：190+ 包（torch/sentence-transformers/crawl4ai/langchain/langgraph/playwright/spacy），建议 ≥10GB 磁盘 + 8GB 内存，首装 10-30 分钟起；Windows 沙箱回收站不可用会报 `SAFE_DELETE_FAIL_CLOSED`，需正常用户环境安装
- 接入：`pip install local-deep-research` + `playwright install`；`ldr-web`（127.0.0.1:5000）或 `ldr` CLI

### F.4 跨设备接入 SearXNG MCP（仅新接入时读）

1. Node 20+（WorkBuddy 自带 Node 22 满足）
2. `~/.workbuddy/mcp.json` 的 `mcpServers` 加入：
   ```json
   "searxng": {
     "command": "npx", "args": ["-y", "mcp-searxng"],
     "env": { "SEARXNG_URL": "实例URL" }, "timeout": 60000
   }
   ```
3. 连接器管理页 → `searxng` → **Trust** 启用（MCP 不自动激活）
4. 调 `searxng_web_search` 验证

实例选择：自托管优先（须开 JSON 端点，见 F.1；从大陆访问须部署在出墙 VPS）；公共实例 JSON API 普遍被禁，仅浏览器兜底。暴露工具中 `web_url_read` 可把任意 URL 读成 Markdown，补 JS 重页面盲区，是最大增量价值。排查：`ECONNREFUSED`/`403` → 实例不可达或 JSON 未开；`Network Error: aborted` → 确认 `timeout: 60000`。

### F.4.1 Exa 语义搜索 MCP 首次接入与排查（v2.9.0 新增）

**定位**：Exa 是神经语义搜索 MCP（engines.yaml `semantic` 组），与 SearXNG 同属 MCP 接入体系。免费层无 Key 可用基础工具；注册 API Key（dashboard.exa.ai/api-keys）解锁 advanced/deep 工具。

**接入三步**：

1. `~/.workbuddy/mcp.json` 的 `mcpServers` 加入（保留已有 server 不动）：
   ```json
   "exa": {
     "url": "https://mcp.exa.ai/mcp",
     "timeout": 60000
   }
   ```
   解锁全部工具时 `url` 追加 `?exaApiKey=你的Key`。
2. 连接器管理页 → 自定义连接入口 → `exa` → **Trust** 启用（MCP 不自动激活，Trust 后需重开会话加载工具）。
3. 验证：
   - 命令行握手（返回 `exa-search-server` 即通）：
     ```bash
     curl -s -X POST "https://mcp.exa.ai/mcp" -H "Content-Type: application/json" \
       -H "Accept: application/json, text/event-stream" \
       -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"probe","version":"1.0"}}}'
     ```
   - 会话内冒烟：调 `web_search_exa` 一条语义查询，看是否返回标题/URL/Highlights。

**免费层工具边界（2026-08-29 实测）**：仅 `web_search_exa`（语义搜索：query/numResults/type/livecrawl/contextMaxCharacters，返回标题/URL/发布日/Highlights）+ `web_fetch_exa`（URL 抓取，参数 `urls` 数组）。`web_search_advanced_exa`（域过滤/时间/category/contents.summary+JSON schema）/ `deep_search_exa` 需 API Key。语义雪球免费层用 `web_search_exa` 的 query 传 URL 近似替代（Exa 语义搜索原生支持 URL 相似查询）。

**JS 重页面盲区由 web_fetch_exa 承担（2026-09-05 实测，优先级声明）**：`web_fetch_exa` 是服务端渲染抓取——实测掘金 SPA 首页（纯 JS 渲染）返回 3029 字符真实文章内容，裸 web_fetch 拿空壳的页面它能读。**这意味着 F.1/F.4 的"SearXNG 自建"不再是补 JS 盲区的必需路径**：web_fetch + web_fetch_exa 双层已覆盖（静态页走 web_fetch，JS 页走 web_fetch_exa）；SearXNG 自建降为可选——仅当需要"一个调用覆盖 70+ 引擎"的元搜索本身时才值得投入（F.1 的 VPS/docker 条件不变）。另：python-urllib 默认 UA 会被 Exa 的 Cloudflare 403，裸脚本调 MCP 端点必须带浏览器 UA。

> **v3.0.0 分工修订**：web_fetch_exa 覆盖"JS 渲染只读页"；**登录态/验证码/交互/反爬强站四类场景 web_fetch_exa 无能为力，由 `browser_act`（⑤.10）接管**——两层互补不重叠。

**排查表**：

| 现象 | 原因 | 处理 |
|------|------|------|
| 工具列表无 `web_search_exa` | 没配置 或 没 Trust | 补 mcp.json 条目 → Trust → 重开会话 |
| 初始化超时 / ECONNREFUSED | 端点不可达（出口问题） | 换出口重试（大陆直连实测可用 ~2s）；`timeout` 调到 60000 |
| HTTP 405 | 用 GET 访问 MCP 端点 | 正常现象——MCP 必须 POST + JSON-RPC，GET 405 不是故障 |
| **HTTP 403（脚本直调）** | python-urllib/requests 默认 UA 被 Exa 的 Cloudflare 拦 | 请求头带浏览器 UA（实测 2026-09-05：urllib 裸 UA 403 → 浏览器 UA 通过） |
| 限流 / 429 | 免费层配额 | 降频（请求间间隔）；升级 API Key |
| 工具存在但调用报错 | 参数与免费层不匹配（如用了 `contents` 参数） | 免费层只用 query/numResults/type/livecrawl/contextMaxCharacters；web_fetch_exa 的参数是 `urls`（数组）不是 `url` |

**降级兜底**：exa 是增强通道非必需——连不上时层 0 语义探路退回 baidu/bing_intl 泛搜，⑤.8 语义雪球跳过，⑤.5 描述性查询退回关键词重构，主流程零削弱（增强不削弱红线）。

**服务层自愈预检门（2026-09-06 补，不升版）**：Agent 调用 MSE 前应先探测增强通道在位状态（会话内工具列表 / 连接器查询），缺失时按三档处理：
1. **自行建立**——通道须同时满足「零 Key、免账号授权、参数已知、可一键移除」全部条件才允许自建（如 exa：QwenWork 环境经 `qw_action add` 添加自定义连接器 `https://mcp.exa.ai/mcp` 实测 ~2s 自动握手 connected，惰性加载当场可拾取工具，无需重开会话；WorkBuddy 环境 mcp.json 加条目后需重开会话），建立后引擎调用日志记一行「服务层自建：exa」留痕；未拾取则转第 3 档。
2. **提示用户**——需要 API Key / OAuth 账号授权 / 付费的通道（exa advanced 层、企典类）：必须先征得用户确认，不得擅自注册、授权或填 Key。
3. **降级继续**——自建失败、用户未确认、或建立后仍连不上：按降级声明走兜底（层 0 语义探路退回 baidu/bing_intl 泛搜），主流程零阻断（增强不削弱红线）。

### F.4.2 BrowserAct（browser_act 通道）接入与排查（v3.0.0 新增）

**定位**：浏览器自动化 CLI 通道（engines.yaml `browser` 组），MSE 的第二执行范式。**不参与层 3 常规执行与渠道数计数**，仅在强制规则 5 第四级兜底或渠道条目标注 `executor: browser_act` 时调用。

**接入全流程（2026-09-19 实测三坑全记录）**：

1. **安装（坑① Python 版本）**：`browser-act-cli` 强制 `requires-python ==3.12.*`——3.13 的 pip 直接拒装。**禁用官方安装器**（Windows 静默安装 exit 67 不可靠，实测失败）。**成功方案（embed zip 解压法）**：
   ```bash
   # ① 下载 embed 包（华为云镜像，国内可达）+ get-pip
   curl -L -o py312-embed.zip https://mirrors.huaweicloud.com/python/3.12.10/python-3.12.10-embed-amd64.zip
   curl -L -o get-pip.py https://bootstrap.pypa.io/get-pip.py
   # ② 解压到受管目录，改 python312._pth：去掉 "#import site" 的注释并加 "Lib\site-packages"
   # ③ 装 pip 与 CLI（清华镜像）
   python.exe get-pip.py -i https://pypi.tuna.tsinghua.edu.cn/simple
   python.exe -m pip install browser-act-cli -i https://pypi.tuna.tsinghua.edu.cn/simple
   ```
2. **WorkBuddy 权限门（坑② 错误码 230331）**：WorkBuddy 环境内首次调用报 `Error 230331` 时，CLI 协议强制三步顺序——**Step 1 用户在 WorkBuddy 开启 "Allow Full Access"（Agent 必须停下等用户确认，不得代跳）→ Step 2 Agent 跑 `browser-act daemon restart` → Step 3 重试原命令**。附带效果：Full Access 同时解锁沙箱出网（curl 可下载华为云等国内源）。
3. **授权（坑③ 免费注册）**：`browser-act auth login` 生成注册链接（60 分钟有效）→ 用户浏览器打开注册 → `browser-act auth poll` 轮询至 `status=completed`（Key 自动保存，不回显）。
4. **验证**：`browser-act --version`（1.4.2）+ `browser-act get-skills core --skill-version 2.0.2` + 一次 stealth-extract 冒烟。

**排查表**：

| 现象 | 原因 | 处理 |
|------|------|------|
| pip 报 No matching distribution | Python 非 3.12（requires-python ==3.12.*） | 用 F.4.2 第 1 步的 embed zip 方案装 3.12 |
| 静默安装器 exit 67 | Windows 安装器在 Agent shell 下不可靠 | 放弃安装器，走 embed zip 解压 |
| Error 230331 | WorkBuddy 沙箱未开 Full Access | 用户开权限 → daemon restart → 重试原命令（严格三步顺序） |
| auth poll 一直 pending | 用户尚未完成网页注册 | 等待；链接 60 分钟过期，过期重新 auth login |
| daemon 报错/工具不响应 | 后台服务状态异常 | `browser-act daemon restart` |
| 免费配额耗尽 | 云浏览器限次 | 降频；确认任务必要性；必要时升级付费 |

**证据板与合规**：浏览器取回内容来源标注 `browser:{原渠道}→{URL}`，信源层级按 D.4 独立判定；登录态 cookie 仅内存保持；**涉用户隐私账号的登录必须先征得用户显式授权**（F.6）；robots.txt 与目标站 ToS 遵守同 F.6。

### F.5 维护流程（渠道状态刷新）

**触发时机**：搜索大面积失败、换网络环境、距 `meta.last_probe` 超过 1 个月。

**步骤**：双击 `selftest.bat`（或 `python selftest.py`）→ 8 线程探活 43 个 web_fetch 渠道（~1 分钟）→ 状态自动写回 engines.yaml `status_cn_direct` 列 + 更新 `meta.last_probe` → **附「渠道依赖集中度体检」输出（v3.1.0）**（fallback 依赖拓扑 / 单点故障 / 二级单点 / 薄弱路由组，回答"它断了还剩什么"）→ 全部内容写入 `selftest_report.txt`。

> **体检与探活是两种失败视角（v3.1.0）**：探活答"这个渠道活着吗"，体检答"**它挂了谁跟着断**"。两者缺一不可——渠道全绿但依赖拓扑脆弱（双枢纽 bing_intl/baidu、hackernews 二级单点）是探活永远看不见的盲区。体检只读，不写回 yaml。速查表见 ⑤.11。

> **语义通道与浏览器通道不参与探活（v2.9.0 / v3.0.0）**：`exa` 是 MCP 端点、`browser_act` 是 CLI 通道，GET 探活无意义；两者条目 `ok_marker` 留空（selftest 判"无 ok_marker 保持原状态"）+ `manual: true`（只报告不写回），selftest 对 43 个 web_fetch 渠道探活，exa 与 browser_act **均不探活不写回**——状态由人工实测维护（exa：2026-08-29 大陆直连实测可达 → ok；browser_act：2026-09-19 雁荡山官网+温州政务 CMS 双站点实测 → ok）。

**判定逻辑**：HTTP 200 不可信——按 `ok_marker`（真实结果特征串）/`bad_marker`（壳页/反爬特征串）判定：`ok` 真实出结果 / `shell` 通但 0 结果 / `blocked` 403 反爬 / `timeout` 超时 / `geo` 地域封锁。

**解析锚点回归（fixtures，v3.1.0 新增，吸收 tanzi 离线回归设计；机制先立法、`fixturetest.py` 脚本待落地）**：

- **为什么 selftest 不够**：探活只测"渠道活着没"，测不出第三类失败——**渠道活着但解析已坏**（实证：2026-08 bing `site:` 被静默忽略仍返 200 且 selftest 判 ok；so360 `eclk?aurl` base64 格式、OpenAlex/HN API 字段路径都可能悄悄变）。这类失效静默产噪，比渠道死亡更危险。
- **机制**：`fixtures/<渠道>/` 存真实响应节选（HTML/JSON 样例，脱敏去 cookie 与用户参数）+ 期望提取结果（`.expect`）；`fixturetest.py` **离线**断言：ok/bad_marker 命中、aurl 解码、标题/直链正则、API 字段路径。全过 = 解析锚点未漂；任一败 = 该渠道解析模式已坏——**即使 selftest 判 ok 也要先修解析再投用**。
- **触发**：执行中发现解析异常（空表/单位不对/常量分——"指标先审取数源码"纪律的机械化）时定点跑；或每月随 selftest 批量跑一次。
- **沉淀纪律**：每次实测发现漂移并修复解析，当场把一份真实响应存为 fixture——漂移教训变成回归资产，同一坑不掉第二次。
- **首批覆盖优先级**：`so360`（aurl 解码 + SERP 直链正则）、公众号提取脚本、`openalex`/`hackernews` API 字段路径、`bing_cn`/`bing_intl` 的 `site:` 行为验证。

**注意**：`status_proxy`（出墙状态）列 selftest 不覆盖，需出墙出口人工实测维护；标 `manual: true` 的渠道（baidu/bing_academic/exa/browser_act）对 urllib 裸探针返反爬页或不适配，selftest 只报告不写回（exa 为 MCP 通道、browser_act 为 CLI 通道，状态人工实测维护，见 F.4.1/F.4.2）；新增渠道直接在 yaml 加条目（带 probe + ok_marker），selftest 自动纳入。

### F.6 安全与隐私说明

**Cookie**：仅内存中用于会话状态维持（403/429 时），永不写盘，会话结束即清。**爬取伦理**：请求间 1-2s 延迟，遵守 robots.txt，仅用于合法搜索聚合。**数据**：不收集/传输用户个人信息，全程本地执行。**合规**：仅用于个人研究、投资分析与 Agent 能力验证；大规模商业调用请切换官方 SERP API（Serper/Tavily/SearXNG 自托管），使用者自行承担目标引擎 ToS 合规风险。**语义通道（Exa）**：MCP 服务端处理查询，免费层无 Key 基础工具可用；深度功能（deep_search 等）需注册 API Key，按 Exa 服务条款使用。**浏览器通道（BrowserAct，v3.0.0）**：云浏览器服务按免费配额使用；**登录态 = 用户真实账号，涉隐私站点的登录/操作必须先征得用户显式授权，不得擅自登录任何账号**；浏览器会话结束必须 `session close`，登录 cookie 不落盘；UGC 平台（小红书等）内容仅限考证，截图/原图的公开发布遵守对应平台条款与博主版权。

### F.7 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| **v3.1.0** | 2026-10-07 | **吸收 tanzi 跨平台采集 Skill 三项设计（对 tanzi 一手核验后择取；_raw 快照落盘评审通过但缓一档未入）**：① 证据板新增**「取回状态」列**（全文/仅摘要/失败）——"不把摘要写成全文结论"从关 2 审计前移到写入打标，「仅摘要」行禁作带量纲结论；关 2 补联动规则（仅摘要/失败行被引用必全量重开核验，通过即升档全文）；deep 完整六列、standard 可选、quick 不变；② 关 1 新增**信源类别覆盖核对**——五类（官方政务/新闻垂直/社区UGC/学术/海外独立索引）deep 档 ≥4 类各有 ≥1 行，防渠道数配额下"域不同但同转载池"的假交叉（吸收 tanzi"最小够用平台**类别**组合"的多样性维度，补 MSE 只按渠道数配额的盲区）；③ F.5 新增**解析锚点 fixture 回归**——selftest 测"渠道活着"、fixtures 测"解析锚点对"，覆盖"渠道活着但解析已坏"第三类失败（bing site: 静默忽略实证），漂移修复后当场沉淀 fixture；机制先立法、fixturetest.py 待落地；④ 三件套表/一句话/增强不削弱原则「五列」措辞同步「各列」；⑤ **wechat 渠道配套工具收编**——tanzi 公众号采集适配器固化 `tools/wx_adapter/`（MIT，上游 wx-search-cli 0.1.0，唯一依赖 cheerio；2026-10-07 源码审计+端到端实测：search/real_url 解析/正文 read 全链路通，11 项离线回归全过），登记在 engines.yaml `weixin` notes（命令/实测边界/首装 `npm ci --ignore-scripts`），手册正文按"URL/状态/工具只在 yaml 维护"规则不重复。**未吸收并明因**：搜查令概念（层 2 已同构）、全量 _raw 存档（产品复核场景才需要，我们按结论页裁剪后缓一档）、其依赖链（32 star 单人项目 30+ 平台适配，维护脆弱——拿设计不拿依赖）<br>（2026-10-09 补，不升版，吸收 Agent-Reach 结构拆解笔记）：① selftest.py 新增**渠道依赖集中度体检**（探活后自动输出四段：集中度枢纽/单点故障/二级单点/薄弱路由组，只读不写回）——补上探活看不见的依赖拓扑盲区，首跑实证双枢纽 bing_intl（10 依赖）/baidu（8 依赖）、hackernews 二级单点；② 关 3 查询日志新增**「降级轨迹」列**（L1 直取/L2 重试/L3 回声/L4 浏览器/L5 fallback + 前一级失败原因）——区分「渠道挂了/本次 blocked/查询跑题」三种根因；③ 新增 ⑤.11 **运行形态速查表**（HTTP 直连 43 条 / MCP 工具 exa / CLI 代理 browser_act / 人工维护 manual 四形态 × 失败症状 × 排查入口，原三处分散排查表合一）；④ F.5 同步体检说明。**暂缓**：fallback 字段升级为 per-channel 有序候选链（方法来自 Agent-Reach 多级后端链）——v3.0.0 刚立的全局四级链尚未跑够时长，且笔记自身提示「每加一个候选就多维护一种运行形态，不是越多越好」，等看清哪些渠道真需要多级再上 |
| **v3.0.0** | 2026-09-19 | **BrowserAct 浏览器自动化第二执行范式接入（温州实战打通：WebFetch 三大死区实测→CLI 全量补齐）**：① engines.yaml 新增 `browser` 组 + `browser_act` 条目（CLI 通道，`manual: true` 不参与探活，status_cn_direct: ok 2026-09-19 雁荡山官网+温州政务 CMS 双站点实测），总条目 44→45；渠道条目新增可选 `executor` 字段（web_fetch 默认 \| browser_act），现有 43 渠道零改动；② 手册 ① 层 3 与强制规则 5 降级链三级扩四级——重试→回声→**浏览器兜底（stealth-extract 重放）**→fallback（⑤.7 适用边界补：独家原创/平台内内容正是浏览器兜底主场）；③ 新增 **⑤.10 浏览器兜底专节**——四大死区（JS渲染/登录态/验证码/交互）+ 第五能力（network requests 抓 XHR JSON）+ 操作速查 + 成本门控（单任务≤3 次/quick 禁用/session close 纪律/隐私授权）+ 实测战果（63 张政务正文图提取、13 张实嵌公众号文章）；④ ② 路由表新增"JS渲染页/登录态/验证码 → browser_act 执行器"行；⑤ ④ 调用分级备注浏览器通道不计渠道数；⑥ 新增 **F.4.2 接入与排查附录**——安装三坑全记录（Python ==3.12.* 强制→embed zip 解压法/静默安装器 exit 67 不可靠、WorkBuddy 230331 权限门三步协议、auth 注册流程）+ 排查表；⑦ F.4.1 补 v3.0.0 分工修订（web_fetch_exa 管 JS 只读页，browser_act 管登录态/验证码/交互/强反爬，两层互补）；⑧ F.5/F.6 同步浏览器通道探活与合规（登录态隐私授权红线）；⑨ meta.version 2.9.0→3.0.0（engines.yaml 同步） |
| **v2.9.0** | 2026-08-29 | **Exa 语义搜索通道接入（A+B 方案落地，依据 Exa MCP 技能实测能力盘点）**：① engines.yaml 新增 `semantic` 组 + `exa` 条目（MCP 神经语义搜索，`ok_marker` 留空 + `manual: true` → selftest 不误报不写回，保持 untested），meta.version 2.5.0→2.9.0（顺手修复跨区不一致）；② 层 0 侦查新增**语义探路**通道（用户原话直投 Exa，结果标题/摘要暴露行话，绕过"不知道关键词就搜不到"死穴）；③ ⑤.8 滚雪球新增**路径 C 语义雪球**（URL 作 query 找内容同构但无引用关系的页面，补引用链盲区，来源标注 `snowball-sem:`，交叉验证计数不变）；④ ⑤.5 查询构造新增**描述性查询**（关键词搜不到时改写为语义描述投语义通道）；⑤ 证据板可选增强：Exa summary+JSON schema 结构化提取作证据板行预制草稿（独立信源数/置信度仍是 Agent 职责）；⑥ 语义通道不参与层 3 常规执行与 ④ 渠道数计数（MCP 工具调用非 web_fetch），零结构改动；⑦ F.5/F.6 同步语义通道探活与合规说明（同日实测补说明，不升版）：端到端实测 mcp.exa.ai/mcp 大陆直连可达；免费层仅暴露 web_search_exa + web_fetch_exa 两工具且无 Key 可调；新增 F.4.1 接入与排查；2026-09-05 补：engines.yaml 新增 4 个 fact 裸接口（hko/open_meteo/frankfurter/currency_api），总条目 40→44；web_fetch_exa 实测承担 JS 重页面盲区（掘金 SPA 3029 字符真数据），SearXNG 自建降为可选 |
| **v2.8.0** | 2026-08-26 | **加工层默认启用**（用户指令：调用多元搜索即默认调度傀儡模型，免显式指示）：① 加工层由「可选增强 / 触发条件按需启用（不强制）」改为**默认启用**——按档位成本门控执行（deep 全开 / standard 难度前置+去重摘要 / quick 禁用），豁免仅两类：用户显式说"不用傀儡"、短文本（单条 <100 字）；② 原触发条件三情形（≥15 条重复高、超上下文预算、deep 结构化整理）转正为**必执行情形**，不因豁免条款跳过；③ 一句话/标题/定位/增强不削弱原则同步改「默认」措辞，证据板五列/信源层级 D.4/规则 6/三道质检零削弱不变（2026-08-28 补说明，不升版）：仲裁环节改为主模型实质仲裁——傀儡裁判撞 LLM-as-Judge 两大偏差（self-preference 同族溢价 5-25 点；position bias A 位占优 10-15 点），judge 命令降为可选省资源档（跨家族裁判+双序跑）；傀儡池引用 Ver1.6.0→Ver1.8.0。历史日志条目保留当时事实不改 |
| **v2.7.0** | 2026-08-23 | **加工层难度路由 + Judge 仲裁 + 证据板置信度联动**（对齐 PuppetMaster v1.4 / Puppeteer 范式研究简报）：① 新增**难度感知前置路由**——加工前 `route` 评估层 3 结果难度（简单→lite/agnes-2.0 快跑、困难→agnes-2.5+glm 双交叉），依据 Dynamic Model Routing 综述，成本降 60% 质量仅差 1 分；② 新增**Judge 仲裁位**——双模型交叉不一致时先用 `judge`（agnes-2.0 当裁判）判 A/B/一致，仍不分胜负才触发 ⑤.5 交 Agent，仲裁走「层2 judge→层3 主模型」级联，主模型只做最终裁决；③ **证据板置信度列接加工层**——加工层双交叉一致→置信度升一档（低→中、中→高），独立性判断不变（规则 6 计数仍按原始渠道）；成本门控：standard 加难度前置，deep 全开（route+双交叉+judge+打标+摘要）（同日补说明，不升版）：① judge 判「一致」→ 重测一次仍一致才任取；② 触发条件补**短文本不加工**（<100 字直接主模型读）；③ 加工层"为什么需要"补**省 token 量化锚点**（输入密集全类型 total 均值省 64%：classify 94%/tags 92%/route 90%/summarize 82%）；④ 红线补**输出密集型产出一律主模型校验后采纳**（prompt 倒挂 -38%~-115%）；⑤ 成本门控补**批量调用节流纪律**（>5 轮加 sleep+重试+`except BaseException` 捕获 SystemExit）；⑥ 新增**六维增量价值**框架（省token/并行/专注度/防限流/窗口/异构——"注意力解放器+并行手脚，不是省钱工具"）；⑦ 证据板置信度升档**仅适用 classify 类别一致性**，summarize/tags 不触发升档 |
| **v2.6.1** | 2026-08-23 | **子代理加工层扩为 4 傀儡 + 三层兜底**：① 傀儡池从 glm-4-flash/lite 两傀儡扩为 agnes-2.5-flash（主判·推理）/glm-4-flash（交叉验证）/agnes-2.0-flash（打标+摘要）/lite（层2 二次补位）四傀儡，分工明确；② **deep 档分类双模型交叉验证**（agnes-2.5 + glm-4-flash 并行，一致→高置信，不一致→Agent 仲裁触发 ⑤.5），提准不提弱；③ **三层兜底逻辑**：层1主链路(agnes-2.5+glm并行)→层2补位(agnes-2.0，文本≤8K 时 lite 二次补位)→层3终极兜底(**当前对话主模型亲自执行子代理任务**，不依赖外部 API，绝不空转)；免费模型不稳定必须有兜底，红线增补「三层兜底缺一不可」；成本门控：quick 禁用 / standard 仅摘要(agnes-2.0 单模型) / deep 全开(分类双交叉) |
| **v2.6.0** | 2026-08-22 | 新增**子代理加工层**（可选增强，不削弱整体能力）：层 3 结果进证据板前，用免费子代理模型（**glm-4-flash 优先** / lite 兜底，脚本 `<PuppetMaster>/free_models.py`）做分类/打标/去重摘要，提前识别查询跑题（⑤.5 触发信号）；实测验证（Alibaba 财报案例：跑题"科技"→重构后"财经"）；证据板五列/信源层级 D.4/交叉验证规则 6/三道质检全部不变，红线：加工不替代判断、不降低证据标准；按档位成本门控（quick 禁用 / standard 摘要 / deep 全开） |
| **v2.5.0** | 2026-08-16 | **四大搜索范式融入**（OSINT/Deep Research/学术检索方法论综合）：① 新增 ⑤.8 **滚雪球搜索**（Snowball/Citation Chaining）——从 L1/L2 高价值文章出发沿引用链/外链/作者名纵向扩展发现，与证据板横向验证互补；② D.3 增强为「政策溯源 + 时间考古」，新增 D.3.1 **时间考古**（Temporal Archaeology）——wayback 渠道从"死链还原"提升为独立时间维度搜索策略，追踪政策/企业表述/数据的历史演变；③ 新增 ⑤.9 **实体追踪**（Entity Tracing）——从"查询驱动"切换到"实体驱动"，deep 档层 2 额外产出实体追踪表，跨平台追踪关键人/机构/概念踪迹；④ 反思三问增补 **Learnings 查询树**——deep 档每轮搜索提炼 learnings 列表（新行话/新实体/新角度），下一轮查询必须用新知识构造，形成递归扩展树（与 IterResearch/DeepRAG 同构）。层 2/层 3 描述同步更新 |
| **v2.4.1** | 2026-08-16 | **回声搜索（Echo Search）降级策略**：层 3 反爬 blocked 时，强制规则 5 从「重试→降级」二级扩为「重试→回声→降级」三级递进——提取被 blocked 页面标题/关键句投到其他渠道（baidu→bing_cn→yandex→so360/sogou）找同内容镜像，回声失败才走 fallback。新增 ⑤.7 回声搜索专节（查询构造/渠道优先级/适用边界/成本门控/来源标注 `echo:{原}→{镜像}`）；⑥ 补回声搜索示例；Yandex 备注修正（直连相对稳定，非「不稳定」）；渠道数 33→39 已在 v2.3.0 体现，本版本无新增渠道 |
| **v2.4.0** | 2026-08-13 | **实测暴露修复 + 三道质检**：自测执行暴露四问题——⑤ 缺 API 型渠道查询语法（新增 ⑤.6：OpenAlex `title.search`/GitHub `in:name+created:`/HN `tags`/Crossref `bibliographic`，实测教训「search= 模糊匹配召回无关文献」）；规则 7 口径客观化（「零新增关键事实」→「零新增行」）并声明规则 7 优先于规则 1（饱和优先但渠道缺口须声明）；quick 档 ddg 降级目标写明（→ so360，禁止降双引擎）；⑤.5 补失败后重构示例。新增**三道质检**（综合前）：关 1 覆盖度检查表（子维度 covered-or-declared，依据 DeepSearchQA comprehensiveness）/ 关 2 引用抽查协议（L1+高置信全重开核验、中置信抽查≥30%，依据 From Fluent to Verifiable）/ 关 3 查询日志（审计+经验沉淀）；层 0 brief 补合格样例与合格线；渠道数 31→33 修正 |
| v2.3.1 | 2026-08-13 | **前沿范式对齐（三处外科手术）**：新增**证据板**（层 3→综合的工作记忆，事实-来源-层级-独立信源数-置信度五列，交叉验证计数落地，最终报告只从证据板生成）；强制规则 7 **饱和即停**（连续 2 轮零新增关键事实即转综合，与规则 1 下限互补）；回退循环扩为**反思三问**（新事实/缺口/下轮构造依据）。依据：S1-DeepResearch（少而准的工具调用）、IterResearch（轮次工作区重构）、dzhng/deep-research（learnings 机制） |
| v2.3.0 | 2026-08-13 | **三件套重构**：政策/机制分离——渠道数据（原 ③⑤ 特化端点/F.2/F.3 端点表）全部抽为机器可读 `engines.yaml`（31 渠道 + 路由组 + 分级 + 探活标记），手册瘦身为纯方法论；新增**层 0 侦查搜索**（两类歧义分流 + research brief 模板 + 按档成本门控，设计源自 STORM 视角发现 + LangChain open_deep_research Scope 阶段）；新增 `selftest.py`/`selftest.bat` 一键探活写回 yaml（HTTP 200 不可信，按 ok/bad_marker 判真实性）；强制规则补第 6 条交叉验证；本手册成为唯一入口，URL/状态/降级只在 yaml 维护 |
| v2.2.6 | 2026-08-13 | 审计修复：网络出口维度（③ 双状态列、层 3 预检、规则 1 放宽、② 受限兜底路由）；google/google_hk 实测 JS 壳页 → 主力改 bing_intl/ecosia；baidu_xueshu 403 弃用；cache: 下线改 Wayback；附录编号修复（F.3 缺口/字母乱序）；A.1.4/A.1.5 补 Panorama 指针；F.1 补 limiter:false |
| v2.2.5 | 2026-08-13 | local-deep-research 完整依赖链实测（190+ 包），F.5 补重型依赖警告 |
| v2.2.4 | 2026-08-13 | academic-search 实测为 Claude Code Skill 非 MCP Server；stars 修正 ~8.9k |
| v2.2.3 | 2026-08-13 | 新增 AI 答案层（local-deep-research 参考实现）；展开 academic-search 技术栈 |
| v2.2.2 | 2026-08-13 | 学术端点补测（bing_academic ✅ / google_scholar 反爬 / weixinmoni 死链删除） |
| v2.2.1 | 2026-08-13 | 审计修复：补回 ⑤.5、重试机制、F.1 实例表修正 |
| v2.2.0 | 2026-08-13 | 三层执行架构；引擎速查表单一事实源；Bing News RSS→Google News RSS；arXiv/bioRxiv 端点修正 |
| v2.1.9 | 2026-08-12 | 附录 G 跨设备接入 SearXNG MCP；实测可用性标注 |
| v2.1.8 | 2026-08-12 | D.6 社区信号源；SearXNG MCP 桥；academic-search 语义检索 |
| v2.1.7 | 2026-08-12 | 附录 F 免费搜索资源增补 |
| v2.1.6 | 2026-06-28 | D.2 论点驱动逆向查询；D.4 信源分层 L1-L4；D.5 想象族 |
| v2.1.5 | 2026-06-28 | 附录 D：Panorama 投研搜索 Playbook |
| v2.1.4 | 2026-06-28 | 意图路由规则表；调用分级；合规声明 |
| v2.1.3 | 2026-06-28 | 初始版本（拆分自 Panorama 附录） |
| v2.0.x | 2026-02-06 | 9 国际引擎、DDG Bangs、WolframAlpha |
| v1.0.0 | 2026-02-04 | 初始版本，8 国内引擎 |

---

`[KuMoo]` · 多元搜索方法论手册 v3.1.0（2026-10-07：吸收 tanzi 采集 Skill 三项设计——证据板「取回状态」列 + 关 1 信源类别覆盖核对 + F.5 解析锚点 fixture 回归）· 三件套：本手册 + engines.yaml + selftest.py
