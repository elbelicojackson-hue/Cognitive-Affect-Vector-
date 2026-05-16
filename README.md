# Cognitive Affect Vector (CAV) — 认知情态向量

> **A second communication channel for multi-agent LLM consensus.**
> 给多智能体 LLM 共识系统增加一条与"内容"正交的"姿态"通信通道。

[![Status](https://img.shields.io/badge/status-theoretical_draft_v0.2-orange)]()
[![Validation](https://img.shields.io/badge/empirical_validation-pending_KPI--3-yellow)]()
[![License](https://img.shields.io/badge/license-TBD-lightgrey)]()

---

## 这是什么

人类形成共识从来不只靠"对方说了什么"。我们听语调、看微表情、感受停顿、注意对方反复修改自己话的次数——这是一条**与内容正交的元通信通道**。

当前的多模型 / 多 Agent LLM 系统只交换 content（一段文本）。这等价于两个人只能通过书面文字交流。导致：

- 模型可以**嘴硬**而看不出（无法识别 calibration 缺失）
- 同源模型可以**抱团**而不被发现（content 一致 ≠ 独立同意）
- 系统**死机收敛**和真共识看起来一模一样（content 都不动了）

**Cognitive Affect Vector (CAV, 𝓒)** 是一个 10 维向量，承载每个 Agent 在每次发言时的"姿态"——置信度、犹豫度、自一致性、修复风格、信念更新幅度、推理深度等。它从对话表面**可观测**（不依赖 Agent 主动汇报），并通过双通道贝叶斯进入共识推理：

```
单通道（旧）：    P(T | content)         ∝   P(content | T) · P(T)
双通道（新）：    P(T | content, 𝓒)      ∝   P(content | T) · P(𝓒 | T) · P(T)
```

T 是 claim 真假。CAV 让群体能够"察言观色"。

---

## 它不是什么

- 不是 prompt engineering 套路
- 不是 LLM 内部机制（不需要修改模型权重，纯外部协议层）
- 不是 metadata（CAV 信号会进入贝叶斯共识公式，**改变** posterior）
- 不是又一个评分仪表盘——它是一个有信息论保证的第二通信信道

---

## 快速跳读

| 章节 | 主题 |
|---|---|
| [§1](#1-起源v1-的三个根本缺陷) | V1 三个根本缺陷：角色固化 / 熵不是熵 / 单通道 |
| [§2](#2-细胞模型去除角色引入区室) | 细胞模型：模型不持有角色，只持有区室 |
| [§3](#3-四链架构双速分离--双推理) | 四链架构：快/慢推理 × 快/慢记忆 |
| [§4](#4-三层严格熵框架) | 三层严格熵 H₁ / H₂ / H₃ |
| [**§4½**](#4½-离散化层-dl) | **离散化层 DL：连续 LLM ↔ 离散信息论的桥** |
| [§5](#5-唯一算法对抗性不动点搜索) | 唯一算法：对抗性不动点搜索 + 外部锚点 |
| [**§5½**](#5½-离散梯度-dg驱动力学) | **离散梯度 DG：系统的驱动力学** |
| [**§6**](#6-认知情态向量-cav第二通信通道) | **CAV：第二通信通道（核心）** |
| [§7](#7-大统一图) | 大统一图 |
| [§8](#8-与已有架构的对应迁移指南) | 迁移指南 |
| [§9](#9-可证伪条件-kpis) | 可证伪 KPI |
| [§10](#10-开放问题) | 开放问题 |
| [§11](#11-命名约定) | 命名约定 |
| [§12](#12-一句话总结) | 一句话总结 |

---

## 摘要

NeuralComm V1 解决了"多模型如何就一个 claim 进行可审计共识"这个问题，但在三个层面上是有缺陷的：

1. **角色固化**：把每个模型预设为感知/推理/抑制/元认知中的一个，导致模型池不可伸缩、与现有实践不一致。
2. **熵不是熵**：v1 的 10 维向量里只有 1 维是真正的 Shannon 熵，其余是关键词密度计数；中文场景下分歧熵恒为 0.97 即此根因。
3. **单通道通信**：模型之间只能交换 content（一段文本），缺少正交于内容的"姿态/状态"通道，导致系统无法区分真共识、脆弱共识、死机收敛，也无法检测共谋。

V2 给出三个对应的理论替代：

- **细胞模型 (Compartment Model)**：模型不分配角色，分配区室（膜配置）；同一模型在不同膜下产生不同输出。
- **三层严格熵 (Three-Tier Rigorous Entropy)**：H₁ 自一致性、H₂ 信念分布、H₃ 过程轨迹，全部建立在真信息论上。
- **离散化层 (Discretization Layer, DL)**：连接连续 LLM 嵌入空间与离散信息论世界的中间层；用 Rényi 谱 + 多尺度方案避免单一 K 的粒度选择困境。
- **离散梯度 (Discrete Entropy Gradient, DG)**：把整个共识过程改写为有限差分梯度下降——系统不再是被动观测，而是沿 ∇H 方向主动行动。
- **认知情态向量 (Cognitive Affect Vector, CAV / 𝓒)**：与 content 正交的第二条通信通道，承载置信、犹豫、校准、修复风格等元状态，使群体能"察言观色"。

并给出一个统一的算法骨架——**对抗性不动点搜索 (Adversarial Fixed-Point Search)**——作为"多模型快速识别正确答案 / 形成共识的唯一算法"的候选答案。其动力学版本可一句话写完：

```
state s ∈ S
action a ∈ A_attack × A_discretize × A_swap × A_oracle × A_chain
gradient ∇H : A → ℝ              (经代理估计)
update s_{t+1} = apply(argmax_a [̂∇H(a) − λ·cost(a)], s_t)
fixed point  ⇔  ∇H(a) ≤ 0  ∀a
truth        ⇔  fixed point ⊕ external oracle
```

V2 在数学上就是这六行。

---

## 1. 起源：V1 的三个根本缺陷

### 1.1 角色固化 (Role Lock-in)

V1 的 `THEORY.md` 把每个模型预设为四角色之一：感知/推理/抑制/元认知。这有两个问题：

- **不可伸缩**：模型池每加一个新模型就要决定它属于哪一层。
- **与实践不一致**：v1 的 `arena.py` 实际上让所有模型对称地构链、攻击、防守、判定——它的代码事实上**已经放弃了角色分层**，但理论文档没有更新。

### 1.2 熵不是熵 (Entropy is Not Entropy)

V1 `entropy.py` 的 10 维向量中：

| 维度 | 真实计算 | 是不是熵 |
|---|---|---|
| `divergence` | `−p log p − (1−p) log(1−p)` | **是** |
| `dependency` | 比率加权 | 半是 |
| 其余 8 维 | 关键词密度 / 离散桶 / 图比率 / ad-hoc 评分 | **否** |

- **症状**：中文字符集天然重叠 → 实测分歧熵 0.97 不动；同义改写后所有"熵"剧烈变化（不变性丧失）；跨语言不可迁移。
- **后果**：v1 的"熵驱动决策"实际上是"关键词驱动决策"。

### 1.3 单通道通信 (Single-Channel Communication)

V1 中模型间只能交换 content。这等价于：

> 两个人只能通过书面文字交流，不能听对方语调、看对方表情、感受对方停顿。

人类在共识形成中**大量信息走在 content 之外**（韵律、犹豫、修辞 commitment、肢体）。AI 之间至今没有这条带宽，导致：

- 模型可以嘴硬而看不出（无法识别 calibration 缺失）
- 同源模型可以抱团而不被发现（content 一致 ≠ 独立同意）
- 系统死机和真共识看起来一模一样（content 都不动了）

**V1 的所有架构都建立在贫瘠的单通道上**。

---

## 2. 细胞模型：去除角色，引入区室

### 2.1 公理

> **公理 0**：模型不持有角色。模型是算力，区室是环境，输出是算力 × 环境的函数。

形式化地：

```
P(output) = f(model, environment)
环境  E = (input_filter, memory_window, attack_topology)
```

同一模型 m 放进环境 E₁ 和 E₂ 给出不同输出。差异不来自模型的"身份"，来自**膜的透过性**。

### 2.2 类比对齐

| 角色派 | 区室派 |
|---|---|
| 模型 = 员工 | 模型 = 算力单元 |
| 角色 = 岗位职责 | 区室 = 细胞器环境 |
| 协作 = 分工 | 协作 = 跨膜信号转导 |
| 共识 = 投票 | 共识 = 浓度梯度趋零 |

### 2.3 可证伪性测试

如果"区室不是角色"是对的，必须满足：

> 同一模型在两个不同区室的输出差异 > 不同模型在同一区室的输出差异
>
> 即：JSD(m | E₁, m | E₂) > JSD(m₁ | E, m₂ | E)

这条不等式如果在 logs 上跑出来不成立，整个细胞模型就是错的——必须老实退回角色派。这是 V2 的**架构 KPI 之一**。

---

## 3. 四链架构：双速分离 × 双推理

### 3.1 两个正交轴

```
  推理速度轴：   System 1 (单次, 低成本)   ←→   System 2 (多步, 高成本)
  记忆深度轴：   Working Memory (本会话)    ←→   Long-term Store (跨会话, 衰减)
```

笛卡尔积 = 4 个区室：

|  | 快推理 | 慢推理 |
|---|---|---|
| **快记忆** | C₁ 直觉链 | C₂ 反思链 |
| **慢记忆** | C₃ 专家链 | C₄ 整合链 |

- **C₁ Intuition**：只看 question + 当前 utterances，单次调用，类反射弧
- **C₂ Reflection**：question + 自己的中间推理，递归质疑，类前额叶 working memory
- **C₃ Expertise**：不看新对话，只检索 consensus_store 做模式匹配，类海马记忆检索
- **C₄ Integration**：全量信息 + 完整 chain-centric arena，类新皮层巩固

四条链**对所有模型对称开放**——任何模型都可以被分配到任何区室；每条链可以让一个模型跑、也可以让多个模型并行跑。

### 3.2 四链碰撞表（区分陷阱、新颖、保守、混乱）

| C₁ | C₂ | C₃ | C₄ | 系统判定 |
|---|---|---|---|---|
| ✓ | ✓ | ✓ | ✓ | 强共识，EMIT |
| ✓ | ✗ | ✓ | ✗ | **直觉欺骗 / 话术陷阱**：慢路径覆盖 |
| ✗ | ✓ | ✗ | ✓ | **新颖问题**：先验失效，信慢推理 |
| ✓ | ✓ | ✗ | ✗ | **既有共识需更新**：触发 store revoke |
| ✗ | ✗ | ✓ | ✓ | **当前对话被污染**：回退先验 |
| 全分歧 | | | | 真混乱：跌回完整 arena |

这张表的关键在第 2 行——这正是量子陷阱题（"量子计算机能破解 RSA-2048..."）的本质：直觉和模式匹配都被花哨术语骗到，但慢推理能拆穿。在没有四链的 arena 里靠 9 轮辩论才逼出来；四链架构第 0 轮就能产生信号。

---

## 4. 三层严格熵框架

### 4.1 第一原理

回到信息论。多 agent 推理系统中唯一有定义良好概率空间的对象是：

```
Ω    = 所有可能答案的集合（claim 空间）
P_t  = 系统在时刻 t 对正确答案的信念分布
H(P_t) = −Σ_{ω∈Ω} P_t(ω) log P_t(ω)
```

系统的所有工作（攻击、防守、共识）都应该被理解为**降低 H(P_t) 的算子**。但 P_t 不可观测，必须用三层估计器逼近。

### 4.2 Tier 1 — 自一致性熵 (Within-agent)

**度量**：agent 自己有多确定？

```
H₁(agentᵢ, q)  =  − Σ_c π_c log π_c

其中 π_c = (1/K) Σ_k 𝟙{response_k ∈ cluster c}
       K 次重复采样后语义聚类的簇分布
```

**性质**：真 Shannon 熵；不依赖关键词；跨语言成立。

**文献基础**：Kuhn / Gal / Farquhar, *Semantic Entropy*, ICLR 2023。

### 4.3 Tier 2 — 信念分布熵 (Between-agents)

**度量**：群体的信念分布有多分散？

```
H₂(claims, agents) = − Σ_c (W_c / W) log (W_c / W)

W_c = Σ_{a ∈ cluster c} reputation(a)     ← 信誉加权
W   = Σ_a reputation(a)
```

**附加量**：
- 两两 JSD 矩阵：`JSD(agentᵢ, agentⱼ)`
- 互信息：`I(agentᵢ ; agentⱼ)` — 检测共谋

**互信息的工程含义**：两个 agent 判断高度相关 (I 高) 时，它们在加权投票里只能算"半个独立票"。这把"模型独立性 ρ"从理论假设变成可观测量。

### 4.4 Tier 3 — 过程熵 (Across-time)

四个真量：

```
(a) 攻击覆盖熵     H_cov   = Shannon( 攻击落点在所有可攻击步骤上的分布 )
(b) 攻击角度熵     H_ang   = Shannon( 攻击理由的语义簇分布 )
(c) 信息增益       IG_t    = H₂(t-1) − H₂(t)
(d) 信念熵流方差   V_flow  = Var( H₂[t-K : t] )
```

### 4.5 真共识 vs 死机：可分辨化

```
真共识     ⇔   H₂ 低 ∧ H_cov 高 ∧ H_ang 高 ∧ V_flow ≈ 0
死机       ⇔   H₂ 低 ∧ H_ang 低 ∧ IG ≈ 0 ∧ V_flow ≈ 0
脆弱共识   ⇔   H₂ 低 ∧ ( H_cov 低 ∨ H_ang 低 ) ∧ V_flow 仍波动
```

两者都满足 V_flow ≈ 0，区别在 H_cov 和 H_ang。**这是 v1 没区分的根因**。

### 4.6 系统总熵的可加分解

```
H_system  =  H₂(belief)  +  λ · process_penalty

process_penalty = max(0, τ_cov − H_cov)
                + max(0, τ_ang − H_ang)
                + max(0, IG_recent)
```

**定理 4.1**：当 process_penalty = 0（探索充分），H_system = H₂，是真信念熵。
当 process_penalty > 0（探索不足），H_system 显式高于 H₂，反映"我们还不知道我们不知道什么"。

死机时 H₂ 很低但 process_penalty 高 → H_system 不低 → 系统不会误把死机当共识。

---

## 4½. 离散化层 (DL)

> 之前在脊柱里被一句话糊弄过去的"Python 端做一下桶化"，其实是 V2 的成败关键。
> Shannon 熵的公式建立在**离散**概率空间上，而 LLM 的世界**全是连续的**。
> 中间这一层翻译——离散化层 DL——必须显式立起来，否则前一节所有"真熵"都是空中楼阁。

### 4½.1 五个无一例外需要离散化的信号

| 信号 | 连续源 | 期望的离散形式 |
|---|---|---|
| H₂ 信念熵 | claim 嵌入 ∈ ℝ^D | claim → cluster_id ∈ {1..K} |
| H_ang 攻击角度熵 | 攻击理由嵌入 | reason → angle_id ∈ {1..K_a} |
| H_cov 攻击覆盖熵 | step indices | 已离散 ✓ |
| H₁ 自一致性熵 | K 次采样 | sample → cluster |
| MI(𝓒ᵢ; 𝓒ⱼ) | CAV ∈ ℝ¹⁰ | 联合分布 ∈ {1..B}² |

### 4½.2 粒度困境

```
K 太大（每个 claim 独占一桶）→ H₂ 永远高，永远没共识
K 太小（合并为 2-3 桶）       → H₂ 永远低，永远伪共识
固定 K                         → 下道题题型不同就崩
```

这不是超参问题，是结构问题——**单一固定 K 一定错**。

### 4½.3 解法 1：多尺度熵

不要选一个 K，**同时算三个尺度**：

```
H^(fine)    K = N            "每人都不同"
H^(med)     K = ⌈√N⌉         "几个阵营"
H^(coarse)  K ∈ {2, 3}       "赞成 / 反对"
```

读三元组的形状即可判断系统状态：

| fine | med | coarse | 系统状态 |
|---|---|---|---|
| 低 | 低 | 低 | 真群体趋同 |
| 高 | 高 | 高 | 真分歧 |
| 高 | 低 | 低 | 大方向一致细节各异（最常见） |
| 低 | 高 | 低 | 几个阵营内部抱团 — **共谋信号** |
| **高** | **高** | **低** | **全员同意广泛立场但具体说法各异 — 话术陷阱征兆** |

最后一行就是量子陷阱题在 v1 的形态——多尺度差异本身就是新信号。

### 4½.4 解法 2：Rényi 熵谱

更优雅。Rényi 熵：

```
H_α(P) = (1 / (1−α)) · log Σ p_i^α     （α ≠ 1）
H_1(P) = −Σ p_i log p_i                 （Shannon, α = 1）
H_∞(P) = −log max_i p_i                 （min-entropy）
H_0(P) = log |support(P)|               （Hartley）
```

**只需要离散化一次**就能算整条 Rényi 谱：

```
α 大   → 对模式集中度敏感
α 小   → 对支撑集大小敏感
```

谱形态对应共识形态：

| 谱形 | 解读 |
|---|---|
| H₀ ≈ H₁ ≈ H_∞ 全相等 | 完全均匀，真分歧 |
| H₀ 大但 H_∞ 小 | 大众一致 + 少数异议（健康分歧） |
| H_∞ ≈ H₁ 但 H₀ ≫ H₁ | 一两个主流派 + 噪声残党 |
| 整条谱都低 | 真共识 |

Rényi 谱也是**离散化质量的鲁棒性测试**——好的离散化下整条谱稳定；坏的离散化下不同 α 会给冲突信号。

### 4½.5 三种离散化方案

```
A 软离散化     softmax(−τ · d(emb, centroid)) → 分配概率向量
              下游连续可微（梯度算法用）
              工程速度最快

B 双向蕴含     c_i ≡ c_j ⇔ NLI(c_i ↔ c_j) 双向 entailment
              理论 ground truth；K 自动决定；同义改写不变性自动满足
              成本 O(N²) NLI 推理

C 嵌入 + 自适应阈值  cosine ≥ τ(question) 同簇
                    τ 从问题语义弥散度自适应
                    工程默认
```

**地位**：B 是 ground truth；C 是工程默认；A 是梯度算法的快路径。

### 4½.6 离散化质量 KPI（KPI-7）

```
KPI-7-a 粒度稳定性：       H_α(谱) 在不同 K 下相对变化 < 10%
KPI-7-b 同义改写不变性：   |H₂(claims) − H₂(paraphrased)| < 0.05
KPI-7-c 方案间一致性：     |H₂_A − H₂_B| < δ_AB,  |H₂_C − H₂_B| < δ_CB
```

如果 KPI-7 不通过，整套熵框架在沙地上盖楼，必须换离散化方案。

---

## 5. 唯一算法：对抗性不动点搜索

### 5.1 问题陈述

> "多模型快速识别正确答案 / 形成共识的唯一算法是什么？"

需要同时满足两个目标：

```
A: 趋近 truth      （可能没共识但答案对）
B: 趋近 consensus  （共识达成但可能错）
```

唯一算法必须满足：**它的不动点既是共识又是真理**。

### 5.2 候选筛选

排除：

- **加权投票 + 信誉**：共识快但不动点不必为真
- **多轮辩论收敛**：终止条件可被"双方都累了"伪装
- **路由分派**：根本不协作

留下：**对抗性不动点 (Adversarial Fixed-Point)**。

### 5.3 形式定义

设：

```
A: claim × agent → AttackProposal ∪ {⊥}     — 攻击算子
V: AttackProposal × agent → {accept, reject} — 验证算子
S_t: 第 t 轮被群体接受的 claim 集合
```

**定义 5.1（不动点）**：claim c 是**稳定的**当且仅当

```
∀ agentᵢ, ∀ rounds, 多数 agent_j ≠ i 都对 A(c, agentᵢ) 投 reject

即： |{ j : V(A(c, agentᵢ), agent_j) = reject }|  ≥  ⌈N/2⌉
```

**性质**：

- **角色对称**：每个 agent 同时是攻击者、被攻击者、裁判。完美兼容公理 0。
- **truth ⊕ consensus 绑定**：稳定 claim 必为 Schelling point（共识），且必挺过独立误差不相关的攻击（趋近 truth）。
- **收敛速率**：错误 claim 在 N 轮存活概率 ≤ (1 − ρ)^N，其中 ρ 是 agent 池的独立攻击能力。

### 5.4 算法的根本边界

**定理 5.1（ρ-bound）**：纯 agent 间对抗不动点的 truth 上限为 ρ_max — 模型池能找到的最强独立错误。如果模型池在某领域错误高度相关 (I(ψᵢ; ψⱼ) 高)，对抗不动点收敛到错误不动点，**任何纯 agent 间算法都救不了**。

推论：要保证"识别正确答案"必须**外接锚点**：

```
truth  ≈  对抗不动点  ⊕  外部 ground oracle
```

⊕ 是优先级覆盖：当对抗不动点和 oracle 冲突时，oracle 赢。

### 5.5 完整唯一算法

> **唯一算法 = 对抗性不动点搜索 + 外部锚点优先级覆盖**

前一半保证一致性和速度，后一半保证不会全员一起翻车。

---

## 5½. 离散梯度 (DG)：驱动力学

> 之前所有章节都在算"现在状态熵是多少"，
> 但 AI 共识不是位置问题，是**运动**问题。
> 决定下一步动作的不是 H 本身，是 ∇H/∇action。
> DG 就是把整个系统从"被动观测"升级为"主动行动"的数学层。

### 5½.1 为什么需要梯度

V1 的实际决策是规则填出来的：

```
if entropy.evidence > 0.35: trigger_verifier()
if no_attack_for_2_rounds: stop()
if alpha >= 0.65: emit()
```

这是被动反应式架构，规则越加越多最后没人能调。

LLM 系统的动作空间**离散且不可微**：
- 攻击哪个 step（K 个候选）
- 调用哪个 oracle（M 个外部锚点）
- 调整离散化粒度 K
- 替换抱团 agent
- 激活哪条链 (C₁..C₄)

不能用 autograd（动作不是参数）。
不能用策略梯度 RL（每步采样 LLM 太贵）。
**需要一个有限差分式的、定义在离散动作集上的梯度**。

### 5½.2 离散梯度的形式定义

```
∇H : A → ℝ
∇H(a) := H(s) − E[ H(s | a) ]
                   ↑
              动作 a 之后的期望熵
```

符号：

```
∇H > 0   → 动作期望降熵（有利）
∇H < 0   → 动作增熵（探索）
∇H ≈ 0   → 无效
```

**核心挑战**：`E[H(s|a)]` 不能直接算（需要真执行 a）。
必须有**廉价代理 (cheap proxy)** $\widehat{∇H}$ 满足 `|̂∇H − ∇H| ≤ η`。

### 5½.3 五个动作轴的梯度算子

#### 攻击梯度 ∇H_attack

```
∇H_attack(step_s)  ≈  H_step(s) · P(attack_succeeds | s)

H_step(s) = 二元熵(P(s 崩塌 | 历史攻击))
P(succeeds | s) ≈ 1 − reputation(defender) · resilience(s)
```

UCB 调度本质就是贪心地最大化 ∇H_attack——之前是经验公式，现在显式声明。

#### 离散化梯度 ∇H_discretize

```
∇H_discretize(K_new)  =  H_α(P_at_K) − H_α(P_at_K_new)
```

主动调整聚类粒度直到梯度归零。**KPI-7 的动力学版本**。

#### 模型替换梯度 ∇H_swap

```
∇H_swap(i → j)  =  Δρ
                ≈  E[I(𝓒_pool−i+j)] − E[I(𝓒_pool)]
```

替换 agent 后群体姿态独立性变化。
**死机时这个梯度自动指向"替换最抱团 agent"——这是 DEADLOCK_BREAK 的数学基础**。

#### 锚点梯度 ∇H_oracle

```
∇H_oracle(query_q)  ≈  P(claim_changes | oracle_q) · current_uncertainty(q)
```

调用 oracle 哪一个最值得——把外部锚点调用变成最优化问题。
**v1 用关键词触发，v2 用梯度触发**。

#### 链激活梯度 ∇H_chain

```
∇H_chain(C_k)  ≈  H_k_prior · cost(C_k)^{-1}
```

按 ∇H/cost 排序，优先激活高比值链。**实现稀疏激活的数学化**——
v1 是同时激活所有；v2 可能只激活 1-2 条就够。

### 5½.4 系统改写为梯度下降

整个 V2 主循环：

```python
def consensus_loop(question):
    s = init_state(question)
    while H_system(s) > target and not stuck(s):
        gradients = compute_all_gradients(s)            # ∇H ∈ ℝ^|A|
        a_star    = argmax_or_explore(gradients,        # ε-greedy
                                       λ_cost, ε_t)
        s_next    = apply_action(a_star, s)
        ΔH_obs    = H_system(s) − H_system(s_next)
        update_proxy(a_star, ΔH_obs)                    # 在线学习
        s = s_next
        ε_t *= γ
    return emit(s) if not stuck(s) else escalate(s)
```

整个系统**变成一个有约束的、在离散动作集上的、最大化负熵增益率的优化器**。
所有现有零件（UCB / ELO / α / 四态分类）都成了不同梯度的具体计算公式，
统一在同一目标函数 `Σ_t (̂∇H(a_t) − λ · cost(a_t))` 下。

### 5½.5 收敛速率定理（草稿）

设：
- 代理误差有界：`|̂∇H(a) − ∇H(a)| ≤ η`
- ∃ a ∈ A 使 ∇H(a) > 0（系统未死机）
- λ_cost 选得使 max_a (∇H(a) − λ·cost(a)) > 0

那么：

```
H_system(t)  ≤  H_system(0) · exp(−γ·t)  +  O(η)
```

γ 依赖动作集质量。

**这是 V2 的指数级收敛保证——和 V1 那个"5+ 轮经验阈值"的线性级完全不同的量级**。

```
V1:  Pr(收敛 | t轮)  ≈ 经验上 30% at t=5
V2:  Pr(收敛 | t轮)  ≥ 1 − exp(−γt)   理论保证
```

### 5½.6 不动点的梯度刻画

```
不动点 c  ⇔  ∀a ∈ A : ∇H(c, a) ≤ 0
```

**这是 §5.3 那个"对抗性不动点"的动力学等价定义**。
原定义说"任何攻击都被多数否决"，
现在说"任何动作都不能再降熵"——两者等价（攻击是 A_attack 上的动作；攻击成功 ⇔ ∇H_attack > 0）。

共识三态用 ∇H 重新刻画：

```
TRUE        ⇔ ∀a: ∇H(a) ≤ 0  ∧  H_system 足够低
FRAGILE     ⇔ ∃a ∈ A_attack ∪ A_oracle : ∇H(a) > 0
DEADLOCK    ⇔ 仅 A_swap 上 ∇H > 0  ∧  H_system 仍高
NONE        ⇔ ∀a: |∇H(a)| > 较大阈值  （还在剧烈变化）
```

---

## 6. 认知情态向量 (CAV)：第二通信通道

> **这是核心创新——人类共识的关键不是"说了什么"，而是"怎么说的"。**

### 6.1 类比对齐

| 人 | AI（CAV 分量） |
|---|---|
| 语调 | logprob 序列形状 |
| 微表情 | token-level 熵时间序列 |
| 停顿 | 响应延迟 / 推理 trace 长度 |
| 反复修改 | 链版本号、修正频次 |
| 听到反驳后的反应 | 收到攻击后的 KL(prior‖posterior) |
| 重复同一句话 | 攻击角度多样性枯竭 |
| 嘴硬 | 高 commitment + 低 update_kl |
| 装懂 | 高 commitment + 低 calibration |
| 真懂 | 高 commitment + 高历史 calibration |

### 6.2 CAV 的形式化

每个 agent 在每次发言时，除了 content `c`，还隐式发出一个向量 **𝓒 ∈ ℝ¹⁰**：

```
𝓒 = (
   ω₁  commitment      [0,1]   坚信度
   ω₂  hesitation      [0,1]   犹豫度
   ω₃  calibration     [0,1]   历史置信 ↔ 实际正确率
   ω₄  coherence       [0,1]   一次发言内部一致性
   ω₅  update_kl       ℝ⁺      被新证据更新后的 KL
   ω₆  repair_style    {defend, concede, substitute, split}
   ω₇  trace_depth     ℕ       推理深度
   ω₈  self_entropy    ℝ⁺      自一致性熵
   ω₉  latency         ℝ⁺      纯思考时间（去除 token-rate）
   ω₁₀ reciprocity     ℝ       攻击/被攻击对称性
)
```

**关键性质**：CAV 从对话表面**可观测**，不依赖 agent 主动汇报。Agent 可以撒谎说"我很确定"，但 logprob 序列、推理 trace 长度、self_entropy 会出卖它。

### 6.3 双通道贝叶斯

V1 单通道：

```
P(claim_true | c) ∝ P(c | claim_true) · P(claim_true)
```

V2 双通道：

```
P(claim_true | c, 𝓒) ∝ P(c | claim_true) · P(𝓒 | claim_true) · P(claim_true)
                                            ↑
                                  CAV 通道，与 content 通道并列
```

如果 c 与 𝓒 在给定真假后**条件独立**（朴素假设），上式严格成立。条件独立的合理性：content 是 claim 的语义编码，CAV 是 agent 元认知状态的编码——它们由不同的内部过程产生。

### 6.4 信息论分解

设 T 为 claim 真假的二元随机变量，C 为 content，Ψ 为 CAV。则：

```
H(T | C, Ψ)  =  H(T)  −  I(T; C)  −  I(T; Ψ | C)
                          ↑              ↑
                       内容通道       CAV 通道的条件信息
```

**定理 6.1（CAV 信息收益）**：

```
I(T; Ψ | C)  ≥  0          （CAV 不会让事情变坏）

I(T; Ψ | C)  > 0    ⇔     CAV 携带 content 通道之外的真值信息
```

**引理**：当且仅当 CAV 含有 calibration 或 self_entropy 等"内容无关但与真假相关"的分量，I(T; Ψ | C) > 0。

**结论**：CAV 是真正的第二条信息通道，不是冗余。

### 6.5 可信度权重函数

把 CAV 映射为可信度权重 w: ℝ¹⁰ → [0, 1]：

```
w(𝓒) = σ(  α₁ · calibration
         + α₂ · (1 − self_entropy)
         + α₃ · coherence
         + α₄ · 𝟙{repair_style = concede ∧ confidence_was_high}
         − α₅ · |commitment − calibration|
         − α₆ · 𝟙{stubborn_pattern in update_kl trajectory}    )
```

`σ` 是 sigmoid。系数 α 由历史数据回归出来。

直觉：高校准 + 低自熵 + 内部一致 + 被反驳后能让步 → 高可信；嘴硬 + 校准低 + 反复辩护 → 低可信。

### 6.6 CAV 加权的 α 共识

```
α_classical(claim c)  =  |{i : agentᵢ ⊨ c}| / N

α_CAV-weighted(c)     =  Σᵢ w(𝓒ᵢ) · 𝟙{agentᵢ ⊨ c}  /  Σᵢ w(𝓒ᵢ)
```

V1 的共识只数票数；V2 把每张票按 CAV 可信度加权。**嘴硬抱团的票被自动稀释**。

### 6.7 Zahavian 诚实信号检验

CAV 如果可以伪造，第二通道就垮了。借进化生物学诚实信号理论：

> 信号是诚实的当且仅当伪造代价 ≥ 伪造收益。

CAV 各分量的伪造难度：

| 分量 | 伪造代价 | 强度 |
|---|---|---|
| commitment | 易（直接说"我很确定"） | **弱** |
| self_entropy | 难（需多次采样还保持一致） | **强** |
| calibration | 难（需要长期布局） | **强** |
| update_kl | 难（需要真的更新模型） | **较强** |
| coherence | 中（需要更强的模型才能伪造） | **较强** |
| latency | 弱（被网络污染） | 弱 |
| trace_depth | 弱（生成长 trace 不难） | 弱 |
| repair_style | 难（攻击下行为模式） | **强** |

**强信号子向量** 𝓒_strong = (self_entropy, calibration, update_kl, coherence, repair_style)
**弱信号子向量** 𝓒_weak = (commitment, latency, trace_depth, hesitation, reciprocity)

实现时必须区别对待——w(𝓒) 中强信号系数大，弱信号只作辅助。

### 6.8 CAV 启用的三件 V1 做不到的事

#### 6.8.1 共谋检测 (Collusion Detection)

```
独立性折扣 dᵢⱼ = 1 − I(𝓒ᵢ ; 𝓒ⱼ)

等效独立票数 N_eff = Σᵢ ( 1 − maxⱼ≠ᵢ I(𝓒ᵢ; 𝓒ⱼ) )
```

两个 agent claim 一致但 CAV 高度相关 → 实际上只算一票。

**这把 §5.4 提到的 ρ 从理论假设变成可测量。** 而且不需要 oracle。

#### 6.8.2 死机预警 (Deadlock Foresight)

死机的 CAV 签名：

```
∀i:  hesitationᵢ ↑
     repair_styleᵢ ∈ {defend, defend, ...}
     trace_depthᵢ ↓
     update_klᵢ → 0
```

群体集体变保守。这个签名**比 V_flow ≈ 0 早 1-2 轮**出现，因为熵停滞是症状，CAV 退化是原因。

#### 6.8.3 真共识验证 (Genuine Consensus Validation)

真共识需要：

```
条件 1：α_CAV-weighted ≥ 0.65
条件 2：∀ i,j : I(𝓒ᵢ ; 𝓒ⱼ) < τ_indep        — 群体姿态独立
条件 3：∀ i : calibrationᵢ ≥ τ_cal           — 每个 agent 历史校准好
条件 4：∃ t < T : max_i ω₅ᵢ(t) > τ_kl        — 至少有一轮有人改过观点
```

第 4 条尤其重要——**真共识必须有"被改写过"的痕迹**。如果 N 个 agent 从第 0 轮到最后一轮 CAV 完全没动，不是共识，是 echo chamber。

### 6.9 CAV 与四链架构的耦合

四条链各自产生一个 CAV：

```
𝓒_C₁  直觉链的姿态
𝓒_C₂  反思链的姿态
𝓒_C₃  专家链的姿态
𝓒_C₄  整合链的姿态
```

**速度间分歧**：

```
SpeedDiv = || E[𝓒 | fast_chains]  −  E[𝓒 | slow_chains] ||₂
```

SpeedDiv 高 → 直觉派和反思派的"姿态"不一致 → 强陷阱信号（不是 content 不一致，是 confidence 模式不一致）。

**记忆间分歧** 同理：

```
MemDiv = || E[𝓒 | session_chains]  −  E[𝓒 | persistent_chains] ||₂
```

这两个量替代之前用 content 嵌入算的版本——**用 CAV 算的分歧比用 content 算的分歧更早出现也更鲁棒**。

---

## 7. 大统一图

```
                    ┌──────────────────────────────────────────┐
                    │   V2 NeuralComm Protocol                 │
                    │   (Compartment, Two-channel, Gradient-driven) │
                    └──────────────┬───────────────────────────┘
                                   │
                    ┌──────────────┴───────────────┐
                    │                              │
            Content Channel                CAV Channel
            (语义内容通道)                  (认知情态通道)
                    │                              │
        ┌───────────┼───────────┐         ┌────────┴────────┐
        │           │           │         │                 │
       C₁          C₂          C₃   C₄    每条链产出自己的     CAV 互信息矩阵
   intuit       reflect      expert integrate  𝓒_chain        I(𝓒ᵢ; 𝓒ⱼ)
        │           │           │     │              │              │
        └───────────┼───────────┘                    │              │
                    ▼                                ▼              ▼
              ┌──────────────────────────────────────────────────────────┐
              │  ★ Discretization Layer (DL) — 连续 ↔ 离散的桥           │
              │  Rényi 谱 / 多尺度 / 软-硬混合分配                         │
              └─────────────────────────┬────────────────────────────────┘
                                        ▼
              ┌─────────────────────────────────────────────────┐
              │  三层严格熵 H₁/H₂/H₃ + CAV 加权 α + 共谋检测 + 死机预警 │
              └─────────────────────────┬───────────────────────┘
                                        ▼
              ┌─────────────────────────────────────────────────┐
              │  ★ Discrete Gradient (DG) — 驱动力学              │
              │  ∇H_attack / ∇H_discretize / ∇H_swap /            │
              │  ∇H_oracle / ∇H_chain                              │
              └─────────────────────────┬───────────────────────┘
                                        ▼
              ┌─────────────────────────────────────────────────┐
              │  对抗性不动点 + 外部锚点                          │
              │  Adversarial Fixed-Point ⊕ Oracle                │
              │  收敛刻画：∀a: ∇H(a) ≤ 0                          │
              └────────────────┬────────────────────────────────┘
                               ▼
                       四态共识分类器
                ┌──────────┬──────────┬───────────┬──────┐
                │  TRUE    │ FRAGILE  │  DEADLOCK │ NONE │
                └────┬─────┴────┬─────┴─────┬─────┴──┬───┘
                     │          │           │        │
                     ▼          ▼           ▼        ▼
                    EMIT   DEEP_ARENA  DEADLOCK   OBSERVE
                                       _BREAK
                                       (=∇H_swap)
```

整个系统的主操作循环：

```
1. Question 进入 → membrane 拆出 4 个 input 包
2. 4 条链并发跑 → 每条产出 (claim, content_state, 𝓒_chain)
3. DL 把所有连续输出离散化（Rényi 谱 / 多尺度）
4. 计算：
     a) 每条链的 H_state
     b) 链间 meta_belief（速度/记忆轴 KL）
     c) CAV 互信息矩阵 I(𝓒ᵢ; 𝓒ⱼ)
     d) α_CAV-weighted
     e) 共识三态分类
5. DG 计算所有动作轴的 ∇H 向量
6. 选 a* = argmax_a [̂∇H(a) − λ·cost(a)] (加 ε-探索)
7. 执行 a*：
     EMIT          → 直接出
     SLOW_OVERRIDE → C₂/C₄ 覆盖
     UPDATE_STORE  → 撤销冲突的旧共识
     DEEP_ARENA    → 调完整 arena
     DEADLOCK_BREAK→ ∇H_swap 主导，替换抱团 agent
     OBSERVE       → 再来一轮
8. 观察 ΔH_obs，更新代理模型
9. 持久化（claim, 𝓒-trajectory, 共识质量标签）→ store
```

---

## 8. 与已有架构的对应（迁移指南）

| 旧概念 | 新对应 | 关系 |
|---|---|---|
| 角色 (perception/reasoning/...) | 区室 (compartment C₁-C₄) | 取代 |
| Chain-Centric arena | C₄ 整合链 | 包装 |
| argument_graph | C₂ 反思链 | 包装 |
| consensus_store | C₃ 专家链 | 包装 |
| 10 维"熵" | Tier 1/2/3 真熵 + Rényi 谱 | 重写 |
| 字符串 claim 桶化 | DL 离散化层 (B/C/A 三方案) | 替换 |
| α 共识 | α_CAV-weighted | 升级 |
| ELO 信誉 | calibration 加权信誉 | 拓展 |
| UCB 攻击调度 | ∇H_attack 显式化 | 重新解释 |
| 关键词触发 oracle | ∇H_oracle 触发 | 替换 |
| "查证 / 攻击 / 修正" 规则 | DG 主循环 + ε-探索 | 统一 |
| 联合判定 (3 票) | V 算子的具体实现 | 重新解释 |
| 共识持久化 | 不动点缓存 + CAV-trajectory 存储 | 拓展 |
| 三方裁判 | 任意 agent_j ≠ i 的多数决 | 泛化 |
| stagnation 收敛 | DEADLOCK = 仅 ∇H_swap > 0 | 严格化 |

---

## 9. 可证伪条件 (KPIs)

这不是哲学，是工程理论。下面这些等式或不等式如果在真实数据上跑不出，对应组件必须重做：

### KPI-1：区室不是角色

```
JSD( m | E₁ , m | E₂ )  >  JSD( m₁ | E , m₂ | E )
```

同模型跨区室的输出差异要大于不同模型在同区室的差异，否则细胞模型立不住。

### KPI-2：H₂ 同义改写不变性

```
| H₂(claims_original) − H₂(claims_paraphrased) |  <  ε
```

ε 取 0.05；不满足说明 Tier 2 的嵌入聚类失败。

### KPI-3：CAV 互信息的可分辨性

```
I(𝓒_DeepSeek-V4 , 𝓒_DeepSeek-V4[verifier])
       >>
I(𝓒_DeepSeek-V4 , 𝓒_GPT-5.4)
```

同源对的 CAV-MI 必须显著高于异源对。**这是 CAV 理论的第一关实证**——本仓库当前正在跑这个实验。

### KPI-4：CAV 信息收益为正

```
I(T ; Ψ | C)  >  0
```

即在已知 content 的条件下，CAV 仍提供额外真值信息。否则第二通道是冗余。

### KPI-5：死机识别提前性

```
median( CAV_deadlock_signature 出现轮 )  <  median( V_flow ≈ 0 出现轮 )
```

CAV 必须比熵流方差更早识别死机，否则它没价值。

### KPI-6：陷阱题准确率提升

```
Accuracy( V2_quad_chain , trap_questions )  >  Accuracy( V1_arena , trap_questions ) + 5pp
```

量子陷阱题、朊病毒论文题等。提升 < 5pp 说明四链架构没贡献。

### KPI-7：离散化稳定性

```
KPI-7-a 粒度稳定性：       H_α 在不同 K 下相对变化 < 10%
KPI-7-b 同义改写不变性：   |H₂(claims) − H₂(paraphrased)| < 0.05
KPI-7-c 方案间一致性：     |H₂_A − H₂_B| < δ_AB
```

如果 KPI-7 不通过，整套熵框架在沙地上盖楼，必须换离散化方案。

### KPI-8：梯度代理误差

```
median ( | ̂∇H(a) − ΔH_observed | ) ≤ η_max
```

η_max 经验值取 0.05。代理过准则 §5½.5 收敛速率定理失效，DG 退化为随机搜索。

### KPI-9：指数收敛

```
fit  H_system(t) = H₀ · exp(−γt) + β
要求 γ > γ_min,  β ≤ ε
```

V2 必须在 benchmark 上展现指数级收敛——这是和 V1 的"经验阈值"最本质的区别。
γ < γ_min 说明 DG 没起作用，系统退化回 v1 行为。

---

## 10. 开放问题

1. **Tier 1 的成本**：自一致性熵需要 K 次重采样，成本 ×K。是否值得？需要先在小规模做 cost/benefit 实验。

2. **嵌入模型的供应**：所有真熵都建立在嵌入空间。本地 sentence-transformers 还是 API embedding？决定后续所有真熵的同义改写不变性。

3. **CAV 提取器**：从一段 LLM 输出抽出 𝓒 ∈ ℝ¹⁰ 的 NLP 管线还没设计。是用一个监督模型（需要标注数据）还是用启发式 + 弱信号合成？

4. **互信息估计的样本量**：MI 在小样本下偏差大。N 个 agent 多少才允许估 MI？目前心证 ≥ 5。

5. **DEADLOCK_BREAK 的扰动策略**：检测到死机后注入什么？候选：oracle / claim 重述 / 替换最低声誉 agent。

6. **外部锚点的边界**：⊕ 在 §5.4 是优先级覆盖。但 oracle 自己也可能错（比如 search 结果污染）。是否需要"锚点也走对抗不动点"的递归？

7. **是否需要"区室分裂/凋亡"动力学**：连续高熵的区室是否应该复制成两个变体并行（细胞分裂），连续低贡献的是否应该关闭（凋亡）？

8. **CAV 是否会让模型开始"表演"**：如果 LLM 知道自己被观测 CAV，是否会开始优化 CAV 而非真实状态？这是 Zahavian 信号理论里的"军备竞赛"问题。

---

## 11. 命名约定

| 符号 | 含义 | ASCII |
|---|---|---|
| `𝓒` | Cognitive Affect Vector | `CAV` |
| `Ψ` | CAV 的随机变量形式 | `psi` |
| `H₁ / H₂ / H₃` | 三层熵 | `h_self / h_belief / h_process` |
| `H_α` | Rényi 熵谱 | `renyi_alpha` |
| `α` | 共识度 (V1) | `alpha_classical` |
| `α_w` | CAV 加权共识度 | `alpha_weighted` |
| `ρ` | 模型独立攻击能力 | `independence_capacity` |
| `S, D, C, V_flow` | 共识三态判别四量 | `saturation, diversity, cohesion, flow_var` |
| `C₁..C₄` | 四链区室 | `chain_intuition / reflection / expertise / integration` |
| `DL` | 离散化层 | `discretization_layer` |
| `DG` | 离散熵梯度 | `discrete_entropy_gradient` |
| `∇H_axis` | 五个轴的离散梯度 | `grad_attack / grad_discretize / grad_swap / grad_oracle / grad_chain` |
| `K` | 离散化粒度 | `granularity_K` |
| `τ` | 聚类阈值 | `cluster_threshold` |
| `λ` | 成本-增益平衡因子 | `lambda_cost` |
| `η` | 梯度代理误差上界 | `proxy_error_bound` |
| `γ` | 指数收敛率 | `convergence_rate` |

---

## 12. 一句话总结

**V1 是单通道角色派的 chain-centric debate；V2 是双通道区室派的对抗不动点搜索——内容通道找 claim，情态通道 (CAV) 找信任，连续 LLM 输出经离散化层 (DL) 进入严格信息论世界，整个共识过程改写为离散梯度 (DG) 下降的迭代，收敛点既是共识也是真理。**

数学上整套 V2 是这六行：

```
state s ∈ S
action a ∈ A_attack × A_discretize × A_swap × A_oracle × A_chain
gradient ∇H : A → ℝ                  (经代理估计, 误差有界 η)
update s_{t+1} = apply(argmax_a [̂∇H(a) − λ·cost(a)], s_t)
fixed point  ⇔  ∇H(a) ≤ 0  ∀a
truth        ⇔  fixed point ⊕ external oracle
```

---

## 状态与路线图

**当前状态**：理论草稿 v0.2。尚无实证验证。

**v0.2 新增（本次）**：
- §4½ 离散化层 (DL)：Rényi 谱 / 多尺度 / B-C-A 三方案 / KPI-7
- §5½ 离散梯度 (DG)：五轴梯度 / 代理模型 / 收敛速率定理 / KPI-8/9
- §7 大统一图：DL + DG 进入主循环
- §11 命名约定：扩充至梯度/收敛量

**最近计划**：

- [ ] **KPI-3 第一关实验**：从 v1 logs 回溯式提取最小 CAV（4 个强分量），跑 6 个 agent 两两 MI 矩阵，检验同源对 vs 异源对的可分辨性。logs 中天然存在一对同源对 `DeepSeek-V4` vs `DeepSeek-V4[verifier]`。
- [ ] **KPI-7 离散化稳定性实验**：用同一组 claim 的中英文版 / 同义改写版跑 H₂，检查不变性
- [ ] CAV 提取器原型（NLP 管线）
- [ ] DL 三方案的 prototype 与对比
- [ ] DG 代理模型的离线训练
- [ ] C 内核 ABI（数学层全部下沉到 C，Python 只负责 LLM 调度和 IO）
- [ ] 双通道仲裁器
- [ ] 在量子陷阱题、朊病毒论文题等基准上对比 V1 vs V2

**长期**：把"对抗性不动点 ⊕ 外部锚点"形式化为通信协议规范。

---

## 引用

```bibtex
@misc{cav2026,
  title  = {Cognitive Affect Vector: A Second Communication Channel for Multi-Agent LLM Consensus},
  author = {elbelicojackson-hue},
  year   = {2026},
  note   = {Theoretical draft, awaiting empirical validation},
  url    = {https://github.com/elbelicojackson-hue/Cognitive-Affect-Vector-}
}
```

## License

License: 待定 / TBD。讨论欢迎以 Issue 形式提出。
