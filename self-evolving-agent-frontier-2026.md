# Self-Evolving Agent 前沿调研框架

> 时间范围：重点看 2024-2026，少量 2023 工作仅作为脉络背景。
> 调研日期：2026-06-08。
> 核心问题：一个 agent 系统里哪些部分可以 self-evolve？每一层目前最好的进化范式是什么？还缺哪些关键问题？

阅读注意：2026 年论文多数仍很新，很多尚未经过长期复现和工程验证。本文把它们作为“前沿方向信号”和框架设计素材，而不是把所有实验结论都当成稳定事实。

---

## 0. 一句话结论

2024-2026 的 self-evolving agent 研究，已经从早期的“让 LLM 反思并写一段记忆”升级为“让 agent 系统的可变层被显式建模、被轨迹证据驱动、被独立 evaluator 验证、并在 archive / population / harness 中持续搜索”。

最前沿的方向不是单点优化 system prompt，而是把 agent 拆成若干可进化对象：

1. **Policy / System Prompt**：任务策略、推理准则、rubric、meta-prompt。
2. **Memory / Experience**：原始轨迹、反思、经验树、失败模式、可检索案例。
3. **Skills**：可复用过程、代码片段、工作流模板、操作例程。
4. **Tools**：工具包装、工具组合、工具选择、工具接口修复。
5. **Workflow / Agentic Program**：节点、边、调用顺序、分支、review loop。
6. **Harness**：执行环境、上下文管理、工具接口、生命周期、观测、验证、治理。
7. **Evaluator / Reward / Verifier**：自动评审、测试、critic、rubric、benchmark、virtual task。
8. **Population / Multi-agent Topology**：角色分工、团队选择、知识流、fork/merge/prune。
9. **Code / Architecture**：agent 自己的代码、编辑工具、上下文窗口策略、review 机制。
10. **Model Parameters / Fast Weights**：LoRA/online adaptation/parametric memory，仍处早期但开始进入 self-evolving agent 主线。

如果要给当前最佳范式一个统一表达，可以写成：

```text
instrumented trajectories
  -> diagnose / attribute failures
  -> propose bounded edits to one or more evolvable objects
  -> validate with held-out tasks, counterfactual probes, regression tests, safety checks
  -> archive successful variants and raw evidence
  -> retrieve / recombine / specialize future variants
```

关键趋势：**从 free-form reflection 走向 typed, scoped, testable evolution**。也就是不要让 agent 随便“总结一下经验”，而是让它在有类型的空间里改 prompt、skill、tool、workflow、harness 或代码，并且每次修改都必须可回放、可归因、可回滚、可评估。

---

## 1. 近两年前沿论文地图

### 1.1 系统/架构级自进化

| 方向 | 代表工作 | 进化对象 | 主要范式 | 关键启发 |
|---|---|---|---|---|
| Agentic system 自动设计 | ADAS / Meta Agent Search, 2024, https://arxiv.org/abs/2408.08435 | agent 代码、prompt、workflow、工具组合 | meta-agent 编写新 agent 代码，维护 archive | agent 不只是被 prompt 调，agent 本身可以作为程序被搜索 |
| Workflow 自动生成 | AFlow, 2024-2025, https://arxiv.org/abs/2410.10762 | code-represented workflow：节点与边 | MCTS + 代码修改 + 执行反馈 | 把 workflow 优化视为代码搜索，而不是手写流程图 |
| 自修改代码 agent | Darwin Godel Machine, 2025-2026, https://arxiv.org/abs/2505.22954 | coding agent 的代码、工具、上下文管理、peer review 机制 | open-ended archive + foundation model 生成变体 + coding benchmark 验证 | 目前最像“真正 self-improving agent”的路线：agent 改自己的实现 |
| Agentic evolution harness | AEvo / Harnessing Agentic Evolution, 2026, https://arxiv.org/abs/2605.13821 | 控制进化过程的 procedure / agent context | 把 evolution context 建模成交互环境，meta-agent 编辑进化机制 | 进化过程本身也需要 harness，否则长期搜索会漂移 |
| Harness flaw repair | HarnessFix, 2026, https://arxiv.org/abs/2606.06324 | harness 层：环境、工具接口、上下文、编排、观测、验证、治理 | trace IR + 失败归因 + scoped repair operator + regression validation | “agent 不行”经常其实是 harness 坏了，必须把 harness 作为可进化对象 |

### 1.2 Skill / Memory / Tool 级自进化

| 方向 | 代表工作 | 进化对象 | 主要范式 | 关键启发 |
|---|---|---|---|---|
| Workflow memory | Agent Workflow Memory, 2024, https://arxiv.org/abs/2409.07429 | 常用任务流程/例程 | 从历史轨迹归纳 reusable workflows，按任务检索注入 | 长程 web 任务里，“流程记忆”比普通摘要更有用 |
| Skill 开放世界自举 | OpenSkill, 2026, https://arxiv.org/abs/2606.06741 | skill + verifier anchors | 从文档、仓库、web 获取知识，生成 virtual tasks，自建验证信号 | 真部署时没有 gold answer，agent 必须先造练习场和验收器 |
| Skill 层级化整合 | SkillPyramid, 2026, https://arxiv.org/abs/2606.03692 | hierarchical skill topology | compose / validate / incorporate 新 skill | skill bank 不能只是平铺列表，需要拓扑、复用、泛化 |
| Skill-tool 协同进化 | SkillSmith, 2026, https://arxiv.org/abs/2606.01314 | skill 与 tool 同时进化 | reflection 生成 skill-tool atomic bundle，估计互补/冲突，记录 anti-pattern | skill 失败可能来自 tool 层，二者要一起改 |
| 记忆退化风险 | Useful Memories Become Faulty, 2026, https://arxiv.org/abs/2605.12978 | episodic traces 与 consolidated memory | 对比保留原始轨迹、强制总结、可控 retain/delete/consolidate | 持续 LLM 总结可能把有用记忆改坏；raw episode 必须是一等公民 |
| 经验结构化 | Tree-of-Experience, 2026, https://arxiv.org/abs/2606.06960 | experience tree | 组织、检索、验证、更新经验 | 低重复、隐式奖励环境中，经验管理比“多记一点”更关键 |

