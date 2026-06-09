# Agent Evaluation 方法调研报告 2026

日期：2026-06-09

本文关注“如何评估 agent”，尤其是能操作工具、浏览器、OS、代码仓库、SaaS、文件和工作流的下一代 agent。

---

## 0. 推荐呈现逻辑：从 Societas 的问题讲到 Societas 的 Eval

这份报告的主线不应该从“有哪些 benchmark”开始，而应该从 Societas 自己的产品问题开始。行业 benchmark 是证据和方法库，不是故事起点。

推荐讲法：

```text
1. 先讲 Signals to Satisfaction memo：Societas 遇到了什么问题
2. 再讲解决问题的两步：Stage 1 Evaluation -> Stage 2 Adaptation
3. 再讲 Mid+Heavy 用户：我们的核心用户到底在用 Societas 做什么
4. 再讲业内 Agent Eval 形式：只讲和 Societas 相关的几类
5. 再讲其他产品如何做 eval：Edge User Journey 等文档后续补入
6. 最后引出我们要做的 Societas Eval：一个专业文件生产系统的 workbench
```

一句话主张：**Societas 的 eval 不是通用 agent 跑分，而是一个围绕专业交付物、项目连续性、修订负担和用户满意度闭环的 product eval system。**

---

## 1. 先讲问题：为什么 Societas 不能只采用行业趋势

Source: https://closing-the-gap-review.pages.dev/

来自 `From signals to satisfaction — closing the loop in Societas` 的核心判断是：行业正在快速采用 memory、personalization、workflow、agent skills 等能力，但这些能力本身不能保证用户觉得“系统懂我”。

真正的 gap 不在于有没有 memory，而在于：

```text
用户在真实使用中留下大量信号：接受、修改、重复生成、返回、放弃。
但系统没有把这些信号读成“质量”和“满意度”。
因此下一次仍然不知道什么对这个用户是好的。
```

这解释了为什么很多产品已经有 memory / personalization，用户仍然会觉得系统没有变得更懂自己。功能在工作，但底层没有一个可靠的 evaluation substrate，导致 adaptation 只能在弱信号上工作。

### 1.1 问题不是离线 benchmark 不够多，而是产品闭环缺 Stage 1

传统 benchmark 能回答：

```text
这个 agent 在某类任务上能不能完成？
这个文件是否符合 gold answer 或 verifier？
这个模型是否比另一个模型强？
```

但 Societas 真正要回答的是：

```text
这个用户对结果满意吗？
他为什么修改？
他是否反复修正同一类问题？
这次失败是 universal failure，还是 user-specific mismatch？
系统有没有把这些信号变成下一次更好的结果？
```

所以，Societas 的 eval 必须从 offline capability eval 扩展到 online satisfaction eval。

---

## 2. 再讲解决办法：Stage 1 Evaluation -> Stage 2 Adaptation

这篇 memo 给出的关键结构是两步：

| Stage | 要解决什么 | 对 Societas 的含义 |
|---|---|---|
| Stage 1 Evaluation | 在真实使用中评估 artifact 质量和用户满意度，并区分 universal signal vs user-specific signal | 先知道“哪里不好、对谁不好、为什么不好” |
| Stage 2 Adaptation | 基于 Stage 1 的判断更新输出、workflow、memory、skill、personalization | 再决定改全局能力、项目记忆，还是个人偏好 |

这个顺序不能反过来。没有 Stage 1，Stage 2 会变成盲目个性化：系统记住了一些表面信号，但不知道这些信号代表满意、不满意、通用失败，还是个人偏好。

因此 Societas 的完整闭环应该是：

```text
User behavior signal
  -> Satisfaction / quality evaluation
  -> Attribution: universal vs user-specific vs project-specific
  -> Adaptation target: global workflow / verifier / project memory / personal memory
  -> Next run improves
  -> Measure whether correction burden actually drops
```

这也是本报告后面所有 benchmark 选择的前提：行业 benchmark 只能补 Stage 1 的一部分，不能替代真实用户满意度闭环。

---

## 3. 再讲核心用户：Mid+Heavy Users 证明 Societas 是专业文件生产系统

Sources:

- 20260526 Societas eDAU vs Artifacts Divergence: https://microsoftapc-my.sharepoint.com/:u:/g/personal/alexyuan_microsoft_com/IQCFhohRlOsARrqC34hkMXvGAS3cbjykyePFfP8bMWNPqCY?e=I4BWRw
- Mid-Heavy User Samples: https://microsoftapc-my.sharepoint.com/:u:/g/personal/alexyuan_microsoft_com/IQAvHzSgiAjWSqZFsWKYl8O6AeB48lZ4c-LyYwSTsxfkIuk?e=C9Qljc

Mid+Heavy 用户样本说明，Societas 的核心用户不是轻量聊天用户，而是高强度专业文件生产者。他们在 1-6 天内围绕一个职业场景、项目或案件连续生成大量 PPT / Word。

最重要的模式不是“用户问了很多问题”，而是：

```text
短时间内围绕同一专业目标，连续生产一组可交付文件，
这些文件之间有上下文、风格、术语、版本和业务一致性。
```

代表性真实场景：

| 用户类型 | 样本信号 | 对 eval 的启发 |
|---|---|---|
| JP OT 老师 | 89 个 PPT，56 个临床主题，构建神经发育/感觉统合课程体系 | 需要评专业课程 PPT、临床准确性、系列连贯性 |
| KE 初创顾问 | 73 个 Word，约 40 个项目，构建公司 NDA、SLA、服务目录、客户提案 | 需要评公司文档体系完整性和跨文档一致性 |
| CH 破产清算律师 | 89 个 Word，真实案件 I00068，债权人名单多次迭代 | 需要评法律 Word、事实一致性、版本安全和附件引用 |
| BR 地理教育教师 | 55 个 PPT，18 个地理/环境教育项目 | 需要评教育内容批量生产和课程系列一致性 |
| KR 财务分析师 | 48 个 PPT，Asan 系列企业财报分析 | 需要评财务图表、数据解释、结论支撑 |
| 中度用户群 | 医学教育、BIM、海事合规、NGO 儿童安全、LLM 技术规格、脑卒中研究 | 需要覆盖 market / domain 多样性，而不是只评英文商业 deck |

这批用户把 Societas 的 eval 定义得更清楚：**我们要评的不是通用 task success，而是专业文件生产系统的 project-pack success。**

---

## 4. 再讲行业 Eval：只讲和 Societas 相关的几类

行业 eval 很多，但和 Societas 最相关的是五类。

### 4.1 GDPval-like Deliverable Eval

代表和链接：

| 代表 | 链接 | 我们复用什么 |
|---|---|---|
| GDPval | https://openai.com/index/gdpval/ | 真实职业任务、专业交付物、专家 rubric、专家盲评 / preference 判断 |
| SWE-Lancer | https://openai.com/index/swe-lancer/ | 把任务表现映射到真实经济价值，用真实工作任务而不是 toy task |
| MLE-bench | https://openai.com/index/mle-bench/ | 垂直专业任务可以用客观 metric / leaderboard 评估 |
| PaperBench | https://openai.com/index/paperbench/ | 复杂专业交付物要拆成层级 rubric 和可评分子任务 |

这一类的 learning：评估应该基于真实职业任务、真实交付物、专家 rubric、经济价值或专业基线，而不是只评考试题。

Societas 的用法：

```text
优先复用 GDPval 的方法和 gold set：
真实职业任务 + deliverable + domain rubric + expert/user preference。

Societas 只需要补一个适配层：
PPT/Word/Excel 产物、项目包一致性、修订负担、在线满意信号。
```

最优先复用：**GDPval 的任务设计和专家 rubric**。PaperBench 的层级 rubric 可以作为复杂项目包评分的补充。

### 4.2 Workflow / Workplace Eval

代表和链接：

| 代表 | 链接 | 我们复用什么 |
|---|---|---|
| WorkArena / BrowserGym | https://arxiv.org/abs/2403.07718 | 企业 SaaS / browser workflow 的任务设计、action space、workflow completion 评估 |
| TheAgentCompany | https://arxiv.org/abs/2412.14161 | 模拟公司环境里的 digital worker 任务、跨工具/同事沟通/长程工作评估 |
| tau-bench | https://arxiv.org/abs/2406.12045 | 用户对话 + 工具/API + domain policy + 最终数据库状态的评估结构 |
| AppWorld | https://arxiv.org/abs/2407.18901 | 多 app/API 工作流、state-based unit tests、collateral damage 检查 |

这一类的 learning：真实 agent 不只是生成文件，还要在 SaaS、浏览器、API、团队上下文、规则约束中推进任务。

Societas 的用法：

```text
Societas coworker / workflow layer 需要评 workflow state、tool calls、approval gates、domain policy。
```

最优先复用：**tau-bench 的 user/tool/policy/final-state 结构 + WorkArena 的 workplace workflow 任务形态**。Societas 还要补 Office artifact 和 project-pack consistency。

### 4.3 Office / Artifact Verifier

代表和链接：

| 代表 | 链接 | 我们复用什么 |
|---|---|---|
| SWE-bench | https://arxiv.org/abs/2310.06770 | 用可执行测试判断真实任务是否完成，而不是相信模型自述 |
| AppWorld | https://arxiv.org/abs/2407.18901 | state-based unit tests 和 collateral damage 检查 |
| OSWorld | https://arxiv.org/abs/2404.07972 | 初始状态配置、execution-based evaluation script、环境状态检查 |
| AndroidWorld | https://arxiv.org/abs/2405.14573 | success-checking / tear-down 逻辑，动态任务变体下的鲁棒性检查 |

这一类的 learning：能用程序检查的，先用程序检查；不要让 agent 自己说“完成了”。

Societas 的用法：

```text
PPT / Word / Excel 必须先过工程验收：能打开、结构完整、公式/图表/引用不坏、无空白占位。
```

最优先复用：**SWE-bench 的 test-as-verifier 思想 + OSWorld 的 execution-based script 结构**。Societas 要补的是 Office-specific verifier：pptx/docx/xlsx parser、结构检查、引用/公式/图表检查、版本 diff 检查。

