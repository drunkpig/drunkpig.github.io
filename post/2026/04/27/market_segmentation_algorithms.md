# K 线转折点检测算法对比：从 BEAST 到 psimpl 八种折线简化

> 生成时间：2026-04-27
> 测试数据：上证指数 30 分钟 K 线（`sh000001` 1m 聚合）
> 测试窗口：3 段代表性行情（2 周左右）
> 源代码：
> - [`celue/psimpl_all_algos.py`](https://github.com/drunkpig/tdx-signal-viewer/blob/main/celue/psimpl_all_algos.py) — 9 个折线简化算法实现 + 对比图生成
> - [`celue/polysimplify.py`](https://github.com/drunkpig/tdx-signal-viewer/blob/main/celue/polysimplify.py) — Visvalingam-Whyatt 实现（来自 Permafacture/Py-Visvalingam-Whyatt，MIT）
> - [`celue/beast_turning.py`](https://github.com/drunkpig/tdx-signal-viewer/blob/main/celue/beast_turning.py) — BEAST 季度变点检测
> - [`celue/psimpl_turning.py`](https://github.com/drunkpig/tdx-signal-viewer/blob/main/celue/psimpl_turning.py) — 最早的 PD / RW 单算法实验
> - [`celue/psimpl_tol_compare.py`](https://github.com/drunkpig/tdx-signal-viewer/blob/main/celue/psimpl_tol_compare.py) — RW 不同 tol 值的粒度对比

---

## 背景：为什么要检测 K 线转折点？

在个股 Signal 1 策略回测中发现：**当大盘同向时胜率明显高，反向时亏损居多**。要把"大盘跟随"做成过滤器，需要一个算法能实时告诉我：

- 当前大盘处于什么趋势段
- 刚刚有没有发生"方向翻转"

这就是典型的**时序分段 / 转折点检测（Changepoint Detection）** 问题。要求：

1. **在线算法** — 只能用当前和过去的 bar，不能用未来数据
2. **转折点稳定** — 今天确认的转折点，明天不要又被改写
3. **延迟可控** — 转折确认最多晚 1~2 根 bar
4. **参数少、易调** — 一个阈值就能换档

下文对比 8 个来自经典折线简化库 [psimpl](https://psimpl.sourceforge.net/) 的算法，以及作为对照组的 BEAST 贝叶斯变点检测。

---

## 算法全景

### 完整清单

| # | 算法 | 一句话思路 | 在线? | 本文实现 |
|---|---|---|---|---|
| 1 | **Nth Point**       | 每隔 N 个点保留一个（最朴素） | ✅ | ✅ |
| 2 | **Radial Distance** | 距上一保留点 < r 的都删掉       | ✅ | ✅ |
| 3 | **Perpendicular Distance (PD)** | b 到 ac 线段垂距 < tol 删掉，迭代直到稳定 | ⚠️ 迭代 | ✅ |
| 4 | **Reumann-Witkam (RW)** | 锚点→下一点方向建走廊，冲出时前一点成新锚点 | ✅ | ✅ |
| 5 | **Opheim**          | RW 变体：走廊 + 径向距离双约束 | ✅ | ✅ |
| 6 | **Lang**            | 滑动窗口 [i, i+k]，检查窗内所有点到弦的垂距 | ✅ 有 k 窗延迟 | ✅ |
| 7 | **Douglas-Peucker (DP)** | 递归分治：找整段最大偏离点分裂 | ❌ 离线 | ✅ |
| 8 | **Douglas-Peucker-N** | DP 变体，限制最多保留 N 个点 | ❌ 离线 | ✅ |
| 9 | **Visvalingam-Whyatt (VW)** | 按**三角形面积**排序，面积小的点删掉 | ❌ 离线 | ✅ |
| 10 | **BEAST**          | 贝叶斯 MCMC 变点模型 | ❌ 离线 | ✅（对照） |

### 在线 / 离线 对比

| 算法 | 在线 | 转折点稳定 | 确认延迟 | 参数难度 | 适合实盘 |
|---|---|---|---|---|---|
| BEAST            | ❌ | — | — | 中 | ❌（依赖未来） |
| Nth Point        | ✅ | ✅ 固定间隔 | 0 | 无（只有 N） | ⚠️ 太死板 |
| Radial Distance  | ✅ | ✅ 永不改写 | 0 | 简单 | ⚠️ 仅作滤波 |
| Perpendicular D. | ⚠️ | ❌ 每轮会重排 | 不稳定 | 难 | ❌ |
| Reumann-Witkam   | ✅ | ✅ 永不改写 | 1 bar | 简单（只有 tol） | ✅ |
| Opheim           | ✅ | ✅ 永不改写 | 1 bar | 中（dmin+dmax） | ✅ |
| Lang             | ✅ | ✅ 永不改写 | k bar | 中（tol+窗口 k） | ✅ |
| Douglas-Peucker  | ❌ | — | — | 简单 | ❌（需整段） |
| DP-N             | ❌ | — | — | 简单（点数） | ❌（需整段） |
| Visvalingam-Whyatt | ❌ | — | — | 简单（面积或点数） | ❌（需整段） |

---

## 同窗口 9 算法大对比图

同一段 K 线，9 种算法分割结果上下叠放，直观看到各自粒度差异：

![2024年9月底大涨](market_seg/30m_ALL_compare_2024-09-18_2024-10-11.png)

![2024年1月暴跌+2月反弹](market_seg/30m_ALL_compare_2024-01-15_2024-02-08.png)

![2024年7月下跌](market_seg/30m_ALL_compare_2024-07-15_2024-08-07.png)

---

## 逐算法详解

> 下面 9 个算法统一使用 **2024-09-18 ~ 2024-10-11** 这段"9 月底大涨"行情作为样例（30m × 104 根 bar），方便横向对比。
> 优缺点重点评估两件事：**是否需要未来数据**、**新 bar 到来时转折点会不会被改写**——这两点直接决定能否用作实盘信号。

### 1. Nth Point（定步长抽样）

**思路**：从第 0 根 bar 开始每隔 N 根保留一个，首末点必保留。最原始的降采样，等价于均匀下采样。

- **参数**：步长 N（本文 N=8，对应 ~1 交易日）

![Nth Point 9月大涨](market_seg/30m_nth_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ❌ 不需要，严格在线 |
| 新 bar 会改写历史转折？ | ❌ 不会，已保留的点永不变动 |
| 确认延迟 | 0 bar |
| **能做大盘跟随信号吗？** | **不能** |

**原因**：这个算法**完全忽略价格**——不管是一天暴涨 3% 还是窄幅盘整，都统一每 8 根取一个。对策略毫无信息量，只能作为降采样 baseline 用来对照其他算法。

---

### 2. Radial Distance（径向距离）

**思路**：保留首点；遍历后续点，只要与"上一个保留点"的**径向（欧氏）距离** ≥ r，就保留。

```
dist = sqrt((i - last_x)^2 + (prices[i] - last_y)^2)
```

- **参数**：半径 r（本文 r=15 指数点）

![Radial Distance 9月大涨](market_seg/30m_radial_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ❌ 不需要，严格在线 |
| 新 bar 会改写历史转折？ | ❌ 不会，单向扫描 |
| 确认延迟 | 0 bar |
| **能做大盘跟随信号吗？** | **不能直接做，但可作预处理** |

**原因**：只看"点与点距离"不看"形状"——缓慢上涨 100 点 vs 暴跌 100 点输出相似的点数分布，无法区分趋势方向。可以作为 RW / Lang 之前的第一道降噪（先过滤掉微小抖动），但单独无法触发交易信号。

---

### 3. Perpendicular Distance（垂直距离，迭代）

**思路**：对每 3 个相邻保留点 (a, b, c)，算 b 到线段 ac 的垂直距离；若 < tol 则删除 b。**反复扫描直到没有点被删**。

- **参数**：垂距阈值 tol（本文 tol=10）

![PD 9月大涨](market_seg/30m_pd_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ⚠️ 严格说不需要，但行为不稳定 |
| 新 bar 会改写历史转折？ | ✅ **会！**这是致命缺陷 |
| 确认延迟 | 不确定 |
| **能做大盘跟随信号吗？** | **不能** |

**原因**：每次来新 bar 都要重新迭代到"稳定态"，而新 bar 可能改变中间某个三元组的垂距判定，导致**前几天已经识别的转折点在今天被撤销**。实盘中这等于"昨天说转折要开仓，今天又说没转折撤单"——信号反复，无法执行。理论在线但实际不可用。

---

### 4. Reumann-Witkam（走廊算法）⭐

**思路**：

1. 从锚点 A 出发，用 A→A+1 方向作为走廊的中轴，走廊宽度为 2·tol
2. 逐个检查 A+2, A+3, ... 是否在走廊内（点到中轴的垂距 ≤ tol）
3. 某点 J **冲出走廊** → J-1 成为新转折点和新锚点
4. 从 J-1 重新建走廊，继续向前

```
           走廊上界 ─────
                       ╲
  ●──●──●──●──●──●──●───●  ← 冲出走廊
   A    ↑
        J-1 (新锚点)
           走廊下界 ─────
```

- **参数**：走廊半宽 tol（本文 tol=10）

![RW 9月大涨](market_seg/30m_rw_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ❌ 不需要，严格前向 O(n) |
| 新 bar 会改写历史转折？ | ❌ **绝对不会** |
| 确认延迟 | 1 bar（冲出走廊的那根 bar 触发前一根确认） |
| **能做大盘跟随信号吗？** | **✅ 可以，本文首选** |

**原因**：算法数学上保证"锚点前移后永不回溯"——一旦某个 bar 被标记为转折点，无论后面来多少新数据都不会撤销。唯一的不确定性在"当前走廊"里最新那段，但这对交易而言也合理：新趋势段要等到它真正形成（下一根 bar 冲出走廊）才能确认。30 分钟的确认延迟对日内跟随策略完全可接受。

---

### 5. Opheim

**思路**：RW 的增强版。在走廊约束之外，再加"距锚点的**径向距离**不得超过 dmax"的硬约束；任一触发即切断。

- **参数**：走廊半宽 `dmin`、最大径向 `dmax`（本文 10 / 30）

![Opheim 9月大涨](market_seg/30m_opheim_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ❌ 不需要，继承 RW 在线特性 |
| 新 bar 会改写历史转折？ | ❌ 不会 |
| 确认延迟 | 1 bar |
| **能做大盘跟随信号吗？** | **✅ 可以，RW 的替代方案** |

**原因**：RW 在一段非常平缓的长趋势中可能把整段"吞掉"只给一个转折点，错过中间的小级别调整。Opheim 的 dmax 约束强制"每走过 dmax 距离就重新评估一次方向"，在震荡行情里比 RW 给出更细的分段，适合交易频率更高的场景。代价是多一个参数要调。

---

### 6. Lang

**思路**：维护一个**前瞻窗口 [i, i+k]**：

1. 计算窗内所有中间点到弦 i–(i+k) 的**最大垂直距离**
2. 若 max_d < tol → 删除窗内所有中间点，i 直接跳到 i+k
3. 否则 k-- 缩小窗口重试

- **参数**：垂距阈值 tol、窗口 k（本文 10 / 5）

![Lang 9月大涨](market_seg/30m_lang_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ⚠️ 需要窗口内 k 根 bar（本文 k=5） |
| 新 bar 会改写历史转折？ | ❌ 不会 |
| 确认延迟 | **k bar**（30m × 5 = 2.5 小时） |
| **能做大盘跟随信号吗？** | **⚠️ 勉强可以，但延迟偏大** |

**原因**：Lang 判断更稳健（看窗内最大偏离，不是首个冲出），抗噪性比 RW 好。但它需要等窗口内所有 k 根 bar 都到齐才能判断前一段是否结束——对 30m 周期就是 2.5 小时延迟，对交易时机敏感的场景偏慢。适合做**事后段落总结**（比如每天收盘后给当天的几段趋势贴标签），不适合做**实时开仓信号**。

---

### 7. Douglas-Peucker（经典，递归分治）

**思路**：

1. 连接首末两点为一条弦
2. 找段上**偏离弦最远**的点 P
3. 若 P 到弦的距离 > tol：保留 P，以 P 为分界递归处理 [起点, P] 和 [P, 末点]
4. 否则：段内所有中间点全删

- **参数**：垂距阈值 tol（本文 tol=10）

![DP 9月大涨](market_seg/30m_dp_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ✅ **是，完全需要未来数据** |
| 新 bar 会改写历史转折？ | ✅ **每来一根新 bar 都可能全盘改写** |
| 确认延迟 | 不适用 |
| **能做大盘跟随信号吗？** | **不能** |

**原因**：DP 从"整段的首末两点"开始切分，需要知道整段的结束位置。实盘中你永远不知道"整段"在哪结束——每来 1 根 bar 都会改变终点，从而改变最大偏离点的位置，导致之前所有转折点重新排布。这也是文献里 DP 被归类为"offline simplification"的根本原因。**只能用于事后复盘、给历史 K 线打段落标签**，不能产生交易信号。

---

### 8. Douglas-Peucker-N（限点数）

**思路**：DP 的变体。使用优先队列（最大堆）逐步执行分裂：

1. 初始：首末两点入列
2. 从堆里弹出"当前最大偏离段"，在偏离最大处分裂，保留分裂点
3. 重复直到保留点数达到 N

- **参数**：目标点数 N（本文 N=12）

![DP-N 9月大涨](market_seg/30m_dpn_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ✅ **是** |
| 新 bar 会改写历史转折？ | ✅ **会** |
| 确认延迟 | 不适用 |
| **能做大盘跟随信号吗？** | **不能** |

**原因**：和经典 DP 一样需要整段数据。但它适合**固定输出维度**的特征工程场景——比如要给每段 2 周的大盘行情生成一个 12 维的"关键转折点特征"喂给机器学习模型。这是回测 / 训练场景，不是实盘信号。

---

### 9. Visvalingam-Whyatt（面积权重）

**思路**：前面的 RW / Opheim / Lang / DP 都用"**距离**"作为点的重要性度量，VW 换了一个维度——用"**三角形面积**"。

1. 对每个中间点 `P[i]`，计算三角形 `(P[i-1], P[i], P[i+1])` 的**面积**（叫 effective area，有效面积）
2. 找面积最小的点，把它删掉（面积小 = 这个点几乎在直线上 = 不重要）
3. 删掉后，重算它**左右两个邻居**的有效面积（邻居的相邻点变了）
4. 反复直到最小面积 > 阈值，或剩余点数达到 N

这个算法有个漂亮的副产物：每个点都得到一个"effective area"分数，相当于**重要性排名**。可以一次计算后，在不同阈值下切换分辨率——低阈值看细节，高阈值看大局。GIS / 地图软件里被广泛使用（按 zoom level 切换折线粒度）。

- **库**：[Permafacture/Py-Visvalingam-Whyatt](https://github.com/Permafacture/Py-Visvalingam-Whyatt)（纯 Python，MIT）
- **参数**：面积阈值（本文 area_tol=30，单位 = bar × 指数点）

![VW 9月大涨](market_seg/30m_vw_2024-09-18_2024-10-11.png)

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ✅ **是**，计算 P[i] 的面积必须知道 P[i+1] |
| 新 bar 会改写历史转折？ | ✅ 会，末端三角形重算会触发优先级队列重排 |
| 确认延迟 | 不适用（离线） |
| **能做大盘跟随信号吗？** | **不能，但非常适合特征工程** |

**原因**：和 DP 一样，它需要知道整段序列才能排面积。但相比 DP-N，**VW 的重要性排序在"加数据"后更稳定**——因为面积是个局部量（只依赖 3 个相邻点），而 DP 的距离是全局量（依赖整段的首末点）。实际意义：如果你要给过去 2 周大盘生成"12 个最重要转折点"的特征向量，VW 比 DP-N 更合理。

**和 DP / DP-N 的区别**：

| 维度 | Douglas-Peucker | Visvalingam-Whyatt |
|---|---|---|
| 重要性度量 | 垂直距离（到整段弦） | 三角形面积（仅邻近 3 点） |
| 工作方向 | 递归自顶向下分裂 | 迭代自底向上合并 |
| 输出特性 | 单一 tol 阈值 | 每点一个"重要度分数"，支持多分辨率 |

---

### 10. BEAST（贝叶斯变点检测，对照组）

**思路**：用 MCMC 采样估计一个"分段线性 + 变点数量"的贝叶斯模型，输出每个位置的**变点概率**。

```python
import Rbeast as rb
o = rb.beast(Y, season='none', tcp_minmax=[0, 15],
             torder_minmax=[0, 1], mcmc_samples=8000)
cp_prob = o.trend.cpOccPr  # 每个点的变点概率
```

- **库**：[Rbeast](https://github.com/zhaokg/Rbeast)（`pip install Rbeast`）
- **参数**：最大变点数、趋势阶数、MCMC 采样数

![BEAST 2024Q3 30m](market_seg/sh000001_30m_2024Q3_beast.png)

> 注：BEAST 由于计算开销大（每季度 30s），这里用 2024Q3 的完整季度图代替同一 2 周窗口，同样能看到它识别的变点位置。

**用于选股场景的评估**：

| 维度 | 结论 |
|---|---|
| 需要未来数据？ | ✅ **是，MCMC 对整段序列采样** |
| 新 bar 会改写历史转折？ | ✅ **会，重跑结果可能不同** |
| 确认延迟 | 不适用 |
| **能做大盘跟随信号吗？** | **不能** |

**原因**：BEAST 输出的是**概率**，比 DP 的布尔值更有信息量，可以用置信度阈值过滤。但它本质还是离线批处理：每次新 bar 来了要从头重跑 8000 次 MCMC 采样，单次运行 ~30 秒。即便接受这个延迟，结果本身也会因为"采样序列变长"而重新分布概率，导致昨天的变点今天可能消失。**适合做研究**（比如统计分析去年所有确认变点的胜率），**不适合实盘**。

---

## 结论与选型

| 场景 | 推荐算法 | 理由 |
|---|---|---|
| **实盘趋势跟随**（如大盘 30m 方向过滤器） | **Reumann-Witkam** | 最简单、延迟 1 bar、转折点稳定 |
| **实盘但需更精细**（震荡行情） | **Opheim** / **Lang** | RW 加强版，控制最大段长 |
| **实盘第一道降噪** | **Radial Distance** | O(n)、无偏好、可作为预处理 |
| **事后复盘 / 回测打标签** | **Douglas-Peucker** | 段质量最高 |
| **ML 特征工程**（固定点数的关键点特征） | **Visvalingam-Whyatt** | 面积权重更稳定，附带重要度排名 |
| **结构性变点研究** | **BEAST** | 概率输出、统计可靠 |
| **不推荐** | Perpendicular Distance | 迭代收敛会改写历史，实盘不可用 |

**最终选择**：接下来把 **Reumann-Witkam（tol=10）** 接入 `strategy1_simulate.py` 作为大盘方向过滤器——只有当 RW 当前走廊方向向上时才允许开仓。

---

## 参考资料

- **psimpl 算法库**：<https://psimpl.sourceforge.net/>
- **polyline-simplification（Python 实现 + Jupyter 教程）**：<https://github.com/keszegrobert/polyline-simplification>
- **CodeProject 原始文章**：<https://www.codeproject.com/Articles/114797/Polyline-Simplification>（psimpl 与上述仓库的共同出处）
- **Reumann-Witkam 论文**：Reumann K., Witkam A.P.J. (1974). *Optimizing curve segmentation in computer graphics*.
- **Douglas-Peucker 论文**：Douglas D., Peucker T. (1973). *Algorithms for the reduction of the number of points required to represent a digitized line or its caricature*. [Cartographica 10(2):112–122](https://doi.org/10.3138/FM57-6770-U75U-7727)
- **Opheim 论文**：Opheim H. (1982). *Fast data reduction of a digitized curve*.
- **Lang 论文**：Lang T. (1969). *Rules for the robot draughtsmen*.
- **Visvalingam-Whyatt 论文**：Visvalingam M., Whyatt J.D. (1993). *Line Generalisation by Repeated Elimination of Points*. Cartographic Journal 30(1):46–51.
- **Py-Visvalingam-Whyatt（本文使用的 VW 实现）**：<https://github.com/Permafacture/Py-Visvalingam-Whyatt>
- **BEAST 项目**：<https://github.com/zhaokg/Rbeast>
- **BEAST 论文**：Zhao K. et al. (2019). *Detecting change-point, trend, and seasonality in satellite time series data to track abrupt changes and nonlinear dynamics: A Bayesian ensemble algorithm*. Remote Sensing of Environment.
- **折线简化综述**：Shi W., Cheung C. (2006). *Performance evaluation of line simplification algorithms for vector generalization*. The Cartographic Journal.