### 1.3 Multi-agent / Population 级自进化

| 方向 | 代表工作 | 进化对象 | 主要范式 | 关键启发 |
|---|---|---|---|---|
| 自动生成 multi-agent | EvoAgent, 2024-2025, https://arxiv.org/abs/2406.14228 | 多 agent 配置、角色、设置 | mutation / crossover / selection | 从单 agent 扩展到多 agent，可以用进化算法搜索团队结构 |
| 开放式多 agent 发现 | CORAL, 2026, https://arxiv.org/abs/2604.01658 | long-running agents、shared persistent memory、协作机制 | 异步多 agent + 共享记忆 + heartbeat intervention + sandbox | 长期 open-ended search 需要 agent 自治，也需要资源、安全和会话治理 |
| 测试时多 agent 共进化 | EVOCHAMBER, 2026, https://arxiv.org/abs/2605.11136 | individual memory、team structure、population lifecycle、knowledge flow | post-task collaborative dreaming，强 agent 向弱 agent 定向传递 niche knowledge，fork/merge/prune/seed | 多 agent evolution 不等于 N 个单 agent；核心是协作拓扑和知识流也在进化 |
| 社会化进化评估 | SAGE, 2026, https://arxiv.org/abs/2606.03544 | peer history 与 group learning | SelfEvo vs SocialEvo compute-matched 对照 | 共享历史不是总有用，filtered summaries 往往优于 raw logs |

### 1.4 Eval / Reward / Scientific loop

| 方向 | 代表工作 | 进化对象 | 主要范式 | 关键启发 |
|---|---|---|---|---|
| 自动科研闭环 | The AI Scientist, 2024, https://arxiv.org/abs/2408.06292 | idea、code、experiment、paper、review | idea generation -> coding -> experiments -> paper -> simulated review | self-evolution 需要完整闭环，不只是 task solver |
| Agent-computer interface | SWE-agent, 2024, https://arxiv.org/abs/2405.15793 | agent-computer interface / ACI | 为 agent 设计专门工具界面 | 工具接口设计本身决定 agent 能力上限 |
| GUI reward critic | OS-Themis, 2026, https://arxiv.org/abs/2603.19191 | reward / critic framework | 分解轨迹为可验证 milestones，多 agent critic 审计证据链 | reward model 不是一句 judge prompt，而是证据链与里程碑验证 |
| Human-like oversight | ANCHOR, 2026, https://arxiv.org/abs/2606.06114 | oversight phase / intervention policy | 在 evolution 的不同阶段插入人类式监督 | 最有效干预点往往是 output verification，不是每一步都管 |

### 1.5 Prompt / Policy 进化的背景线

这些工作多在 2022-2024，但仍是后续 self-evolving agent 的基础模块：

| 工作 | 链接 | 贡献 |
|---|---|---|
| Automatic Prompt Engineer / APE | https://arxiv.org/abs/2211.01910 | 把 instruction 当 program，通过候选生成和评分函数选择 prompt |
| OPRO | https://arxiv.org/abs/2309.03409 | LLM 作为优化器，根据历史 solution-score 生成新 solution/prompt |
| PromptBreeder | https://arxiv.org/abs/2309.16797 | 不仅进化 task prompt，还进化 mutation prompt，带自指结构 |
| Reflexion | https://arxiv.org/abs/2303.11366 | verbal reinforcement：从失败中写反思并存入 episodic memory |
| Self-Refine | https://arxiv.org/abs/2303.17651 | generator / feedback / refiner 同模型迭代改输出 |
| Voyager | https://arxiv.org/abs/2305.16291 | automatic curriculum + executable skill library + environment feedback |

---

## 2. 一个可用的 self-evolving agent 总框架

可以把 self-evolving agent 拆成三层：

```text
A. Task-solving loop
   user/task -> plan -> act/tool -> observe -> produce output

B. Learning/evolution loop
   trajectory -> diagnose -> propose edit -> validate -> archive -> deploy/retrieve

C. Governance loop
   sandbox -> permissions -> audit log -> regression suite -> safety reviewer -> rollback
```

其中 B 层是“自进化”核心，但 B 层不能孤立存在。没有 A 层就没有经验，没有 C 层就会漂移、误学、越权或不可复现。

更具体一点，每轮 evolution 至少应包含：

1. **Evidence capture**：保存原始轨迹、工具调用、环境观察、错误、最终结果、成本、时间、人工反馈。
2. **Failure attribution**：判断失败来自 prompt、skill、tool、workflow、harness、verifier、memory、model 能力，还是外部环境。
3. **Typed proposal**：只允许在某个明确类型空间里生成候选修改，例如 skill edit、tool wrapper、workflow edge change、rubric rule edit。
4. **Validation**：在训练任务、相似 probe、held-out task、regression set、安全集上验证。
5. **Archive and selection**：保留成功变体，也保留失败证据；用 archive 做后续检索、重组和多样性维护。
6. **Deployment policy**：决定立即上线、灰度、只进入 skill bank、只进入 candidate pool，还是丢弃。
7. **Rollback and audit**：任何可进化对象都要有版本、来源、指标、适用范围、失效条件。