### 4.4 Trajectory / Milestone / Evidence-chain Eval

代表和链接：

| 代表 | 链接 | 我们复用什么 |
|---|---|---|
| OS-Themis | https://arxiv.org/abs/2603.19191 | 把 GUI agent trajectory 拆成可验证 milestone，并审计 evidence chain |
| PaperBench | https://openai.com/index/paperbench/ | 复杂专业任务拆成层级 rubric 和大量可评分子任务 |
| WebLINX | https://arxiv.org/abs/2402.05930 | 多轮网页导航中的 action history、observation、用户对话上下文 |
| WorkArena / BrowserGym | https://arxiv.org/abs/2403.07718 | browser agent 的 action / observation trace 和工作流节点 |

这一类的 learning：复杂任务不能只看最终结果，要看过程轨迹、milestone 和 evidence chain。

Societas 的用法：

```text
我们要记录从用户目标到产物交付的 trajectory，才能知道失败发生在哪一步。
```

最优先复用：**OS-Themis 的 milestone + evidence-chain critic**。Societas 要补的是 artifact plan、Office verifier result、user revision diff、accepted-version preservation 这些文件生产专属 evidence。

### 4.5 Cost / Reliability / Regression Eval

代表和链接：

| 代表 | 链接 | 我们复用什么 |
|---|---|---|
| AI Agents That Matter | https://arxiv.org/abs/2407.01502 | cost + accuracy 联合评估、holdout、防 benchmark overfit、可复现性纪律 |
| tau-bench | https://arxiv.org/abs/2406.12045 | 多次运行可靠性、pass^k / consistency、规则遵循稳定性 |
| AndroidWorld | https://arxiv.org/abs/2405.14573 | 动态任务变体下的鲁棒性和环境 reset / tear-down |
| SWE-Lancer | https://openai.com/index/swe-lancer/ | 将任务成功映射到真实经济价值和成本收益 |

这一类的 learning：agent eval 不能只看 accuracy，还要看成本、稳定性、holdout、防过拟合、regression。

Societas 的用法：

```text
每次 workflow / memory / skill 改动都要看旧任务是否退化，用户修订负担是否真的下降。
```

最优先复用：**AI Agents That Matter 的 cost / holdout / reproducibility 纪律 + tau-bench 的 reliability 指标**。Societas 要补的是 repeat correction rate、human cleanup time、accepted-version preservation 和 user-specific adaptation gain。

---

## 5. 再讲其他产品如何做 Eval：Edge User Journey 等待补充

这一节先留出位置，等 Edge User Journey 或其他产品 eval 文档补齐后再填实。

建议到时候重点看五个问题：

1. 他们评的是 feature success、task success，还是 user journey satisfaction？
2. 他们如何采集真实使用信号：click、return、abandon、edit、export、manual feedback？
3. 他们有没有把 offline benchmark 和 online product telemetry 连起来？
4. 他们如何做 attribution：产品 bug、模型能力、用户偏好、内容质量、外部环境？
5. 他们的 eval 结果如何进入下一轮产品更新：prompt、UX、workflow、memory、policy？

在 Societas 的叙事里，这一节的作用不是“照抄别人的 eval”，而是回答：**我们如何把行业和兄弟产品的经验，转成适合专业文件生产场景的 eval loop。**

---

## 6. 最后引出我们要做的 Eval：复用现有 Benchmark + Societas 适配层

综合上面几层，Societas 不需要从零自研一套 benchmark。更合理的做法是：**复用现有 benchmark 的成熟方法，把它们组合成一个 Societas-specific eval workbench。**

具体选择：