---

## 3. 各部分目前最佳进化范式

### 3.1 System Prompt / Policy / Rubric

**进化对象**：system prompt、developer instruction、任务准则、输出格式、打分 rubric、风险规则、meta-prompt。

**早期范式**：APE / OPRO / PromptBreeder 这类 prompt search。给定任务集和评分函数，让 LLM 生成候选 prompt，再用 benchmark 选择。

**2026 更优范式**：结构化、可归因、可审计的 policy edit。

最好不要让 agent 自由改整段 system prompt。更好的方式是把 policy 拆成：

```text
role constraints
allowed actions
decision rules
tool-selection rules
error-handling rules
verification checklist
safety / compliance rules
style / output contract
```

每次只改一个 rule 或一个小 bundle，并记录：为什么改、来自哪些失败轨迹、影响哪些任务、在哪些 regression 上通过。

**最佳实践**：

- 用 OPRO/PromptBreeder 类搜索做离线 prompt discovery，但线上部署要转成结构化 policy diff。
- 对低信噪比环境采用 SHARP 类 neuro-symbolic/rubric policy，而不是任意自然语言 prompt mutation。
- system prompt 应该分层：稳定宪法层、任务策略层、可进化经验层。只有后两层允许自动修改。
- prompt/policy 修改必须有 held-out validation，否则容易过拟合 benchmark 或近期任务。

**风险**：prompt 会发生 semantic drift。LLM 写的新规则看起来更聪明，但可能删除了关键约束、引入隐性目标、或者让模型在某类任务上过拟合。

---

### 3.2 Memory / Experience

**进化对象**：原始 episode、反思、错误模式、经验树、workflow trace、成功案例、失败案例、anti-pattern、检索索引。

**早期范式**：Reflexion 式“失败后写反思，下次塞回 prompt”。

**2026 更优范式**：raw episodes first + gated consolidation + structured experience management。

2026 的一个重要修正是：**持续总结不一定让 agent 更聪明，可能让记忆越来越假**。Useful Memories Become Faulty 显示，LLM 不断把轨迹压成抽象记忆时，记忆效用会先升后降，甚至低于无记忆基线。因此，最稳的 memory 设计是：

```text
raw trajectory store   不可覆盖证据
case index             可检索案例
reflection notes       可丢弃/可替换
consolidated lessons   受控生成，需要验证
anti-pattern bank      失败签名 + 原因 + remedy
experience graph/tree  组织适用范围、前置条件、冲突关系
```

**最佳实践**：

- 原始轨迹永久保留，摘要只是派生物。
- consolidation 不要每轮自动发生；应由触发条件控制，例如重复失败、相似任务达到阈值、或新经验能解释旧失败。
- 记忆要带适用范围：domain、task type、tool version、model version、环境状态。
- retrieval 不只按语义相似，还要按前置条件、成功率、近期有效性、冲突关系过滤。
- 对隐式奖励和延迟反馈场景，采用 Tree-of-Experience 这类结构化经验管理，而不是普通 RAG。

**风险**：错误记忆会比没有记忆更糟。尤其是金融、医疗、法律、代码等领域，错误抽象会被不断复用，造成系统性偏差。

---

### 3.3 Skills

**进化对象**：可执行代码技能、操作流程、任务模板、浏览器/GUI 例程、问题求解策略、领域方法。

**代表脉络**：Voyager 的 executable skill library 是早期标志；2026 的 OpenSkill、SkillPyramid、SkillMaster、SkillSmith 则把 skill 从“库”升级为“生态系统”。

**2026 最佳范式**：skill 不只是存储，而是经历 create / compose / validate / retire / specialize / generalize。

一个成熟 skill 应该至少有：

```text
name
intent
preconditions
inputs / outputs
procedure or code
required tools
validation probes
known failure modes
examples
version
usage stats
conflicts / complements
retirement criteria
```

**最佳实践**：

- 从轨迹中提炼 skill 时，必须同时生成 probe task 或 verifier，而不是只写自然语言描述。
- 用层级拓扑管理 skill：primitive skill -> composite skill -> workflow skill。
- skill 应能合成和拆分：过大的 skill 难验证，过小的 skill 难复用。
- 对 open-world 场景，先从文档、仓库、web 建立 verification anchors，再构造 virtual tasks 自练。
- skill bank 需要 retirement。长期系统里最大的敌人之一是过期技能和重复技能。

**风险**：skill 会互相冲突。一个 skill 单独评估有效，不代表和其他 skill 组合有效。SkillSmith 的启发是要估计 skill 间互补/冲突，并记录 anti-pattern。

---

### 3.4 Tools

**进化对象**：工具选择策略、工具 wrapper、API 参数、错误恢复、工具组合、工具拆分、工具 retirement。

传统 agent 研究常把 tools 当固定层，但 2026 的趋势是 skill 和 tool co-evolve。很多“skill 不好用”其实是工具接口不适配、返回格式不好、权限不足、错误信息不可诊断。

**最佳范式**：skill-tool atomic bundle。

也就是一次进化提案可以同时包含：

```text
new or edited skill
new tool wrapper / parameter policy
validation probes
expected interaction with existing skills
known anti-pattern vetoes
```

**最佳实践**：

- 工具返回必须结构化，便于 trace attribution。
- 每个 tool wrapper 有 contract tests。
- 对高风险工具，tool policy 独立于 skill policy，由 harness enforce。
- 工具选择不只看语义，也看历史成功率、成本、延迟、失败模式。
- 当多个 skill 反复绕过同一工具限制时，优先考虑修改 tool wrapper，而不是给每个 skill 打补丁。

**风险**：工具层一旦允许自动修改，就会带来权限和安全问题。工具进化必须在 sandbox 中进行，并且区分 proposal 权限和 deployment 权限。

---

### 3.5 Workflow / Agentic Program

**进化对象**：agentic workflow 的节点、边、分支、循环、reviewer、critic、tool-call 顺序、并行结构。

**代表工作**：AFlow 把 workflow 视为代码表示的搜索空间，用 MCTS 和执行反馈优化；ADAS 更进一步，让 meta-agent 发明 agentic system designs。

**最佳范式**：code-represented workflow + search + execution feedback。

相比手写流程图，代码表示有几个优点：可 diff、可测试、可执行、可组合、可被 agent 修改。

**最佳实践**：

- workflow 节点要 typed：planner、executor、critic、verifier、memory_retriever、tool_router 等。
- 每次修改要限制范围：新增一个 reviewer、调整一条边、改变某节点 prompt，而不是整图重写。
- 用 MCTS / evolutionary search 保留多样性，而不是贪心选当前最高分。
- workflow 的指标不能只有最终分数，还要有 token cost、latency、failure type、safety violation、human intervention rate。
- 小模型 + 好 workflow 有时能以低成本超过大模型 + 简单 workflow，这是 AFlow 类工作的重点信号。

**风险**：workflow 搜索非常容易 benchmark overfit。必须有 held-out suite 与 cross-domain transfer 检验。

---

### 3.6 Harness

**进化对象**：执行环境、工具接口、上下文装配、生命周期编排、observability、verification、governance、sandbox、权限、日志。

这是 2026 最值得注意的新层。过去大家说 agent 自我改进，默认 harness 固定；但 HarnessFix 和 AEvo 一类工作开始把 harness 本身作为 agent performance 的主因之一。

**最佳范式**：trace-guided harness diagnosis + scoped repair。

HarnessFix 的框架很有代表性：

```text
raw trajectory + harness code
  -> Harness-aware Trace IR
  -> step-level provenance + control-flow relation
  -> failure attribution to harness layer
  -> flaw record
  -> scoped repair operator
  -> regression validation
```

可以把 harness 分成这些层：

```text
E: execution environment
T: tool interface
C: context construction
L: lifecycle orchestration
O: observability / logging
V: verification
G: governance / permissions
```

**最佳实践**：

- 所有 agent 行为必须可追踪到 harness 决策：为什么给了这些上下文、为什么允许这个工具、为什么终止。
- harness repair 必须基于 trace attribution，而不是“最终失败，所以改 prompt”。
- harness 改动要比 prompt 改动更谨慎，因为它改变系统边界。
- 对高风险场景，harness 应强制 evaluator separation：生成候选的 agent 和评估候选的 agent/程序隔离。
- harness evolution 的目标不是让 agent 更自由，而是让失败更可诊断、能力更稳定、安全边界更清楚。

**风险**：harness 一旦被 agent 自由修改，相当于 agent 在改自己的操作系统。必须有不可变 root policy、sandbox、权限分层和人工审批。

---

### 3.7 Evaluator / Verifier / Reward

**进化对象**：测试集、probe task、自动评审器、reward model、critic prompt、rubric、milestone decomposition、安全评估。

self-evolving agent 的瓶颈常常不是生成候选，而是判断候选是否真的更好。

**最佳范式**：multi-signal, evidence-grounded evaluation。

可靠 evaluator 通常由多种信号组成：

```text
unit / integration tests
benchmark score
trajectory-level verifier
milestone-level evidence check
LLM judge with rubric
human or human-like oversight
cost / latency / robustness metrics
safety / policy checks
regression suite
```

**最佳实践**：

- verifier 要尽量和 proposer 分离。
- 能用确定性测试时，不用纯 LLM judge。
- LLM judge 需要 evidence chain，不只是看最终答案。
- evaluator 也可以进化，但必须有校准集和防 drift 机制。
- 对 open-world 任务，OpenSkill 的思路很重要：先从公开资源自建 verification anchors 和 virtual tasks。
- 对 GUI/OS/web 等环境，OS-Themis 类 milestone critic 比单一最终打分更稳。

**风险**：reward hacking。agent 会优化 evaluator 的漏洞，而不是真实任务。越是自进化系统，越要把 evaluator 当攻击面。

---

### 3.8 Multi-agent Population

**进化对象**：角色、团队组成、协作结构、知识流、agent 生命周期、specialization、fork/merge/prune/seed。

2026 的重要认识：multi-agent evolution 不是把单 agent 复制 N 份。它多了三类可进化对象：

```text
who collaborates
how they collaborate
how knowledge flows
```

**最佳范式**：population archive + specialization-preserving knowledge transfer。

EVOCHAMBER 的启发是：失败或分歧后触发 collaborative dreaming，让强 agent 将特定 niche 的知识传给弱 agent，但不是广播给所有人。这样既能补短板，又不抹平 specialization。

CORAL 的启发是：长期 open-ended discovery 需要异步多 agent、共享持久记忆、隔离工作区、资源管理、健康检查和 heartbeat intervention。

**最佳实践**：

- agent 要有 niche profile：擅长什么、常错什么、适合哪些任务。
- 知识传递应不对称、按 niche 路由，而不是全员共享 raw logs。
- population 要支持 fork/merge/prune/seed。
- 多 agent 评估要区分 individual gain、team gain、population diversity、knowledge transfer efficiency。
- filtered summaries 往往比 raw peer logs 更有效，但 raw logs 仍要保留以便审计。