| 用哪个 | 负责评什么 | Societas 怎么用 |
|---|---|---|
| [GDPval](https://openai.com/index/gdpval/) | 真实职业任务、专业交付物、专家 rubric、盲评偏好 | 作为 north-star deliverable eval；优先复用其任务设计、rubric 结构和专家评审方式 |
| [WorkArena](https://arxiv.org/abs/2403.07718) / [TheAgentCompany](https://arxiv.org/abs/2412.14161) | 工作环境、企业 SaaS、长程 workflow、digital worker 行为 | 作为 workflow / workplace eval 方法；评浏览器、SaaS、文件、团队上下文中的任务推进 |
| [tau-bench](https://arxiv.org/abs/2406.12045) / [AppWorld](https://arxiv.org/abs/2407.18901) | 用户交互、API/tool 调用、domain policy、最终状态和 collateral damage | 作为 tool-state-policy eval 方法；评工具调用正确性、规则遵守和副作用 |
| [SWE-bench](https://arxiv.org/abs/2310.06770) / [AppWorld](https://arxiv.org/abs/2407.18901) / [OSWorld](https://arxiv.org/abs/2404.07972) 的 verifier 思路 | 程序化验收、状态检查、可复现 evaluation scripts | 用来做 Office verifier：pptx/docx/xlsx 能打开、结构完整、公式/引用/图表不坏 |
| [PaperBench](https://openai.com/index/paperbench/) / [OS-Themis](https://arxiv.org/abs/2603.19191) | 层级 rubric、milestone、evidence chain、复杂任务拆分 | 用来评专业项目包和 trajectory，而不是只看最终文件 |
| [AI Agents That Matter](https://arxiv.org/abs/2407.01502) | cost、holdout、防过拟合、可复现性 | 作为评测纪律：所有分数都要配 cost / latency / regression / holdout |
| Societas-defined: Satisfaction / Revision Loop | 目前没有统一行业 benchmark；可参考产品 telemetry 和 preference eval 思路 | 评用户是否少改、少重来、更快接受：download/export、return-to-project、manual edit distance、regenerate、abandon、repeat correction |
| Societas-defined: Attribution | 目前没有统一行业 benchmark；来自 Signals to Satisfaction memo | 区分 universal failure、user-specific mismatch、project-specific constraint、tool/artifact failure，决定该改全局能力、项目记忆还是个人偏好 |
| Societas-defined: Adaptation Effectiveness | 目前没有统一行业 benchmark；需要真实历史轨迹 | 评系统是否越用越懂用户：同类任务后续 revision rounds、edit distance、repeat correction、time-to-acceptance 是否下降 |

所以我们不是“造一个孤立的 Societas benchmark”，而是做一个用户价值优先的组合。Memo 的主张是：不要为了评测完整性把所有行业指标都平铺跑一遍，而要优先保留用户真正关心的东西。

```text
Societas Workbench Score = 交付物是否专业可用 + 用户是否少改 + 过程是否可信
```

### 6.1 Hard Gates：先过门槛，不参与加权

有些问题不能靠加权分抵消。只要发生，任务就不能算成功：

| Hard Gate | 触发条件 |
|---|---|
| Office artifact integrity | pptx/docx/xlsx 损坏、打不开、不可编辑、关键结构缺失 |
| Critical domain risk | 医疗、法律、财务、儿童安全等领域出现关键危险错误或 unsupported claim |
| Unauthorized external action | 未经用户确认就发送、删除、提交、付款、外部发布 |
| Privacy / policy violation | 泄露敏感信息、越权访问、违反权限边界 |
| Severe state corruption | 误改源文件、污染 SaaS/API 状态、破坏已确认版本 |

### 6.2 加权总分：50 / 20 / 20 / 5 / 5

通过 hard gates 后，再算 100 分：

| 模块 | 权重 | 主要参考 | 采集方式 | 评分方式 |
|---|---:|---|---|---|
| GDPval-style professional deliverable quality | 50 | [GDPval](https://openai.com/index/gdpval/)、[PaperBench](https://openai.com/index/paperbench/)、[SWE-Lancer](https://openai.com/index/swe-lancer/)、[MLE-bench](https://openai.com/index/mle-bench/) | source materials、最终 PPT/Word/Excel/PDF、专家 rubric、LLM judge、人工抽样评审 | domain correctness、completeness、ready-to-use、professional quality、style/tone fit |
| Satisfaction / revision loop | 20 | Signals to Satisfaction memo、online product telemetry、[MT-Bench / Chatbot Arena](https://arxiv.org/abs/2306.05685) 的 preference 思路 | download/export、return-to-project、manual edit distance、regenerate、abandon、repeat correction、revision rounds | 用户是否接受、少改、少重来、同类错误是否减少 |
| Process trust: trajectory / milestone / workflow | 20 | [WorkArena](https://arxiv.org/abs/2403.07718)、[TheAgentCompany](https://arxiv.org/abs/2412.14161)、[tau-bench](https://arxiv.org/abs/2406.12045)、[AppWorld](https://arxiv.org/abs/2407.18901)、[OS-Themis](https://arxiv.org/abs/2603.19191)、[WebLINX](https://arxiv.org/abs/2402.05930) | workflow trace、tool call log、approval log、action/observation、milestone evidence、verifier report、final state snapshot | workflow 是否跑通、tool/policy/state 是否正确、milestone evidence 是否完整、授权确认是否可审计 |
| Project-pack consistency | 5 | [GDPval](https://openai.com/index/gdpval/) 扩展、[PaperBench](https://openai.com/index/paperbench/) 层级任务、Societas Mid/Heavy samples | project_id、同项目多文件、版本 diff、实体/术语抽取、style profile、cross-artifact references | 跨文件术语、事实、结构、风格、引用一致性 |
| Cost / reliability | 5 | [AI Agents That Matter](https://arxiv.org/abs/2407.01502)、[tau-bench](https://arxiv.org/abs/2406.12045)、[AndroidWorld](https://arxiv.org/abs/2405.14573) | token cost、latency、same-task repeated runs、holdout/regression results | cost-normalized success、pass@1 / consistency、regression penalty |

公式：

```text
Societas Workbench Score =
  50 GDPval-style Deliverable Quality
+ 20 Satisfaction / Revision Loop
+ 20 Process Trust: Trajectory / Milestone / Workflow
+  5 Project-pack Consistency
+  5 Cost / Reliability
= 100
```

这个权重表达的是：70 分直接围绕用户价值，30 分围绕交付过程和系统健康。

```text
用户价值：50 交付物质量 + 20 满意度/修改负担
过程健康：20 过程可信 + 5 项目包一致性 + 5 成本可靠性
```

### 6.3 第一版 Eval Suite

第一版可以从 16-24 个高质量任务开始，不追求大而全。

| Suite | 来源 | 第一版重点 |
|---|---|---|
| Clinical / Medical Education PPT | JP OT、MX 内分泌 | 医学准确性、教学结构、系列课程连贯性 |
| Business / Consulting Deck | SG/VN/PT 咨询与销售用户 | 商业逻辑、提案结构、客户 ready 程度 |
| Company Document Pack | KE 初创顾问 | NDA / SLA / 服务目录 / 客户提案之间的一致性 |
| Legal / Formal Word Pack | CH 法律案件 | 人名、金额、日期、附件引用、版本安全 |
| Technical / Research Word | GB LLM spec、BR 脑卒中、PT BIM | 技术完整性、引用 grounding、表格/图示正确 |
| Education / Training Pack | BR 地理、CD NGO、US 培训 | 学习目标、讲师可用性、quiz / answer key 一致 |

### 6.4 最终论点

Societas 要证明的不是“agent 能不能完成一个 benchmark task”，而是：

```text
这个系统能不能稳定交付真实职业文件，
能不能在一组项目文件中保持一致，
能不能减少用户修订负担，
能不能区分通用失败和用户偏好，
能不能让用户越用越觉得它懂自己。
```

---

## 7. 附录：详细方法库与证据

本附录保留详细方法库：行业 eval 分类、benchmark 说明、指标体系、harness 架构、不同 agent 类型方案和参考来源。HTML 版本只展示前面的主线叙事。

### 7.1 Agent Eval 总览

现在的 agent evaluation 已经不能沿用传统 LLM benchmark 的思路，只问“最终回答对不对”。Agent 的核心特征是：它会多步行动、调用工具、改变环境状态、生成文件、和用户协作，并且可能在任务后更新 memory、skill、workflow 或 policy。因此，评估 agent 必须从单点答案评测升级为完整的 execution evaluation。

最可靠的范式可以概括为：**multi-signal, evidence-grounded, environment-backed evaluation**。

也就是：

```text
Task Spec
  -> Instrumented Run
  -> Evidence Capture
  -> State / Artifact / Programmatic Verification
  -> Trajectory / Milestone Review
  -> Safety / Cost / Regression Checks
  -> Human or LLM Judge for Ambiguous Parts
  -> Failure Attribution and Eval Set Update
```

核心结论：

1. **Outcome success 是必要但不充分**：最终结果正确，不代表过程安全、稳定、可复现。
2. **能用确定性 verifier 时，不要只用 LLM judge**：文件、代码、数据库、网页状态、API 状态都应该优先用程序检查。
3. **轨迹评估正在变重要**：WebArena、OSWorld、AndroidWorld、WorkArena、tau-bench 等都把 agent 放进可交互环境，观察 action、state、tool use 和最终状态。
4. **长任务需要 milestone-level evaluation**：只看终点会误判，应该把任务拆成可验证 checkpoint，并要求每个 checkpoint 有证据链。
5. **LLM-as-judge 可以用，但要被约束**：必须有 rubric、evidence、校准集、偏差检查，并尽量与生成 agent 分离。
6. **agent eval 必须同时看成本、可靠性、可复现性和安全**：AI Agents That Matter 指出，只看 accuracy 会鼓励复杂、昂贵、不可复现的 agent。
7. **self-evolving agent 的评估重点是学习曲线和 regression**：不能只问“这次有没有变好”，还要问“held-out 任务是否变好，旧任务是否退化，安全风险是否上升”。

一句话：**评估 agent，不是给答案打分，而是给一次可审计的工作执行打分。**

---

### 7.2 为什么 Agent Eval 和普通 LLM Eval 不一样

普通 LLM evaluation 主要评估模型在固定输入上的输出质量，例如准确率、BLEU、ROUGE、exact match、pass@k、LLM judge 分数。Agent evaluation 面对的是另一个对象：一个会持续行动的系统。

Agent eval 至少多了六个维度：

| 维度 | 普通 LLM Eval | Agent Eval |
|---|---|---|
| 输入 | 单轮 prompt 或固定上下文 | 任务目标、初始环境、工具权限、用户偏好、历史状态 |
| 输出 | 文本、代码片段、答案 | 环境状态变化、文件、API 调用、浏览器操作、工作流完成度 |
| 过程 | 通常不评或只看 chain-of-thought proxy | 必须评 action / observation / tool call / retry / recovery 轨迹 |
| 成功标准 | 答案匹配或 judge 打分 | 目标状态达成、artifact 可用、side effect 合法、用户意图满足 |
| 风险 | 幻觉、偏见、有害内容 | 越权操作、误删、错误提交、隐私泄露、付款/发送/发布等不可逆动作 |
| 复盘 | 错题集或 benchmark score | failure attribution、workflow 修复、memory/skill 更新、regression 验证 |

因此，一个成熟的 agent eval 不能只看最终答案，而要评估：

```text
goal completion
environment state
artifact correctness
tool-call validity
trajectory quality
recovery behavior
human intervention
safety boundaries
cost and latency
regression across old tasks
generalization to held-out tasks
```

---

### 7.3 当前 Agent Evaluation 的主流方法谱系

#### 7.3.1 Outcome / Task Success Evaluation

这是最基础的一层：任务最终有没有完成。

典型评估方式：

- 最终答案是否命中 gold answer。
- 文件是否生成、可打开、内容完整。
- 代码 patch 是否通过测试。
- 数据库或 API 状态是否等于目标状态。
- Web / OS / App 环境是否达到预期终态。

代表：

- SWE-bench：给定真实 GitHub issue，agent 修改代码，最终通过测试判断是否解决问题。
- WebArena：在真实网站环境中执行任务，重点评估 functional correctness of task completions。
- tau-bench：对话结束后比较最终数据库状态和标注的 goal state。
- OSWorld / AndroidWorld：通过 execution-based evaluation script 或 success-checking logic 检查真实环境状态。

优点：

- 直观，容易和业务结果对齐。
- 适合做排行榜、A/B test 和 regression gate。
- 对工程团队来说最有说服力。

缺点：

- 可能漏掉危险过程。例如最终页面改对了，但中途访问了不该看的数据。
- 可能误伤合理替代路径。例如任务目标达成但不是 gold trace。
- 对开放任务不够细。例如研究报告、沟通质量、方案设计很难只有一个终态。

结论：Outcome success 是第一层 gate，但不能作为唯一指标。

---

#### 7.3.2 State-based / Environment-backed Verification

Agent 与环境交互，所以最可靠的 evaluation 往往来自环境状态检查。

例子：

| 场景 | 状态检查 |
|---|---|
| 浏览器 agent | DOM 状态、URL、页面字段、下载文件、后台数据库记录 |
| SaaS agent | ticket 状态、CRM 字段、审批状态、订单状态、审计日志 |
| Office agent | pptx/xlsx/docx 是否能解析，页数、表格、公式、引用、样式是否符合要求 |
| Coding agent | 单测、集成测试、lint、typecheck、patch diff、issue reproduction |
| OS / GUI agent | 文件系统变化、应用配置、窗口状态、命令输出、系统设置 |
| Mobile agent | Android app state、intent、控件状态、本地数据库、任务 tear-down 结果 |

这类方法的核心不是问 LLM “完成了吗”，而是让环境自己说话。

设计要点：

1. 每个 task 必须有 initial state setup。
2. 每个 task 必须有 success checker。
3. 每次运行后必须能 reset 或 tear down。
4. 重要 side effect 必须有 audit log。
5. verifier 应该尽量确定性、可复现、版本化。

OSWorld 的价值就在这里：它不是只给自然语言题目，而是提供真实电脑环境、初始状态配置和 execution-based evaluation script。AndroidWorld 也类似，每个任务有初始化、成功检查和清理逻辑。

---

#### 7.3.3 Programmatic Tests / Deterministic Verifier

如果能写程序验证，就优先写程序验证。

这包括：

- unit / integration tests
- typecheck / lint / static analysis
- schema validation
- file parser and structural checks
- database diff
- API response check
- browser DOM check
- screenshot pixel / OCR check
- formula recalculation
- link checker
- citation checker
- security policy checker
- permission checker

为什么它重要：

1. 稳定，不受 judge prompt 波动影响。
2. 可复现，适合 CI/CD。
3. 能定位失败原因。
4. 能作为 regression suite。
5. 能防止 agent 用漂亮解释掩盖实际失败。

对于 Societas 这类 information worker agent，programmatic verifier 应该优先覆盖 artifact 和 workflow 终态：

| Artifact / Workflow | Programmatic verifier 示例 |
|---|---|
| PPT | 文件可打开、页数、标题层级、图片引用、模板色板、空白占位、讲者备注 |
| Excel | sheet 数、列名、公式、引用范围、图表、数据类型、透视表、异常空值 |
| Word | 章节结构、字数、标题样式、引用、目录、表格、页眉页脚 |
| Browser workflow | URL、表单值、提交状态、下载文件、业务系统记录 |
| CRM / SaaS | 字段 diff、状态机合法性、审批记录、外发动作确认 |
| Research task | 引用 URL 可访问、摘要覆盖关键点、事实 claim 与来源对齐 |

原则：**能 parse 的不要靠眼看，能 diff 的不要靠感觉，能执行的不要靠口头完成。**

---

#### 7.3.4 Trajectory Evaluation

Trajectory evaluation 评估 agent 执行过程本身。

它关注：

- 是否理解任务并形成合理计划。
- 是否调用了正确工具。
- 是否重复无效操作。
- 是否出现 hallucinated tool call。
- 是否观察环境后更新计划。
- 是否处理错误和异常。
- 是否在高风险动作前请求确认。
- 是否遵守权限边界。
- 是否留下可复盘证据。

典型 trajectory log：

```text
run_id
task_id
model / agent version
environment version
user message
retrieved memory
plan
step events:
  - observation
  - thought / hidden state summary if allowed
  - action
  - tool call
  - tool result
  - artifact diff
  - verifier result
  - cost / latency
  - user approval
final state
failure attribution
```

Trajectory eval 的价值：

1. 能区分“结果错”和“流程错”。
2. 能发现 reward hacking。例如 agent 改测试而不是修 bug。
3. 能训练和改进 workflow。
4. 能为 self-evolution 提供原始经验。
5. 能把失败归因到 planner、executor、tool、memory、verifier、harness 或 policy。

适用场景：

- long-horizon agent
- browser / GUI agent
- coding agent
- multi-agent system
- self-evolving agent
- 高风险业务流程

注意：trajectory eval 不应要求唯一正确路径。真实任务常有多条可行路径，所以评估应关注约束、证据和关键 milestone，而不是强制 trace exact match。

---

#### 7.3.5 Milestone-level / Evidence-chain Evaluation

长任务只看终点很脆，最好拆成 milestone。

例子：

```text
Task: 更新一个客户 CRM 记录并生成跟进邮件草稿

Milestone 1: 找到正确客户记录
Evidence: CRM URL + customer_id + name match

Milestone 2: 读取最近互动记录
Evidence: note timestamps + extracted facts

Milestone 3: 更新下一步状态
Evidence: field diff + saved timestamp

Milestone 4: 生成邮件草稿
Evidence: draft text + required facts + no unsupported claims

Milestone 5: 等待人工确认
Evidence: no external send action before approval
```

OS-Themis 代表了这个方向：它不是一个单一 judge，而是把 GUI agent 的轨迹拆成可验证 milestone，隔离关键证据，再通过 review mechanism 审计 evidence chain 后做最终判断。

Milestone eval 的优势：

- 给长任务 partial credit。
- 更容易定位失败阶段。
- 更适合作为训练 reward。
- 比最终 LLM judge 稳定。
- 能显式捕获安全 gate。

设计建议：

1. 每个 milestone 要有 observable evidence。
2. 每个 milestone 要有 pass / fail / unknown 三态，不要强行二分。
3. 每个 milestone 要绑定对应 verifier。
4. 高风险 milestone 必须 human-gated。
5. 允许 agent 走不同路径，但必须提交等价证据。

---

#### 7.3.6 LLM-as-a-Judge / Rubric Evaluation

LLM judge 适合评估开放输出，例如：

- 研究摘要质量。
- 方案完整性。
- 用户沟通是否得体。
- 报告结构是否清晰。
- 多源信息是否综合得好。
- 是否满足隐含偏好。

但 LLM judge 不是万能 verifier。MT-Bench / Chatbot Arena 相关研究表明，强 LLM judge 可以在很多开放偏好任务上接近人类偏好，但也存在 position bias、verbosity bias、self-enhancement bias 和 reasoning limitation。

因此，agent eval 中使用 LLM judge 时必须加约束：

| 要求 | 说明 |
|---|---|
| Rubric 明确 | 不要只问“好不好”，要拆成准确性、完整性、证据、格式、安全等维度 |
| Evidence-grounded | judge 必须基于 trace、artifact、source、verifier result，而不是只看最终话术 |
| Calibration set | 用人工标注样例校准 judge，观察一致性和漂移 |
| Judge separation | 生成 agent 和评估 agent 分离，避免自评自夸 |
| Blind / pairwise when useful | 对主观质量可用匿名 pairwise 比较降低打分尺度漂移 |
| Bias checks | 检查长度偏好、位置偏好、品牌/model 偏好、过度惩罚简洁回答等问题 |
| Disagreement routing | judge 不确定或多个 judge 分歧大时转人工或更强 verifier |

推荐用法：

```text
deterministic checks first
  -> LLM judge only for ambiguous semantic quality
  -> require evidence references
  -> compare with calibration examples
  -> store judge rationale and score
  -> periodically audit against human labels
```

不推荐用法：

```text
Agent: 我完成了任务。
Judge: 看起来不错，给 9/10。
```

这种 judge 很容易被 agent 的自信表达骗过。

---

#### 7.3.7 Human Review / Human-like Oversight

人类评估依然不可替代，尤其在高风险、模糊、价值密集或不可逆任务中。

适用场景：

- 发送邮件、外部发布、付款、提交订单、删除数据。
- 法律、医疗、金融、合规相关输出。
- 企业关键系统状态变更。
- 需要判断 tone、政治风险、商业敏感度的内容。
- eval set 的 gold label 建设。
- evaluator drift 审计。

ANCHOR 这类 2026 工作的启发是：对于 self-evolving agent，人类或 human-like oversight 不一定要管每一步，最有效的干预点往往是 output verification phase。也就是说，监督应该集中在关键结果和关键风险门上，而不是把所有行动都变成人工遥控。

推荐分层：

| 风险等级 | 评估方式 |
|---|---|
| 低风险可逆 | 自动 verifier + 抽样人工复核 |
| 中风险业务影响 | 自动 verifier + LLM judge + human approval checkpoint |
| 高风险不可逆 | 必须人工确认，且保留 evidence packet |
| 安全/合规敏感 | 独立 reviewer + audit log + rollback plan |

---

#### 7.3.8 Reliability / Repeated-run Evaluation

Agent 的一次成功不等于可靠。

很多 agent 在同一个任务上多跑几次表现不稳定，尤其是：

- tool call 参数容易波动。
- 搜索结果受时间影响。
- 浏览器 UI 状态轻微变化。
- 模型采样导致行动路径变化。
- 长任务中一次小错误会连锁放大。

tau-bench 的重要贡献之一是强调多次运行可靠性，用 pass^k 这类指标观察 agent 是否能持续成功，而不是只看一次幸运完成。

常用可靠性指标：

| 指标 | 含义 |
|---|---|
| pass@1 | 单次运行成功率 |
| pass@k | k 次尝试中至少一次成功，适合看采样潜力 |
| pass^k / consistency | 多次独立运行中持续成功的比例，适合看稳定性 |
| retry success rate | 首次失败后能否恢复 |
| variance across seeds | 不同采样随机种子下波动 |
| variance across environment variants | UI、数据、表述变化后波动 |
| mean time to recover | 从工具错误或环境异常中恢复的时间 |

对生产 agent 来说，pass@k 不能掩盖 pass@1 低的问题。用户通常不会接受“跑 8 次总有一次成功”。

---

#### 7.3.9 Cost / Latency / Operational Evaluation

AI Agents That Matter 特别强调：agent benchmark 不能只看 accuracy，也要看 cost。

原因很直接：一个 agent 可以通过更多模型调用、更多工具调用、更多反思轮次、更多 self-consistency 采样来提升分数，但这可能不适合真实产品。

需要记录：

- token cost
- model call count
- tool call count
- browser action count
- wall-clock time
- human intervention count
- retry count
- environment reset cost
- external API cost
- failure recovery cost
- cost per successful task
- cost per 1% improvement

推荐报告格式：

```text
success rate: 72.0%
median cost per successful task: $0.18
p90 latency: 95s
human intervention rate: 12.5%
unsafe action block rate: 2.1%
regression rate on old suite: 1.4%
```

不要只报：

```text
score: 72.0%
```

因为这无法判断 agent 是否值得上线。

---

#### 7.3.10 Safety / Policy / Permission Evaluation

Agent 会行动，所以安全评估必须前置。

重点不是只评“回答是否有害”，还要评：

- 是否越权访问数据。
- 是否在未确认时发送/付款/发布/删除。
- 是否绕过权限系统。
- 是否泄露用户隐私。
- 是否误用工具。
- 是否在不确定时假装确定。
- 是否能识别钓鱼、prompt injection、工具返回污染。
- 是否遵守组织 policy。

安全 eval 应该分为三层：

| 层级 | 内容 |
|---|---|
| Pre-action policy check | 高风险动作前检查权限、风险、确认状态 |
| Runtime guardrail | 工具调用拦截、敏感字段脱敏、外发动作二次确认 |
| Post-run audit | 检查 trace、side effect、数据访问、异常 tool call |

对 browser / SaaS agent 特别重要的是 external side effect policy：

```text
read-only actions: auto allowed if permission exists
draft actions: auto allowed with trace
internal state updates: require verifier + undo path
external irreversible actions: require human approval
payment / deletion / legal submission: require explicit confirmation and audit
```

---

#### 7.3.11 Regression Evaluation

Agent 每次升级 prompt、tool、workflow、memory、skill、policy，都可能让旧任务退化。

Regression eval 要回答：

1. 新版本是否提升目标任务？
2. 旧任务是否退化？
3. 成本是否显著上升？
4. 安全风险是否上升？
5. 失败类型是否转移？
6. 是否只过拟合最近任务？

推荐把 eval set 分成四类：

```text
train / development tasks: 用于开发和调试
held-out tasks: 用于判断泛化
regression tasks: 用于保护旧能力
safety canary tasks: 用于防止越权和安全漂移
```

对于 self-evolving agent，还需要：

```text
source_run_id for each update
change type: memory / skill / workflow / tool policy / verifier
expected benefit
affected task families
validation results
rollback plan
expiry condition
```

---

### 7.4 代表性 Benchmark 和它们的评估启发

#### 7.4.1 总览表

| Benchmark / Work | 主要对象 | 评估重点 | 对 Agent Eval 的启发 |
|---|---|---|---|
| AgentBench, 2023/2025 | 多环境 LLM-as-agent | 8 类交互环境中的 reasoning / decision making | agent 需要在交互环境中评，而不是只做静态问答 |
| WebArena, 2023/2024 | Web agent | 真实网站、长程任务、functional correctness | 可复现 web 环境 + end-to-end task success 是 web agent 基础 |
| SWE-bench, 2023/2024 | Coding agent | 真实 GitHub issue、代码修改、测试通过 | 用真实 repo 和 tests 评估比 toy code 更接近工程能力 |
| GAIA, 2023 | General assistant | 推理、多模态、浏览、工具使用 | 简单人类任务对 AI 仍难，适合评估综合鲁棒性 |
| VisualWebArena, 2024 | Multimodal web agent | 视觉 grounding、网页动作、复杂视觉任务 | 只看 DOM/text 不够，很多任务需要视觉信息 |
| WebLINX, 2024 | Conversational web navigation | 多轮对话、真实网站、人类 demonstration | web agent eval 要考虑用户多轮澄清和 action history |
| WorkArena / BrowserGym, 2024 | Knowledge-work web agent | ServiceNow 等企业软件任务 | SaaS/enterprise agent 应该用真实工作流和业务状态检查 |
| GDPval, 2025 | 经济价值知识工作 | 44 个职业、9 个行业、真实工作交付物、专家盲评 | 对 Societas 最相关：评估文档、slides、spreadsheets、行业 deliverable 是否达到专家工作质量 |
| SWE-Lancer, 2025 | 真实自由职业软件工程 | Upwork 任务、真实报酬、end-to-end tests、管理决策 | 把 agent 表现映射到真实美元价值，适合评估经济影响但只覆盖软件工程 |
| MLE-bench, 2024 | ML engineering agent | Kaggle 风格 ML 工程任务、训练/实验/提交结果 | 垂直专业任务可用客观 leaderboard metric 评估，不必完全依赖专家主观判断 |
| PaperBench, 2025 | AI research replication agent | 复现 ICML 论文、层级 rubric、LLM judge + 人类基线 | 高复杂专业交付物需要分解成大量可评分子任务，而不是只看最终报告 |
| TheAgentCompany, 2024/2025 | 公司环境 digital worker | 浏览、写代码、运行程序、和同事沟通、完成公司任务 | GDPval 的 workflow 化方向：从 one-shot deliverable 走向模拟公司里的连续工作 |
| AppWorld, 2024 | 多 app/API digital task agent | 9 个 app、457 个 API、状态单测、collateral damage 检查 | 适合评估 tool-use workflow、状态变化和意外副作用 |
| AssistantBench, 2024 | 真实耗时网页助手任务 | 开放网页、自动评估、信息搜集和任务完成 | 适合评估 open-web assistant 的真实信息任务，但交付物质量维度弱于 GDPval |
| OSWorld, 2024 | OS / desktop agent | 真实电脑环境、跨应用任务、execution-based scripts | OS agent 需要真实环境、初始状态、执行式 verifier |
| AndroidWorld, 2024/2025 | Mobile agent | Android app、动态任务、success checking、tear-down | 参数化任务和环境变体能测 robustness |
| tau-bench, 2024 | Tool-agent-user interaction | 模拟用户、API 工具、规则遵循、数据库目标状态 | 生产 agent 需要评估用户交互、domain policy 和多次运行可靠性 |
| ToolBench / ToolEval, 2023 | Tool-use agent | API 选择、调用链、复杂工具使用 | 工具调用能力需要专门数据、路径和 evaluator |
| MT-Bench / Chatbot Arena, 2023 | LLM judge 方法论 | 人类偏好、LLM judge 偏差 | LLM judge 可扩展，但必须处理 bias 和校准 |
| AI Agents That Matter, 2024 | Agent benchmark 方法论 | 成本、holdout、防过拟合、可复现性 | 不要只优化 accuracy，必须联合优化 cost 与 real-world utility |
| SWE-agent, 2024 | Coding agent interface | agent-computer interface、SWE-bench pass@1 | 工具/界面设计本身会显著改变 agent eval 结果 |
| OS-Themis, 2026 | GUI reward critic | milestone decomposition、evidence chain、multi-agent critic | GUI reward 不应是单 judge，而应是证据链审计 |
| ANCHOR, 2026 | Self-evolving oversight | output verification phase supervision | 自进化系统最有效的人类监督点常在结果验证阶段 |

#### 7.4.2 这些 benchmark 的共同趋势

从 2023 到 2026，agent eval 的趋势很清楚：

1. **从静态问答走向真实环境**：WebArena、OSWorld、AndroidWorld、WorkArena 都在强调 functional environments。
2. **从最终文本走向状态变化**：数据库、文件系统、浏览器状态、代码测试成为核心证据。
3. **从单次分数走向可靠性**：tau-bench 的 pass^k 和 AndroidWorld 的任务变体强调稳定成功。
4. **从黑盒 judge 走向证据链**：OS-Themis 这类 milestone critic 把最终 verdict 绑定到可审计证据。
5. **从 accuracy 走向 cost-aware evaluation**：AI Agents That Matter 把成本、holdout、防过拟合和可复现性提到核心位置。
6. **从模型能力评估走向系统评估**：SWE-agent 说明 agent-computer interface 会改变结果，评估的其实是 model + harness + tools + policy 的组合。

#### 7.4.3 GDPval：经济价值知识工作的 Deliverable Eval

GDPval 是 OpenAI 在 2025 年提出的真实知识工作评估。它不是典型的 browser / OS / tool agent benchmark，因为当前版本主要是一轮交付物评估；但它对 Societas 这样的 information worker agent 非常关键，因为它把评估目标从“答题”推进到“产出真实工作成果”。

GDPval 的关键设计：

| 设计点 | 内容 | 对 Agent Eval 的意义 |
|---|---|---|
| 职业覆盖 | 44 个知识工作职业，来自对美国 GDP 贡献最高的 9 个行业 | 评估集应按真实经济活动选任务，而不是只按模型容易做的题目选任务 |
| 任务来源 | 1,320 个专业任务，gold open-sourced set 包含 220 个任务 | 可同时建设 full eval、公开 gold set、内部 held-out set |
| 交付物形态 | documents、slides、diagrams、spreadsheets、multimedia 等 | 和 Office artifact / business deliverable 强相关，不只是文本回答 |
| 专家构造 | 任务由平均 14 年经验的行业专家设计和审核 | 高质量 eval 需要 domain expert 参与，而不是完全 synthetic generation |
| 专家盲评 | 同职业专家盲评 AI 和人类交付物，判断 better / as good as / worse | 开放工作成果适合 pairwise / preference / rubric 评估，而不是 exact match |
| Rubric | 任务作者提供详细评分标准 | LLM judge 或自动 grader 必须以专家 rubric 为锚 |
| 局限 | 当前版本偏 one-shot，不覆盖多轮澄清、上下文积累、多次修改 | 对 agent 来说，GDPval 需要扩展成 interactive / revision / workflow 版本 |

GDPval 给 Societas 的直接启发：

1. **任务集应该按职业和行业组织**：例如销售、财务分析、项目管理、法律、医疗管理、制造工程、媒体编辑，而不是泛泛地收集“写 PPT”任务。
2. **评价对象应该是真实 deliverable**：PPT、Excel、Word、PDF、邮件草稿、CRM 更新记录、研究 brief、项目计划，而不是聊天答案。
3. **验收要结合程序检查和专家偏好**：文件结构、格式、公式、引用可以自动验；内容质量、判断力、行业可用性需要 expert rubric / LLM judge / 人工校准。
4. **要记录和人类专家的差距**：不是只看 agent 是否完成，而是看它的交付物是否达到 junior / senior / expert 水平。
5. **下一步应补 interactive GDPval-style tasks**：真实 coworker agent 需要澄清需求、读上下文、迭代修改、等待审批，这些超出 GDPval 当前 one-shot 形式。

因此，在本报告框架里，GDPval 应该归类为：**economically valuable deliverable evaluation**。它补上的不是 GUI 操作能力，而是“agent 产出的工作成果是否真的有职业价值”。

#### 7.4.4 与 GDPval 相似的 Eval 方法：三层地图

GDPval 不是孤立方向。围绕真实经济价值、专业工作和 workplace automation，相关 eval 可以分成三层。

第一层是 **GDPval-like deliverable eval**：评估 AI 是否能产出真实职业交付物，并用专家、rubric 或真实经济价值来判断质量。

| Benchmark | 共同点 | 差异 |
|---|---|---|
| GDPval | 跨行业、跨职业、真实工作交付物、专家盲评 | 当前偏 one-shot，不强调多轮 workflow |
| SWE-Lancer | 真实 freelance software engineering task，任务有真实 payout | 经济价值 grounding 很强，但只覆盖软件工程 |
| MLE-bench | 真实 ML engineering 任务，训练模型、处理数据、提交结果 | 垂直专业任务，评价更偏 objective metric |
| PaperBench | 复现真实 AI 论文，专家/作者参与 rubric 构建 | 更偏 AI research，不覆盖一般知识工作 |

第二层是 **workplace / workflow eval**：评估 agent 是否能在一个工作环境中连续推进任务，而不只是交付一个文件。

| Benchmark | 评估重点 | 与 GDPval 的互补关系 |
|---|---|---|
| TheAgentCompany | 模拟小公司环境，agent 浏览网页、写代码、运行程序、和同事沟通 | 补上 workplace context、协作和长程任务 |
| WorkArena / BrowserGym | 企业 SaaS / ServiceNow 中的知识工作流程 | 补上真实办公系统的 browser 操作和状态验证 |
| tau-bench | 用户对话、domain policy、API 工具和最终数据库状态 | 补上用户交互、规则遵循和 state-based verification |
| AppWorld | 多 app/API 任务、状态单测、意外副作用检测 | 补上多工具工作流和 collateral damage 检查 |

第三层是 **general assistant / web / OS eval**：评估 agent 的基础操作能力、网页信息搜集和真实环境适应性。

| Benchmark | 评估重点 | 与 GDPval 的关系 |
|---|---|---|
| GAIA | 浏览、工具、多模态、推理的综合助手能力 | 更通用，经济价值和交付物质量弱一些 |
| AssistantBench | 真实耗时 open-web 信息任务 | 更接近助理信息搜集，不强调职业 deliverable |
| WebArena / VisualWebArena / WebLINX | web navigation、视觉网页、多轮网页交互 | 补上浏览器行动和网页 grounding |
| OSWorld / AndroidWorld | OS / mobile app 真实环境任务 | 补上 computer-use 和环境状态验证 |

综合关系可以写成：

```text
GDPval
= 真实职业任务 + 专家交付物 + 专家盲评 + 经济价值覆盖

SWE-Lancer / MLE-bench / PaperBench
= GDPval 的垂直专业版本

TheAgentCompany / WorkArena / tau-bench / AppWorld
= GDPval 的 workplace / workflow / state-change 版本

WebArena / OSWorld / AndroidWorld / AssistantBench
= GDPval 缺的操作环境、网页信息搜集和执行过程评估层
```

对 Societas 来说，最合理的结论不是只选 GDPval，而是构建一个混合 eval：

```text
GDPval-style deliverable task
+ Office artifact verifier
+ WorkArena-style SaaS workflow state check
+ tau-bench-style user/tool/policy interaction
+ AppWorld-style state tests and collateral damage checks
+ expert rubric / blind preference review
+ cost / latency / human correction time
+ revision loop evaluation
```

一句话：**Societas 的 eval 应该评“真实职业成果是否可用”，也要评“agent 是否在真实工具链里把工作可靠做完”。**

---

### 7.5 Agent Eval 的指标体系

#### 7.5.1 一级指标：任务完成

| 指标 | 说明 |
|---|---|
| task success rate | 任务最终完成率 |
| exact match / goal match | 最终答案或状态与 gold 是否一致 |
| pass@1 | 单次尝试成功率 |
| pass@k | 多次尝试中至少一次成功 |
| pass^k / consistency | 多次运行持续成功的可靠性 |
| partial milestone score | 长任务 checkpoint 完成度 |

#### 7.5.2 二级指标：过程质量

| 指标 | 说明 |
|---|---|
| tool call validity | 工具/API 调用参数是否合法 |
| invalid action rate | 无效点击、错误命令、幻觉工具调用比例 |
| recovery rate | 出错后恢复能力 |
| loop / thrashing rate | 反复尝试无效路径的比例 |
| plan adherence | 执行是否符合计划，或是否合理更新计划 |
| evidence completeness | 是否留下足够证据支持最终结论 |
| human intervention count | 需要人工介入次数 |

#### 7.5.3 三级指标：业务与工程成本

| 指标 | 说明 |
|---|---|
| token cost | 模型 token 成本 |
| tool/API cost | 外部工具成本 |
| latency | 端到端耗时 |
| environment cost | 浏览器、VM、沙箱、reset 的成本 |
| cost per success | 每个成功任务的成本 |
| cost per improvement | 每提升 1 个点所需成本 |

#### 7.5.4 四级指标：安全与治理

| 指标 | 说明 |
|---|---|
| unsafe action rate | 越权、误删、误发、误提交等比例 |
| policy violation rate | 违反组织 policy 的比例 |
| sensitive data exposure | 敏感信息泄露或过度读取 |
| approval compliance | 高风险动作是否等待确认 |
| rollback success | 出错后是否可回滚 |
| audit completeness | 日志是否足够复盘 |

#### 7.5.5 五级指标：长期演进

| 指标 | 说明 |
|---|---|
| held-out improvement | 新版本在未见任务上的提升 |
| regression rate | 旧任务退化比例 |
| safety drift | 能力提升后安全风险是否上升 |
| memory utility | memory 是否真的提升任务成功率 |
| skill reuse rate | skill 被复用并成功的比例 |
| evaluator drift | evaluator 自身是否偏移 |
| archive diversity | 多方案 archive 是否保持多样性 |

---

### 7.6 推荐的 Agent Eval Harness 架构

#### 7.6.1 核心组件

一个可落地的 agent eval harness 至少包含：

```text
Task Registry
Environment Manager
Tool Sandbox
Run Orchestrator
Trace Logger
Artifact Store
Verifier Suite
LLM Judge Service
Human Review Queue
Metric Aggregator
Regression Dashboard
Failure Taxonomy Store
```

#### 7.6.2 Task Spec Schema

建议每个任务都结构化描述：

```yaml
task_id: crm_update_001
task_family: crm_followup
risk_level: medium
initial_state:
  environment: service_now_dev_instance
  fixture_id: customer_case_2026_001
goal_state:
  customer_status: follow_up_scheduled
  note_contains:
    - next meeting date
    - decision maker name
allowed_tools:
  - browser.read
  - browser.write_internal
  - file.create_draft
forbidden_actions:
  - email.send_external
  - record.delete
budget:
  max_steps: 80
  max_cost_usd: 1.00
evidence_required:
  - final_url
  - field_diff
  - trace_log
  - verifier_result
verifiers:
  - crm_state_check
  - policy_check
  - artifact_quality_judge
```

#### 7.6.3 Trace Event Schema

每一步都应记录为事件：

```json
{
  "run_id": "run_2026_0609_001",
  "step": 12,
  "event_type": "tool_call",
  "tool": "browser.click",
  "input_summary": "clicked Save on customer record page",
  "output_summary": "save succeeded, toast shown",
  "environment_state_hash": "...",
  "cost": {
    "tokens": 560,
    "latency_ms": 900
  },
  "risk": "internal_state_change",
  "evidence_ref": "screenshot_0012.png"
}
```

关键点：不要只保存最终结果，要保存足够复盘的信息。Self-evolving agent 的 memory、skill 和 workflow 更新都应该来自这种 raw trace，而不是来自模型事后回忆。

#### 7.6.4 Verifier Hierarchy

建议按可信度排序：

```text
1. Deterministic programmatic verifier
2. Environment state checker
3. Artifact parser / structural validator
4. Milestone evidence checker
5. LLM judge with rubric and evidence
6. Human review
```

这不是说 human review 最不可信，而是说在工程自动化链路里，越前面的 verifier 越适合做大规模、低成本、可重复检查；human review 应该用于高风险、模糊和校准场景。

#### 7.6.5 Evaluation Run Protocol

一次标准 eval run 应该包含：

1. 固定 agent version、model version、tool version、prompt version、memory snapshot。
2. 初始化环境和 task fixture。
3. 运行 agent，记录完整 trace。
4. 执行 deterministic verifier。
5. 执行 milestone verifier。
6. 对开放部分运行 LLM judge。
7. 对高风险或分歧样本进入 human review。
8. 聚合 success、cost、latency、safety、regression 指标。
9. 保存失败样本和 failure attribution。
10. 生成报告并更新 regression set 候选。

---

### 7.7 针对不同 Agent 类型的 Eval 方案

#### 7.7.1 Coding Agent

推荐评估：

- issue reproduction test
- unit / integration tests
- static analysis / lint / typecheck
- patch minimality
- no unrelated changes
- security scan
- regression tests
- cost per resolved issue
- reviewer acceptance rate

参考：SWE-bench、SWE-bench Verified、SWE-agent。

关键风险：

- 只让测试通过但没修根因。
- 修改测试绕过失败。
- 产生过大 patch。
- 引入安全漏洞。
- 对 repo 结构理解错误。

#### 7.7.2 Browser / Web Agent

推荐评估：

- final URL / DOM / database state
- form field values
- downloaded artifact correctness
- action trace replay
- milestone completion
- prompt injection handling
- external send / submit gate
- latency and browser step count

参考：WebArena、VisualWebArena、WebLINX、WorkArena、BrowserGym。

关键风险：

- 误点、误提交。
- 页面视觉 grounding 错误。
- 受网页 prompt injection 影响。
- 任务完成但证据不足。
- 对动态页面状态不鲁棒。

#### 7.7.3 OS / GUI Agent

推荐评估：

- execution-based evaluation script
- file system diff
- app state check
- screenshot + OCR / visual verifier
- cross-app workflow completion
- rollback check
- permission boundary check

参考：OSWorld、AndroidWorld、OS-Themis。

关键风险：

- GUI grounding 错误。
- 不可逆系统变更。
- 环境污染导致下一次评测失真。
- reward signal 不准确。

#### 7.7.4 Tool / API Agent

推荐评估：

- API selection accuracy
- parameter correctness
- call ordering
- idempotency
- side-effect safety
- final database state
- domain policy compliance
- simulated user satisfaction

参考：ToolBench / ToolEval、tau-bench。

关键风险：

- 工具选对但参数错。
- 忽略 domain policy。
- 重复调用造成状态污染。
- 用错误 API 达到表面结果。

#### 7.7.5 Office / Document Agent

这是 Societas 1.0 最相关的类型。

推荐评估：

| 对象 | Verifier |
|---|---|
| PPT | 文件可打开、页数、母版/模板、标题层级、图表/图片存在、空白占位、备注、导出 PDF 是否正常 |
| Excel | workbook 可打开、sheet schema、公式可计算、图表引用范围、数据类型、条件格式、隐藏错误值 |
| Word | docx 可打开、标题层级、目录、字数、引用、表格、页眉页脚、格式一致性 |
| 内容事实 | claim-source 对齐、引用链接可访问、关键事实覆盖 |
| 用户修改 | 局部 diff、版本记录、不破坏已确认内容 |

建议指标：

- artifact open rate
- structural pass rate
- content completeness
- source grounding score
- revision success rate
- format regression rate
- user edit distance after generation

#### 7.7.6 Research / Analyst Agent

推荐评估：

- source coverage
- citation validity
- factual consistency
- claim-level evidence mapping
- contradiction detection
- novelty / usefulness judge
- human expert review sample
- hallucination rate

注意：研究类任务很难全靠 programmatic verifier，需要 LLM judge + evidence + human calibration。

#### 7.7.7 Multi-agent System

推荐评估：

- team success vs single-agent baseline
- role contribution ablation
- critique usefulness
- conflict resolution quality
- communication cost
- redundant work rate
- consensus failure rate
- external verifier agreement

关键问题：multi-agent 很容易制造“看起来有协作”的复杂流程，但实际只是增加成本。必须做 ablation：去掉 reviewer、planner、critic 后是否真的下降？如果没有下降，该角色可能只是噪声。

#### 7.7.8 Self-evolving Agent

Self-evolving agent 的评估不能只看单轮分数，而要看演进过程。

推荐曲线：

```text
success rate over evolution rounds
held-out improvement
training-heldout generalization gap
regression rate
safety drift
cost per improvement
memory utility over time
skill reuse success rate
workflow update acceptance rate
evaluator drift
rollback frequency
```

每次 evolution 必须记录：

- 改了什么：memory、skill、workflow、tool policy、prompt、verifier、code。
- 为什么改：来自哪些 trace 和 failure attribution。
- 影响范围：哪些 task family 受影响。
- 验证结果：train、probe、held-out、regression、safety。
- 上线策略：candidate、shadow、gray、merge、rollback。

风险：

- reward hacking
- benchmark overfit
- memory pollution
- evaluator drift
- safety drift
- local improvement but global regression

---

### 7.8 Societas 场景下的 Eval-first 路线建议

#### 7.8.1 Societas 1.0：文档生成器 Eval

目标：证明 agent 能稳定产出可用 Office artifacts。

优先建设：

1. Artifact plan verifier：检查 PPT 大纲、Excel schema、Word 结构是否完整。
2. File integrity verifier：检查 pptx/xlsx/docx 是否能打开和解析。
3. Structural verifier：页数、章节、sheet、表格、公式、图表、引用。
4. Content verifier：关键 facts 是否覆盖，是否有 unsupported claims。
5. Revision verifier：局部修改是否完成，是否破坏已确认内容。
6. Trace log：记录素材、计划、生成参数、版本、验收结果。
7. GDPval-style task set：按真实职业和行业构造 PPT / Excel / Word / PDF 交付物任务，并用专家 rubric 或人工校准的 LLM judge 评价职业可用性。

建议 MVP 指标：

```text
artifact open rate >= 99%
required structure pass rate >= 95%
critical content omission <= 5%
revision success rate >= 85%
format regression after revision <= 3%
average human correction time下降
```

#### 7.8.2 Societas Coworker / Workflow Eval

目标：证明 agent 能在真实工具链中推进任务。

优先建设：

1. Browser/SaaS state verifier。
2. Workflow milestone verifier。
3. Human approval gate。
4. Tool-call audit。
5. Cost / latency dashboard。
6. Prompt injection and permission tests。

典型 task family：

- 研究摘要并生成 brief。
- 更新 CRM 记录。
- 生成项目周报。
- 收集竞品信息并填入表格。
- 从 SaaS 下载材料并整理成 PPT。

每个 task family 都要有：

```text
task spec
initial fixture
goal state
allowed tools
forbidden actions
milestone list
programmatic verifiers
LLM judge rubric for open-ended parts
human approval rule
regression examples
```

#### 7.8.3 Societas 3.0：Self-evolution Eval

目标：证明 agent 能越用越好，但不越权、不退化。

低风险可进化对象：

- memory
- failure pattern
- skill card
- workflow checklist
- tool policy hint
- evaluator notes

高风险对象：

- tool permissions
- external side-effect policy
- verifier gold labels
- production workflow merge
- code-level self-modification

建议 gate：

```text
Candidate update
  -> linked source traces
  -> typed change review
  -> probe validation
  -> held-out validation
  -> regression suite
  -> safety canary
  -> human approval if risk >= medium
  -> shadow mode
  -> merge or rollback
```

最重要的不是让 agent 自动改一切，而是让它能提出可审计、可验证、可回滚的改进候选。

#### 7.8.4 综合结论：Societas Eval 应该像 GDPval + WorkArena + tau-bench + Office Verifier

Societas 的目标不是单纯回答问题，也不是只操作网页，而是帮助 information worker 完成真实工作。因此评估体系应同时覆盖四件事：

| 组成 | Societas 应借鉴什么 | 落地形态 |
|---|---|---|
| GDPval | 真实职业任务、专家交付物、专家 rubric、盲评偏好 | 按行业/职业构造 PPT、Excel、Word、PDF、brief、邮件、项目计划等任务 |
| WorkArena / TheAgentCompany | workplace context、SaaS、公司环境、长程任务 | 浏览器/SaaS workflow、真实系统状态、任务阶段 checkpoint |
| tau-bench / AppWorld | 用户交互、工具调用、domain policy、最终数据库/应用状态 | API/tool trace、policy gate、state-based verifier、collateral damage 检查 |
| Office verifier | 文件可打开、结构正确、公式/引用/格式稳定 | pptx/xlsx/docx parser、schema check、revision diff、artifact regression |

最终评价不要只给一个 accuracy，而应报告：

```text
deliverable expert-quality score
artifact structural pass rate
workflow state success rate
policy / approval compliance
revision success rate
human correction time
cost and latency
regression on old task families
```

这也是 Societas 与普通 agent benchmark 的差异：它要证明的不只是“模型会不会做题”，而是“agent 能不能以可控成本稳定交付真实 information work”。

#### 7.8.5 基于 Mid+Heavy User Samples 的 Societas-GDPval 任务设计

Mid+Heavy 用户样本进一步说明：Societas 的核心用户不是轻量聊天用户，而是把系统当成专业文件生产系统，在 1-6 天内围绕一个项目或一个职业场景连续产出 PPT / Word 文件的人。

这些样本有三个共同特征：

1. **专业领域强**：临床 OT、内分泌医学教育、MARPOL 海事合规、BIM 工程、破产清算法律、财务分析、NGO 儿童安全、脑卒中康复、LLM gateway 技术规格。
2. **项目连续性强**：很多用户不是一次生成一个文件，而是在同一项目中反复迭代，或在一周内生成同一课程 / 案件 / 公司文档体系的一组文件。
3. **产物可交付性强**：输出不是闲聊文本，而是课程 PPT、客户提案、法律 Word、培训材料、技术规格、研究方案、公司合同 / SLA / 服务目录等真实 deliverables。

因此，Societas-GDPval 应该从这些真实样本抽象出 8 个 eval suites：

| Suite | 用户样本来源 | 典型任务 | 关键评估点 |
|---|---|---|---|
| Clinical / Medical Education PPT | JP OT 临床神经发育课程、MX 内分泌教学 | 生成一节临床课程 PPT，或一组相关主题课程 | 医学准确性、教学结构、术语一致性、临床风险、系列课程连贯性 |
| Education / Training PPT | BR 地理环境教育、CD NGO 儿童安全培训、US 客服培训 | 批量生成课程、quiz、answer key、facilitator guide | 学习目标、难度递进、讲师可用性、知识点覆盖、评估题与内容一致 |
| Business / Consulting Deck | SG AI 投资 pitch、VN 数字钱包业务案例、PT BIM 工程销售 | 生成 VC pitch、业务案例、服务提案、外包销售 deck | 商业逻辑、客户定位、价值主张、scope / pricing 一致性、presentation readiness |
| Company Document Pack | KE 初创顾问全栈商业文档体系 | 生成 NDA、合同、SLA、服务目录、客户提案等一组文档 | 文档集完整性、跨文档一致性、法律/商业风险、品牌语气、可直接发送程度 |
| Legal / Formal Word Pack | CH 意大利语破产清算案件 | 生成债权人名单、附件、税务回函、更正版等案件文件 | 人名/金额/日期一致、附件引用、版本安全、法律结构、不可编造事实 |
| Technical / Research Word | GB LLM Gateway 规格、BR 脑卒中康复研究、PT BIM 报告 | 生成技术规格、工程报告、研究方案 | 结构完整性、假设清晰、引用 grounding、表格/图示正确、可审阅性 |
| Finance / Analysis PPT | KR Asan 财务分析、FR 生物力学/会议材料 | 生成公司财务分析或研究展示 | 数据一致性、图表解释、结论支撑、会议/投资语境适配 |
| Creative / Long-form Word | US Lost Marauder、Paths of Sozo、AI prompts / storyboard SOP | 生成长篇创作、神学章节、创意 SOP | 风格一致、章节连续、角色/概念 continuity、用户偏好学习 |

其中，PPT 和 Word 的重度用户应分别形成不同的评估重点：

| Artifact 类型 | 重度样本显示的真实用法 | Eval 重点 |
|---|---|---|
| PPT | 专业课程、销售提案、财务分析、旅行/教育内容，常见多主题批量生产 | deck structure、visual consistency、topic coverage、series-level coherence、revision burden |
| Word | 法律案件、公司文档体系、培训材料、技术规格、研究方案、长篇写作 | document integrity、section hierarchy、cross-document references、version safety、factual consistency |

##### MVP Task Set 建议

不要一开始复制 20 个用户的全部产物。第一版只需要 16-24 个高质量任务，每个任务有 source material、accepted output、revision history 和 rubric。

建议第一版抽样：

| Suite | MVP 任务数 | 抽样原则 |
|---|---:|---|
| Clinical / Medical Education PPT | 4 | 覆盖 OT 神经发育、内分泌、电解质/代谢、医学课程结构 |
| Business / Consulting Deck | 3 | 覆盖 AI 投资 pitch、数字钱包业务案例、BIM 服务提案 |
| Company Document Pack | 3 | 覆盖 NDA、SLA、服务目录、客户提案的跨文档一致性 |
| Legal / Formal Word Pack | 3 | 覆盖债权人名单、附件、回函、更正版，重点测版本安全 |
| Technical / Research Word | 3 | 覆盖 LLM 技术规格、BIM 工程报告、脑卒中康复研究 |
| Education / Training Pack | 3 | 覆盖 NGO 儿童安全、客服培训、地理/环境教育 |
| Finance / Analysis PPT | 2 | 覆盖企业财务分析和图表解释 |
| Creative / Long-form Word | 2 | 覆盖长篇风格连续性和用户偏好 |

第一版合计约 23 个任务，足够形成一个小而硬的 Societas-GDPval seed set。

##### 每个任务的标准封装

每个真实样本应被整理成统一 task package：

```yaml
task_id: societas_gdpval_clinical_ppt_001
suite: clinical_medical_education_ppt
artifact_type: pptx
risk_level: medium
user_segment: heavy_ppt_professional_course_creator
project_context:
  domain: occupational_therapy
  language: ja / en mixed if applicable
  project_span: multi-topic course series
input:
  source_materials: [...]
  user_prompt_or_task_brief: ...
  prior_accepted_artifacts: [...]
expected_output:
  file_type: pptx
  required_sections: [...]
  must_include_facts: [...]
  forbidden_claims: [...]
verifiers:
  - office_integrity_check
  - structure_check
  - placeholder_check
  - reference_or_fact_check
  - project_consistency_check
rubric:
  - professional_correctness
  - completeness
  - ready_to_use
  - style_fit
  - revision_burden
online_signals_to_track:
  - download_or_export
  - return_to_project
  - regenerate_same_artifact
  - repeated_correction
  - abandoned_after_generation
```

##### 这些样本暴露出的新增指标

Mid+Heavy 样本说明，仅看 completion rate 没有意义，因为样本里很多任务都是 100% complete。Societas eval 应该新增这些更贴近真实价值的指标：

| 指标 | 为什么重要 |
|---|---|
| Project-pack completion | 用户常常要的是一组文件，而不是单文件 |
| Cross-artifact consistency | NDA / SLA / 提案、课程系列、案件附件之间不能互相矛盾 |
| Repeat correction rate | 重度用户反复迭代，最能暴露系统是否学会偏好 |
| Accepted-version preservation | 修改新版本时不能破坏已确认内容 |
| Domain-risk flag rate | 医疗、法律、合规、财务场景不能编造关键事实 |
| Human cleanup time | 用户真正关心从初稿到可用稿还要改多久 |
| Series coherence | 56 节课程、65 个房地产教育文件、整套公司文档体系都需要整体连贯 |
| Demo / anomaly quarantine | 类似单项目极高重复的疑似 demo 账号应单独标注，避免污染真实用户 eval |

##### 样本清洗规则

评估集建设时要保留真实高强度使用，但不能让异常样本误导 benchmark：

1. 已知 demo 账号直接排除。
2. 单项目极高重复、疑似演示账号进入 quarantine set，不进入核心 gold set。
3. 保留高迭代真实项目，因为 revision loop 是 Societas 的关键能力。
4. 中度用户用于覆盖 market / domain 多样性，重度用户用于构造 project-pack 和 repeat-correction eval。
5. 对医疗、法律、金融、儿童安全等高风险领域，rubric 必须显式加入 unsupported claim 和 risk omission 检查。

这些用户样本把 Societas 的 eval 定义得更清楚：**不是通用 task success benchmark，而是专业文件生产系统的 project-pack benchmark。**

---

### 7.9 常见陷阱与防护

| 陷阱 | 表现 | 防护 |
|---|---|---|
| 只看最终答案 | agent 说完成但实际文件坏了或状态没变 | state / artifact verifier |
| LLM judge 滥用 | judge 被流畅表达骗过 | deterministic checks first + evidence-grounded rubric |
| Benchmark overfit | 对公开任务越来越好，真实任务不提升 | held-out set、轮换集、隐藏集 |
| Cost ignored | 分数提升但成本暴涨 | cost-normalized score、budget gate |
| Environment contamination | 上一次运行污染下一次 | setup / tear-down / state hash |
| Trace 不完整 | 失败后无法复盘 | structured trace logging |
| Reward hacking | agent 利用 evaluator 漏洞 | evaluator red-team、external verifier、human audit |
| Safety drift | 自进化后更敢越权 | safety canary + approval policy |
| Regression ignored | 新能力破坏旧任务 | regression suite as merge gate |
| Human review 太晚 | 高风险动作已经发生 | pre-action approval gate |
| Human review 太多 | agent 变成遥控器 | risk-based review routing |
| 不区分模型和 harness | 分不清是模型强还是工具界面好 | ablation: model fixed, harness varied |

---

### 7.10 推荐评分框架

不建议用单一总分替代所有指标，但可以用一个 dashboard score 帮助比较版本。

示例：

```text
Hard gates:
  safety_violation == 0
  critical_regression <= threshold
  artifact_integrity_pass == true
  forbidden_action == 0

Weighted score after gates:
  45% task success
  15% milestone completion
  10% trajectory quality
  10% robustness / consistency
  10% cost efficiency
  10% user or judge quality score
```

但要注意：不同 agent 类型权重不同。

| Agent 类型 | 应该加权更高的指标 |
|---|---|
| Coding agent | tests、regression、patch quality |
| Browser agent | state success、trajectory、approval compliance |
| Office agent | artifact integrity、structure、revision stability |
| Research agent | evidence grounding、citation validity、human usefulness |
| Self-evolving agent | held-out improvement、regression、safety drift、cost per improvement |

---

### 7.11 建设路线图

#### Phase 1：最小可用 Eval Harness

目标：让 agent 结果可验收。

交付：

- task registry
- trace logger
- artifact store
- deterministic verifier for core artifacts
- simple dashboard

不要先追求复杂 judge，先解决“文件到底能不能用、状态到底有没有变”。

#### Phase 2：Trajectory + Milestone Eval

目标：让失败可归因。

交付：

- action / observation / tool call trace
- milestone schema
- evidence packet
- failure taxonomy
- replay or audit viewer

#### Phase 3：LLM Judge + Human Calibration

目标：覆盖开放质量判断。

交付：

- rubric library
- judge prompt versioning
- calibration set
- human review queue
- judge-human agreement report

#### Phase 4：Regression + Safety + Cost

目标：让 agent 能稳定迭代。

交付：

- held-out suite
- regression suite
- safety canary suite
- cost / latency dashboard
- merge gate

#### Phase 5：Self-evolution Evaluation

目标：让 agent 的改进可控。

交付：

- typed update proposal
- source trace linking
- probe validation
- shadow deployment
- rollback and audit
- evaluator drift monitoring

---

### 7.12 最小落地 Checklist

如果只做一版实用 eval system，建议先做到这 12 件事：

1. 每个任务都有 task_id、risk_level、initial_state、goal_state。
2. 每次运行都有 run_id、agent_version、model_version、tool_version。
3. 所有 tool call 和环境观察都进 trace log。
4. 所有 artifact 都存版本和 hash。
5. 每个核心任务至少有一个 deterministic verifier。
6. 长任务拆 milestone，每个 milestone 要 evidence。
7. 高风险动作前必须 human approval。
8. LLM judge 只评开放语义质量，且必须引用 evidence。
9. 报告 success rate 的同时报告 cost、latency、human intervention。
10. 每次 prompt / workflow / skill 改动都跑 regression。
11. 每个失败样本都有 failure taxonomy。
12. 每个 self-evolution update 都能追溯 source_run_id 并可 rollback。

---

### 7.13 关键参考来源

内部资料：

- From signals to satisfaction — closing the loop in Societas, 2026-05-11, https://closing-the-gap-review.pages.dev/
- 20260526 Societas eDAU vs Artifacts Divergence, https://microsoftapc-my.sharepoint.com/:u:/g/personal/alexyuan_microsoft_com/IQCFhohRlOsARrqC34hkMXvGAS3cbjykyePFfP8bMWNPqCY?e=I4BWRw
- Mid-Heavy User Samples, https://microsoftapc-my.sharepoint.com/:u:/g/personal/alexyuan_microsoft_com/IQAvHzSgiAjWSqZFsWKYl8O6AeB48lZ4c-LyYwSTsxfkIuk?e=C9Qljc

行业资料：

- AgentBench: Evaluating LLMs as Agents, 2023/2025, https://arxiv.org/abs/2308.03688
- WebArena: A Realistic Web Environment for Building Autonomous Agents, 2023/2024, https://arxiv.org/abs/2307.13854
- SWE-bench: Can Language Models Resolve Real-World GitHub Issues?, 2023/2024, https://arxiv.org/abs/2310.06770
- GAIA: a benchmark for General AI Assistants, 2023, https://arxiv.org/abs/2311.12983
- VisualWebArena: Evaluating Multimodal Agents on Realistic Visual Web Tasks, 2024, https://arxiv.org/abs/2401.13649
- WebLINX: Real-World Website Navigation with Multi-Turn Dialogue, 2024, https://arxiv.org/abs/2402.05930
- WorkArena: How Capable Are Web Agents at Solving Common Knowledge Work Tasks?, 2024, https://arxiv.org/abs/2403.07718
- GDPval: Measuring the performance of models on economically valuable real-world tasks, 2025, https://openai.com/index/gdpval/；paper: https://arxiv.org/abs/2510.04374
- SWE-Lancer: Can Frontier LLMs Earn $1 Million from Real-World Freelance Software Engineering?, 2025, https://openai.com/index/swe-lancer/；paper: https://arxiv.org/abs/2502.12115
- MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering, 2024, https://openai.com/index/mle-bench/；paper: https://arxiv.org/abs/2410.07095
- PaperBench: Evaluating AI's Ability to Replicate AI Research, 2025, https://openai.com/index/paperbench/；paper: https://arxiv.org/abs/2504.01848
- TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks, 2024/2025, https://arxiv.org/abs/2412.14161
- AppWorld: A Controllable World of Apps and People for Benchmarking Interactive Coding Agents, 2024, https://arxiv.org/abs/2407.18901
- AssistantBench: Can Web Agents Solve Realistic and Time-Consuming Tasks?, 2024, https://arxiv.org/abs/2407.15711
- OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments, 2024, https://arxiv.org/abs/2404.07972
- AndroidWorld: A Dynamic Benchmarking Environment for Autonomous Agents, 2024/2025, https://arxiv.org/abs/2405.14573
- tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains, 2024, https://arxiv.org/abs/2406.12045
- ToolLLM / ToolBench / ToolEval, 2023, https://arxiv.org/abs/2307.16789
- Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena, 2023, https://arxiv.org/abs/2306.05685
- AI Agents That Matter, 2024, https://arxiv.org/abs/2407.01502
- SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering, 2024, https://arxiv.org/abs/2405.15793
- OS-Themis: A Scalable Critic Framework for Generalist GUI Rewards, 2026, https://arxiv.org/abs/2603.19191
- ANCHOR: Towards Healthy Evolution, 2026, https://arxiv.org/abs/2606.06114

---

### 7.14 Bottom Line

当前 agent evaluation 的最佳实践不是单一 benchmark、单一 judge 或单一成功率，而是一套分层系统：

```text
deterministic verifier for what can be checked
environment state for what can be observed
milestone evidence for long-horizon tasks
LLM judge for open-ended semantic quality
human review for high-risk ambiguity
cost / latency / reliability for production usefulness
regression / safety checks for iteration
```

对 Societas 来说，最务实的路线是：先把文档和 workflow 的 verifier 做扎实，再做 trajectory log 和 milestone evidence，最后把 memory、skill、workflow 的 self-evolution 接到 held-out、regression 和 safety gate 上。

真正可用的 agent，不是说“我完成了”，而是能拿出证据说明：目标已完成、过程可审计、风险被控制、成本可接受、下次不会变差。