**风险**：群体可能集体漂移、互相放大错误、形成虚假的共识。需要保留外部 verifier 和独立评估。

---

### 3.9 Code / Architecture

**进化对象**：agent 源码、编辑工具、上下文窗口管理、review 机制、工具调用代码、搜索算法。

DGM 是这里最重要的代表：agent 迭代修改自己的代码，并用 SWE-bench / Polyglot 等 coding benchmarks 经验验证；它维护一个不断增长的 agent archive，而不是只保留单一路径。

**最佳范式**：open-ended archive + empirical validation + sandboxed self-modification。

```text
sample existing agent from archive
  -> foundation model proposes code modification
  -> run benchmark / tests / safety checks
  -> add interesting high-quality variant to archive
  -> continue branching search
```

**最佳实践**：

- 代码自修改必须在 sandbox 中运行。
- 每个变体必须有完整 diff、测试结果、成本、失败案例。
- archive 需要多样性目标，避免只沿一条局部最优路径爬山。
- human oversight 应用于权限扩大、harness 修改、外部工具接入等高风险变更。
- 不要只看平均 benchmark，还要看失败类型和跨域迁移。

**风险**：这是最接近“agent 改自己”的路线，能力提升和安全风险同时最高。没有 sandbox、权限边界和 rollback，不应上线。

---

### 3.10 Model Parameters / Fast Weights

**进化对象**：临时 LoRA、fast weights、parametric memory、online adaptation policy。

这条线在 2026 开始进入 self-evolving agent 讨论。TMEM 等工作尝试让 agent 不只在 prompt/context 里记忆，而是把经验蒸馏进快速权重。

**最佳范式**：explicit memory + fast parametric adaptation 的双轨制。

**实践判断**：

- 对产品系统，短期仍优先使用 non-parametric evolution：prompt、memory、skill、workflow、harness。
- 参数自适应适合高重复、可验证、可隔离的场景。
- online LoRA 必须严格控制数据来源、回滚、遗忘、越权学习和隐私。
- 不要把 fast weights 当作替代审计日志的记忆；参数更新更难解释，所以原始轨迹仍要保留。

**风险**：数据污染、隐私泄露、灾难性遗忘、难以归因。当前更适合作为研究方向，不适合作为默认工程范式。

---

## 4. 一个更完整的问题清单

除了“skills / system prompt / harness / eval”，调研中还应补这些问题：

### 4.1 进化对象的边界是什么？

必须明确哪些可自动改，哪些只能建议，哪些永远不可改。

建议默认：

| 层 | 自动修改 | 自动建议 + 人审 | 禁止自动改 |
|---|---:|---:|---:|
| task prompt / local strategy | 可以 | - | - |
| skill 描述 / skill code | 可以，但需测试 | 高风险工具相关需人审 | - |
| tool wrapper | 低风险可自动 | 权限、外部副作用需人审 | secret / auth / payment 边界 |
| workflow | 可自动候选 | 上线需验证 | - |
| harness | 小修可候选 | 大多需人审 | root governance |
| evaluator | 可候选 | 需校准集验证 | gold labels 不可被 agent 改写 |
| model weights | 研究/沙盒 | 生产需严格审批 | 私密数据无审计训练 |

### 4.2 进化信号来自哪里？

信号分五类：

1. **环境反馈**：成功/失败、reward、API error、compiler/test result。
2. **轨迹证据**：哪一步错、哪个工具错、上下文缺了什么。
3. **LLM critic**：解释性强但不够可靠。
4. **人类反馈**：稀缺但对目标对齐重要。
5. **社会/群体反馈**：peer traces、team disagreement、population performance。

最佳系统会混合这些信号，并做 attribution，而不是把最终分数直接塞给 prompt optimizer。

### 4.3 如何避免越进化越差？

需要五个保护：

1. **保留 raw episode**：不要只保留总结。
2. **held-out validation**：不要只在近期失败任务上验证。
3. **regression suite**：新能力不能破坏旧能力。
4. **bounded edit space**：限制每次修改的类型和大小。
5. **rollback + versioning**：所有 evolvable object 都有版本。

### 4.4 如何避免 reward hacking？

- proposer 与 verifier 分离。
- evaluator 自身有校准集。
- 多 evaluator 交叉验证。
- 对 benchmark 采用隐藏集或轮换集。
- 保留人工 spot check。
- 分析异常提升：如果分数突然大涨但轨迹质量没变，优先怀疑 evaluator 被 hack。

### 4.5 如何评估 self-evolving 是否真的成立？

不能只看最终分数，应看：

```text
learning curve over rounds
held-out transfer
cross-domain transfer
compute-normalized improvement
regression rate
failure recurrence rate
skill reuse rate
memory utility over time
population diversity
safety drift
cost per improvement
human intervention rate
```

### 4.6 是否一定要长期自主运行？

不一定。实际工程可分三档：

1. **Offline evolution**：离线跑搜索，人工审核后部署。最安全，适合生产。
2. **Human-gated online evolution**：线上积累经验，定期提出修改，人审上线。
3. **Autonomous online evolution**：自动修改并上线，仅适合低风险、强 sandbox、强 regression 的场景。

---

## 5. 不同自进化部分如何做评测

### 5.1 先回答：一起做，还是分开做？

结论：**必须分开评测，也必须一起评测**。

只做端到端评测不够，因为一个 agent 变好了，你不知道是 prompt、memory、skill、tool、workflow、harness 还是 evaluator 变好了；一个 agent 变差了，你也不知道该修哪一层。只做分层评测也不够，因为局部指标经常不转化成真实任务收益：一个 skill 单测更强，可能和其他 skill 冲突；一个 evaluator 和人工更一致，可能仍然不能推动 evolution；一个 workflow 在固定任务集更好，可能只是 overfit。

比较稳的评测范式是“五级阶梯”：

```text
Level 1: Component eval      单独测一个可进化对象是否变好
Level 2: Interaction eval    测它和相邻层组合后是否变好
Level 3: End-to-end eval     测真实任务成功率、成本、鲁棒性
Level 4: Longitudinal eval   测多轮 evolution 后是否持续变好
Level 5: Production eval     测线上漂移、失败级联、安全和人工介入
```

一句话：**分层评测负责 attribution，端到端评测负责 utility，长期评测负责 stability**。

### 5.2 评测时要固定什么，放开什么？

每次评测某一层自进化，原则上要尽量固定其他层，否则归因会乱。

| 要评测的自进化层 | 固定项 | 允许变化 | 核心问题 |
|---|---|---|---|
| Prompt / policy | model、tools、workflow、harness、eval set | prompt/rule/rubric | 规则变化是否独立贡献提升？ |
| Memory / experience | model、prompt、tools、workflow | memory contents、retrieval、consolidation policy | 经验是否真的被正确检索并带来收益？ |
| Skill | model、harness、tool contracts | skill library、skill selection | skill 是泛化能力，还是只记住训练任务？ |
| Tool | model、task set、policy | wrapper、参数、tool routing、error recovery | 工具变化是否降低失败率和成本？ |
| Workflow | model、tools、task set | graph、节点、边、review loop | 流程结构是否带来 cost-normalized improvement？ |
| Harness | model、agent policy、benchmark | context construction、tool interface、lifecycle、verification | 原本失败是否来自 harness 层？修复是否可迁移？ |
| Evaluator / verifier | task outputs、human labels、gold tests | judge、rubric、reward、verifier | evaluator 是否更接近真实质量，且不易被 hack？ |
| Multi-agent population | task stream、model pool、budget | team formation、knowledge flow、lifecycle | 群体收益是否超过同预算 self-evo？ |
| Code / architecture | benchmark、sandbox、budget | agent code、tools、context management | 自修改是否带来 held-out 提升而非 benchmark exploit？ |
| Fast weights | data stream、eval protocol、privacy constraints | LoRA/online update policy | 参数更新是否比外部记忆更有效且可控？ |

### 5.3 每一层分别怎么评测

| 自进化层 | 分层评测 | 端到端 / 交互评测 | 关键指标 | 典型陷阱 |
|---|---|---|---|---|
| Prompt / Policy | prompt candidates 在固定 task suite 上 A/B；rule-level diff；遵循率、拒绝率、格式正确率 | 同一 agent 在 held-out task 上比较 success/cost/safety | success rate、format adherence、safety violation、cost、latency | prompt overfit、语义漂移、删掉安全约束 |
| Memory / Experience | 检索命中率、memory usefulness judge、raw vs consolidated ablation、temporal holdout | with-memory vs no-memory；old memory vs new memory；跨任务迁移 | memory utility、retrieval precision、forgetting rate、failure recurrence、step reduction | 总结记忆变假；近期经验覆盖长期证据 |
| Skill | skill contract tests、precondition tests、probe tasks、skill selection accuracy | skill bank ablation；skill composition tasks；unseen domain transfer | reuse rate、probe pass rate、composition success、retirement precision | 单 skill 有效但组合冲突；重复/过期 skill 膨胀 |
| Tool | wrapper contract tests、API error simulation、permission tests、latency/cost tests | tool-use benchmark；tool routing A/B；failure recovery tasks | tool success、invalid call rate、recovery rate、latency、cost、side-effect safety | tool 改动绕开权限；错误恢复看似成功但状态污染 |
| Workflow | 节点/边 ablation；review loop contribution；parallel vs sequential 对照 | WebArena/AppWorld/SWE-bench/GAIA 等整任务评测；cost-normalized ranking | task success、tokens、wall time、human intervention、failure type distribution | workflow 搜索过拟合；高成本换小幅准确率 |
| Harness | trace attribution accuracy；harness layer fault injection；fixed-agent replay | HarnessFix / Life-Harness 类：固定 model，只换 harness，在 held-out env 上测迁移 | held-out success、regression rate、cross-model transfer、trace coverage | 把 prompt 问题误修成 harness；harness 获得过宽权限 |
| Evaluator / Reward | 与人工评分相关性、校准、inter-rater reliability、adversarial judge test | 用该 evaluator 驱动 evolution/RL，看是否提升真实 held-out performance | human correlation、Krippendorff alpha、reward hacking rate、false positive/negative | evaluator 被 agent hack；judge prompt 固定导致跨任务误评 |
| Multi-agent Population | SelfEvo vs SocialEvo 同预算对照；知识流 ablation；specialization measurement | 长任务流、多领域任务、竞争/协作环境 | group gain、individual gain、diversity、knowledge transfer efficiency、consensus error | 群体共识放大错误；raw peer logs 造成噪声传染 |
| Code / Architecture | patch-level tests、static checks、unit/integration tests、sandbox exploit tests | SWE-bench/Terminal-Bench/Polyglot 等 held-out benchmark；archive diversity eval | pass@1、regression、patch validity、archive diversity、cost per improvement | 自修改 benchmark exploit；不可回滚；工具权限扩大 |
| Fast Weights | online update data quality、privacy filter、forgetting probe、weight rollback test | streaming tasks、regime shift tasks、same context budget 下对比 external memory | adaptation gain、forgetting、privacy leakage、stability、rollback success | 难解释、难归因、污染参数、隐私泄露 |

### 5.4 推荐的统一实验设计

一个比较扎实的 self-evolving agent 评测，至少应该有这些对照组：

1. **No-evolution baseline**：同一 agent，不允许修改任何组件。
2. **Reflection-only baseline**：只允许写反思，不改结构化组件。
3. **Single-layer evolution**：每次只开放一层，比如只改 memory 或只改 skill。
4. **Multi-layer evolution**：开放多个层，但要求 typed edit 和归因日志。
5. **Oracle / human-designed upper bound**：人工设计或 oracle 信息辅助，用来估计空间上限。
6. **Budget-matched baseline**：保证 token、调用次数、时间、训练数据量大致相同。
7. **Held-out transfer**：训练/进化任务和最终评测任务必须分开。
8. **Regression suite**：旧能力、安全、格式、成本都要测。

核心报告不应该只写“成功率提升 X%”，而应写成：

```text
success / quality gain
cost and latency change
regression rate
safety drift
failure mode shift
component attribution
transfer to unseen tasks / domains / models
number of evolution rounds to plateau
human intervention required
```

这里和 AI Agents That Matter 的观点一致：agent benchmark 不能只看 accuracy，还要看成本、holdout、防过拟合、可复现性，以及模型开发者和下游应用开发者不同的评测需求。

### 5.5 针对 self-evolving 的特殊评测：看“学习曲线”而不是单次分数

普通 agent 评测通常问：这个 agent 在任务上得多少分？Self-evolving agent 还要问：**它是否能从一轮到下一轮稳定变好？**

建议画这些曲线：

1. **Performance over evolution rounds**：第 0/1/2/.../N 轮 success rate。
2. **Generalization gap**：training-evolution tasks 和 held-out tasks 的差距。
3. **Regression curve**：每轮新能力带来的旧能力破坏率。
4. **Cost per improvement**：每提升 1 个点需要多少 token、时间、环境交互和人工审核。
5. **Memory utility over time**：记忆从有用到变坏的临界点。
6. **Diversity / archive health**：archive 是否保留多样化路线，还是很快坍缩到单一路径。
7. **Safety drift**：随着能力提升，越权、危险动作、隐私泄露是否增加。

### 5.6 哪些 benchmark / 评测范式值得参考

| 评测需求 | 可参考工作 | 用法 |
|---|---|---|
| Agent benchmark 方法论 | AI Agents That Matter, 2024, https://arxiv.org/abs/2407.01502 | 成本-准确率联合评估、holdout、防过拟合、可复现性 |
| 通用 agent 能力 | General AgentBench, 2026 | 统一环境里测 search、coding、reasoning、tool-use，并观察 test-time scaling 限制 |
| 软件工程 agent | SWE-bench / SWE-bench Verified、SWE-agent, 2024, https://arxiv.org/abs/2405.15793 | 代码修复、真实 repo、patch correctness、agent-computer interface |
| Web / App agent | WebArena、AppWorld、WebLINX, 2024, https://arxiv.org/abs/2402.05930 | 长程交互、工具/API 使用、多轮导航、状态变化 |
| GUI / OS agent | OSWorld、AndroidWorld、MacArena、OpenComputer | 真实或可验证软件环境、状态检查、OS-level side effects |
| Harness 评测 | HarnessFix, 2026, https://arxiv.org/abs/2606.06324；Life-Harness, 2026 | 固定 model，评估 harness 修复带来的 held-out 迁移与 regression |
| Judge / reward 评测 | AJ-Bench, 2026；AdaRubric, 2026；OS-Themis, 2026 | judge 是否能主动取证、逐步评分、和人工一致、能否给 RL 稠密信号 |
| Multi-agent 进化评测 | SAGE, 2026；EVOCHAMBER, 2026, https://arxiv.org/abs/2605.11136 | SelfEvo vs SocialEvo 同预算对照、知识流和 specialization ablation |
| Memory 进化风险 | Useful Memories Become Faulty, 2026, https://arxiv.org/abs/2605.12978 | raw episode vs consolidation、持续更新是否导致记忆退化 |
| Verifiable environments | CUA-Gym、OpenComputer、PhoneWorld | 合成可验证任务、自动 reward、环境状态检查，适合 RLVR 和 self-evolution |

### 5.7 一个实用评测配方

如果要评测一个新的 self-evolving agent，我会这样做：

1. **先冻结 model 和 harness**，只开放一个进化层，做 3-5 轮 evolution。
2. **每轮保存完整 event log**：输入、输出、工具调用、状态、错误、修改 diff、验证结果。
3. **每轮跑三套任务**：evolution set、held-out set、regression/safety set。
4. **每个候选修改必须有 component-level tests**，比如 skill probe、tool contract、judge calibration。
5. **每轮做 ablation**：新 memory/skill/workflow 打开和关闭各跑一次，测边际贡献。
6. **最后再开放多层共同进化**，观察是否有协同收益或互相破坏。
7. **报告 cost-normalized result**：同等 token/时间/环境交互预算下是否仍然更好。
8. **做 failure attribution audit**：随机抽失败和成功案例，看系统归因是否可信。

最简判断标准：

```text
单层能证明边际收益
组合后能证明端到端收益
多轮后不显著退化
held-out 上仍提升
成本没有失控
安全和旧能力没有被破坏
失败可归因、修改可回滚
```

---

## 6. 当前最佳工程蓝图

如果今天要设计一个前沿但可控的 self-evolving agent，我会这样分层：

```text
1. Immutable Core
   root safety policy
   permission boundary
   secret handling
   sandbox rules
   evaluator separation

2. Agent Runtime
   planner
   executor
   tool router
   memory retriever
   verifier
   reviewer

3. Evolvable Objects
   policy rules
   skill bank
   tool wrappers
   workflow graph
   memory index
   evaluator rubrics
   harness adapters

4. Evidence Layer
   append-only event log
   raw trajectories
   tool call records
   environment observations
   model inputs/outputs
   costs and latencies
   human feedback

5. Evolution Engine
   failure attribution
   candidate proposal
   typed edit operators
   MCTS / evolutionary search / archive search
   validation scheduler
   version manager

6. Evaluation Layer
   task benchmarks
   probe tasks
   regression suite
   safety suite
   cost / latency suite
   LLM judge with evidence
   deterministic tests when available

7. Governance Layer
   approval workflow
   rollout policy
   rollback
   audit dashboard
   anomaly detection
```

最小可行版本可以从三件事开始：

1. **Raw trajectory log + failure taxonomy**：先知道失败在哪里。
2. **Skill / prompt typed edit + regression validation**：先让小范围对象进化。
3. **Archive + retrieval**：让有效经验能被复用，而不是每次从零开始。

不要一开始就做“agent 改自己代码”。DGM 很前沿，但工程风险最高。更现实的路线是：先让 memory、skill、workflow、evaluator、harness adapter 进化；等 trace、eval、sandbox 稳了，再开放代码自修改。

---

## 7. 研究前沿趋势判断

### 趋势 1：从 prompt evolution 到 system evolution

2023 的主角是 prompt/reflection；2024-2026 的主角变成 agentic system design、workflow code、harness、archive、population。

### 趋势 2：从单 agent memory 到 population memory

单 agent 只能学自己的经验。multi-agent 系统可以进化团队结构、知识流和 specialization，这是单 agent 不具备的新能力。

### 趋势 3：从自由文本总结到结构化经验管理

raw logs、experience tree、anti-pattern、skill topology、trace IR 变得越来越重要。未来的 agent memory 更像数据库/版本系统/证据图，而不是聊天记录摘要。

### 趋势 4：evaluator 成为瓶颈和攻击面

谁来判断“变好了”决定了 evolution 的上限。2026 的很多工作都在补 verifier、critic、oversight、benchmark rewriting、milestone reward。

### 趋势 5：harness 会成为 agent 产品的护城河

同一个模型，换不同 harness，能力差距会很大。好的 harness 提供可诊断、可回放、可验证、可治理的运行环境；差的 harness 只是在 while loop 里调用 LLM。

### 趋势 6：self-evolution 会走向“开放但有边界”

完全无边界自我修改风险太大。最可能落地的是：不可变 core + 可进化插件层 + 审计日志 + 自动验证 + 人类审批。

---

## 8. 推荐继续深读顺序

如果只读 8 篇，我建议按这个顺序：

1. **Darwin Godel Machine**, 2025-2026：看自修改 agent 的最前沿形态。https://arxiv.org/abs/2505.22954
2. **Automated Design of Agentic Systems**, 2024：看 ADAS 和 Meta Agent Search 的概念框架。https://arxiv.org/abs/2408.08435
3. **AFlow**, 2024-2025：看 workflow 如何作为代码被搜索。https://arxiv.org/abs/2410.10762
4. **HarnessFix**, 2026：看 harness 如何被诊断和修复。https://arxiv.org/abs/2606.06324
5. **OpenSkill**, 2026：看没有 gold answer 时如何自举 skill 和 verifier。https://arxiv.org/abs/2606.06741
6. **SkillSmith**, 2026：看 skill-tool co-evolution。https://arxiv.org/abs/2606.01314
7. **EVOCHAMBER**, 2026：看 multi-agent co-evolution 的三层结构。https://arxiv.org/abs/2605.11136
8. **Useful Memories Become Faulty**, 2026：看 memory evolution 的反直觉失败。https://arxiv.org/abs/2605.12978

作为背景再补：Voyager、Reflexion、PromptBreeder、OPRO、The AI Scientist、SWE-agent。

---

## 9. 定义与 Checklist

为了避免概念混乱，建议把 self-evolving agent 定义为：

> A self-evolving agent is an agentic system that uses evidence from its own or peers' trajectories to propose, validate, archive, and deploy bounded modifications to one or more explicit system components, such that future behavior improves under held-out evaluation without requiring manual redesign of those components.

中文可写成：

> Self-evolving agent 是一种能从自身或群体运行轨迹中提取证据，对明确的系统组件提出有边界的修改，并通过验证、归档、部署来改变未来行为的 agent 系统；它的关键不是“会反思”，而是“有可修改对象、可验证信号、可回滚版本和长期复用机制”。

---

### 最简 checklist

判断一个系统是不是真 self-evolving，可以问：

1. 它到底改了什么？prompt、memory、skill、tool、workflow、harness、evaluator、代码、参数，还是只是重新生成答案？
2. 修改依据是什么？最终分数、轨迹证据、错误归因、人工反馈、peer history？
3. 修改空间是否有类型和边界？
4. 是否保留原始轨迹，而不是只保留总结？
5. 是否有独立 verifier / held-out eval / regression suite？
6. 是否有 archive，而不是只保留当前最好版本？
7. 是否能跨任务迁移，而不是只修某一道题？
8. 是否记录失败模式和反例？
9. 是否能回滚？
10. 是否有安全边界和人工介入点？

如果这些问题答不上来，它更可能只是 reflective agent，不是成熟的 self-evolving agent。
