# 供应链网络架构（Q2 传播研究闭合版）

> 目标：在保留 `M → P → M` 基本结构的前提下，使网络在给定初始状态与局部冲击后能够唯一演化，从而研究冲击的传播时间、传播范围、传播幅度与恢复过程。

---

# 1. 总体结构

网络分为四层：

\[
\boxed{
\text{生产拓扑}
\rightarrow
\text{静态物理参数}
\rightarrow
\text{动态状态}
\rightarrow
\text{传播过程}
}
\]

当前核心研究对象仍然是生产网络的物理传播机制，不引入价格、企业博弈、采购优化或恢复优化。

---

# 2. 基本生产网络

基本结构保持为：

\[
\boxed{
M\rightarrow P\rightarrow M
}
\]

其中：

\[
M=\text{Material-State-at-Location}
\]

表示某种物料在给定位置、给定状态下的可用库存。

\[
P=\text{Production Process}
\]

表示将一组输入物料转换为一组输出物料的生产工艺。

例如：

\[
M_A
\rightarrow
P_1
\rightarrow
M_B
\rightarrow
P_2
\rightarrow
M_C
\]

一个生产过程可以有多个必要输入：

\[
M_A\rightarrow P,
\qquad
M_B\rightarrow P,
\qquad
M_C\rightarrow P.
\]

一个物料库存也可以同时供给多个下游过程：

\[
M_A\rightarrow P_1,
\qquad
M_A\rightarrow P_2.
\]

因此网络允许 chain、fork、assembly 与 multi-source merge 等局部结构。

---

# 3. 多源生产网络

多源不新增基础节点类型。

若不同来源仍会影响后续可行路径、工艺兼容性、运输条件或约束，则保持为不同分支；若来源差异从某一点开始不再改变后续运行，则可以在该点合并。

来源历史可以保存在 lineage 属性中，例如：

\[
supplier,\quad plant,\quad batch,\quad origin.
\]

因此：

\[
\text{拓扑上的合并}
\neq
\text{删除来源历史}.
\]

对于 Q2 的传播实验，是否允许替代必须事先固定。第一阶段建议默认：

\[
\boxed{\text{无主动替代、无临时新增供应商}}
\]

避免把网络拓扑效应与企业决策混在一起。

---

# 4. 静态物理参数

## 4.1 Material 节点

每个物料节点定义为：

\[
m=(material,\ state,\ location)
\]

并保存库存容量：

\[
\boxed{\bar q_m}
\]

其中：

\[
\bar q_m=\text{该物料节点的最大库存容量}.
\]

---

## 4.2 Production 节点

每个生产过程 \(p\) 保存：

\[
\boxed{
P_p=
\left(
\bar c_p,\,
\tau_p^{prod},\,
u_p^0
\right)
}
\]

其中：

\[
\bar c_p=\text{名义最大生产能力},
\]

\[
\tau_p^{prod}=\text{生产 lead time},
\]

\[
u_p^0=\text{无冲击基准状态下的目标生产流率}.
\]

这里 \(u_p^0\) 不是企业优化决策，只是 Q2 基线实验的外生 reference flow。

为表示冲击后的运行能力，再定义：

\[
\boxed{
c_p(t)\leq \bar c_p
}
\]

其中 \(c_p(t)\) 是时刻 \(t\) 的有效生产能力。

正常状态：

\[
c_p(t)=\bar c_p.
\]

例如工厂停工：

\[
c_p(t)=0,
\qquad
t_0\leq t<t_0+L.
\]

---

## 4.3 输入边 \(M\rightarrow P\)

输入边保存投入系数：

\[
\boxed{a_{mp}}
\]

表示生产过程 \(p\) 每运行一个单位，需要消耗多少单位物料 \(m\)。

若：

\[
2A+3B\rightarrow C,
\]

则：

\[
a_{A,p}=2,
\qquad
a_{B,p}=3.
\]

---

## 4.4 输出边 \(P\rightarrow M\)

输出边保存产出系数：

\[
\boxed{b_{pm}}
\]

表示生产过程 \(p\) 每运行一个单位，产生多少单位输出物料 \(m\)。

若：

\[
2A+3B\rightarrow C,
\]

则：

\[
b_{p,C}=1.
\]

如果输出需要运输到下游库存，则该连接还保存：

\[
\boxed{
\bar c_e^{trans},
\qquad
\tau_e^{trans}
}
\]

其中：

\[
\bar c_e^{trans}=\text{名义运输能力},
\]

\[
\tau_e^{trans}=\text{运输 lead time}.
\]

与生产能力类似，定义运行时有效运输能力：

\[
\boxed{
c_e^{trans}(t)\leq \bar c_e^{trans}.
}
\]

正常状态：

\[
c_e^{trans}(t)=\bar c_e^{trans}.
\]

运输冲击可以通过降低 \(c_e^{trans}(t)\) 表示。

---

# 5. Delay

生产 delay 与运输 delay 分开：

\[
\boxed{
\tau_p^{prod},
\qquad
\tau_e^{trans}
}
\]

Capacity 描述单位时间最多能处理多少流量；Delay 描述已经进入某阶段的物料需要多久才能进入下一阶段。

一批物料的基本传播过程为：

\[
\text{available input}
\rightarrow
\text{production WIP}
\rightarrow
\text{finished-goods waiting queue}
\rightarrow
\text{in-transit}
\rightarrow
\text{downstream available inventory}.
\]

因此，物料离开上游库存以后，不会瞬间进入下游可用库存。

---

# 6. 动态状态

统一时间步长为：

\[
\boxed{\Delta t}
\]

时刻 \(t\) 的系统状态正式定义为：

\[
\boxed{
S(t)=
\left(
Q(t),\,
W(t),\,
H(t),\,
L(t)
\right)
}
\]

其中：

\[
Q(t)=\{q_m(t)\}
\]

为各物料节点的可用库存；

\[
W(t)=\{w_p(t,\ell)\}
\]

为生产中的 WIP pipeline；

\[
H(t)=\{h_e(t)\}
\]

为生产已经完成、但尚未进入运输的 finished-goods waiting queue；

\[
L(t)=\{\ell_e(t,r)\}
\]

为运输中的 in-transit pipeline。

其中 \(\ell\) 与 \(r\) 可以表示剩余生产时间或剩余运输时间。

只要存在非零生产或运输 delay，WIP 与 in-transit 就属于正式状态，而不是可选记录项。

初始状态必须给定：

\[
\boxed{
S_0=
\left(
Q(0),W(0),H(0),L(0)
\right).
}
\]

若实验从完全稳态开始，则可以通过 warm-up 生成一致的 \(W(0)\)、\(H(0)\) 与 \(L(0)\)。

---

# 7. 生产执行规则

这一节用于使系统动力学闭合。

## 7.1 候选生产流率

首先定义过程 \(p\) 在不考虑即时库存竞争时希望执行的候选流率：

\[
\boxed{
\tilde u_p(t)
=
\min
\left(
u_p^0,\,
c_p(t)
\right).
}
\]

因此：

\[
0\leq \tilde u_p(t)\leq \bar c_p.
\]

---

## 7.2 输入需求

过程 \(p\) 在一个时间步内对物料 \(m\) 的候选需求为：

\[
\boxed{
R_{mp}(t)
=
a_{mp}\tilde u_p(t)\Delta t.
}
\]

物料 \(m\) 面对的总候选需求为：

\[
\boxed{
R_m(t)
=
\sum_{p:m\rightarrow p}
R_{mp}(t).
}
\]

---

# 8. 库存不足时的分配规则

为了避免同一个库存同时被多个下游过程重复使用，第一阶段采用确定性的 proportional rationing。

对每个物料 \(m\)，定义库存满足比例：

\[
\boxed{
\rho_m(t)
=
\begin{cases}
1, & R_m(t)=0,\\[1mm]
\min\left(1,\dfrac{q_m(t)}{R_m(t)}\right), & R_m(t)>0.
\end{cases}
}
\]

对于需要多个输入的生产过程，所有必要输入中最紧的那个决定实际执行比例：

\[
\boxed{
\rho_p(t)
=
\min_{m:m\rightarrow p}
\rho_m(t).
}
\]

于是实际生产流率为：

\[
\boxed{
u_p(t)
=
\tilde u_p(t)\rho_p(t).
}
\]

并满足：

\[
0\leq u_p(t)\leq c_p(t)\leq\bar c_p.
\]

实际从物料 \(m\) 消耗：

\[
\boxed{
C_{mp}(t)
=
a_{mp}u_p(t)\Delta t.
}
\]

该规则的目的不是模拟企业最优行为，而是固定一个透明、确定、可重复的 baseline allocation rule，使传播差异主要来自网络结构和物理参数。

后续可以把 proportional rationing 替换为 priority、FIFO、合同优先级或优化策略，并比较传播规律是否发生改变。

---

# 9. WIP、运输等待队列与在途状态

## 9.1 Production WIP

当输入在时刻 \(t\) 被生产过程消耗后，它立即退出可用库存。

由时刻 \(t\) 启动的生产量：

\[
u_p(t)\Delta t
\]

经过：

\[
\tau_p^{prod}
\]

后完成生产。

因此：

\[
\boxed{
\text{production start at }t
\Rightarrow
\text{production completion at }t+\tau_p^{prod}.
}
\]

完成数量按输出系数 \(b_{pm}\) 进入对应输出边的 finished-goods waiting queue。

---

## 9.2 Finished-goods waiting queue

对每条需要运输的输出边 \(e=(p,m)\)，定义：

\[
\boxed{
h_e(t)
}
\]

表示已经生产完成、但由于运输能力限制尚未进入运输的货物。

这是必要状态，因为可能出现：

\[
\text{production completion rate}
>
\text{transport capacity}.
\]

每个时间步能够进入运输的数量为：

\[
\boxed{
x_e^{ship}(t)
=
\min
\left[
h_e(t)+x_e^{new}(t),\,
c_e^{trans}(t)\Delta t
\right],
}
\]

其中 \(x_e^{new}(t)\) 是该时间步刚刚完成生产并进入该边的数量。

等待队列更新为：

\[
\boxed{
h_e(t+\Delta t)
=
h_e(t)+x_e^{new}(t)-x_e^{ship}(t).
}
\]

---

## 9.3 In-transit pipeline

进入运输的物料在：

\[
\tau_e^{trans}
\]

结束前属于 in-transit 状态。

因此：

\[
\boxed{
x_e^{ship}(t)
\Rightarrow
x_e^{arrive}(t+\tau_e^{trans}).
}
\]

在运输完成前，该数量不计入下游可用库存。

---

# 10. 库存更新

对物料节点 \(m\)，在一个时间步内：

\[
\boxed{
q_m(t+\Delta t)
=
q_m(t)
+
A_m(t)
-
C_m(t)
-
S_m(t),
}
\]

其中：

\[
A_m(t)=\text{该时间步从运输管道到达的数量},
\]

\[
C_m(t)
=
\sum_{p:m\rightarrow p}
a_{mp}u_p(t)\Delta t
\]

为生产消耗，

\[
S_m(t)=\text{终端需求实际提取量}.
\]

库存约束：

\[
\boxed{
0\leq q_m(t)\leq \bar q_m.
}
\]

Q2 第一阶段建议令 \(\bar q_m\) 足够大，使 storage capacity 不成为主要瓶颈；若后续研究库存容量冲击，再显式处理满仓、拒收或上游阻塞。

---

# 11. 终端需求

为了使网络存在稳定的物料消耗端，并定义最终产出损失，对 terminal material \(m\) 设置外生需求：

\[
\boxed{
d_m(t).
}
\]

第一阶段可以使用常数需求：

\[
d_m(t)=d_m^0.
\]

实际满足的需求为：

\[
\boxed{
s_m(t)
=
\min
\left(
d_m(t),\,
\frac{q_m(t)}{\Delta t}
\right).
}
\]

对应一个时间步的实际提取量：

\[
S_m(t)=s_m(t)\Delta t.
\]

未满足需求：

\[
\boxed{
z_m(t)=d_m(t)-s_m(t).
}
\]

因此可以定义：

\[
\text{service loss},
\qquad
\text{final-output loss},
\qquad
\text{recovery time}.
\]

终端需求只负责提供固定 sink；第一阶段不让需求根据价格或短缺内生变化。

---

# 12. 单步动力学顺序

为了保证仿真结果可重复，每个时间步按固定顺序执行。

建议顺序：

1. 将本时刻到期的运输批次加入下游可用库存；
2. 将本时刻到期的生产批次加入对应 finished-goods waiting queue；
3. 执行 terminal demand withdrawal；
4. 计算所有过程的候选生产流率 \(\tilde u_p(t)\)；
5. 计算各物料的总输入需求 \(R_m(t)\)；
6. 计算 proportional rationing 系数 \(\rho_m(t)\)；
7. 得到所有过程的实际流率 \(u_p(t)\)；
8. 从可用库存扣除生产投入，并把对应批次加入 WIP pipeline；
9. 根据运输能力从 waiting queue 发货，并加入 in-transit pipeline；
10. 推进时间到 \(t+\Delta t\)。

写成状态方程：

\[
\boxed{
S(t+\Delta t)
=
F_G
\left(
S(t),
c^P(t),
c^T(t),
d(t)
\right).
}
\]

在网络 \(G\)、初始状态 \(S_0\)、能力路径与需求路径给定以后，系统轨迹唯一确定。

---

# 13. 基线稳态

为了研究 shock propagation，建议先构造无冲击基线。

正常状态下：

\[
c_p(t)=\bar c_p,
\qquad
c_e^{trans}(t)=\bar c_e^{trans},
\qquad
d_m(t)=d_m^0.
\]

令系统运行 warm-up，直到主要状态进入稳定或周期稳定区间。

记基准轨迹为：

\[
\boxed{
S^{base}(t).
}
\]

冲击轨迹记为：

\[
\boxed{
S^{shock}(t).
}
\]

所有传播指标都相对于基准轨迹定义，而不是直接看冲击后的绝对水平。

---

# 14. 冲击表示

## 14.1 生产能力冲击

例如过程 \(i\) 在区间 \([t_0,t_0+L)\) 停工：

\[
\boxed{
c_i(t)=0,
\qquad
t_0\leq t<t_0+L.
}
\]

部分减产：

\[
c_i(t)=\gamma\bar c_i,
\qquad
0<\gamma<1.
\]

---

## 14.2 运输能力冲击

例如运输边 \(e\) 发生中断：

\[
\boxed{
c_e^{trans}(t)=0
}
\]

或：

\[
c_e^{trans}(t)
=
\gamma_e\bar c_e^{trans}.
\]

---

## 14.3 需求冲击

若需要研究需求侧冲击，可以外生修改：

\[
\boxed{
d_m(t).
}
\]

第一阶段 Q2 建议先固定 demand，仅研究生产或运输 shock。

---

# 15. 传播指标

Q2 主要关注三个对象：

\[
\boxed{
\text{Arrival Time}
+
\text{Amplitude}
+
\text{System Loss}
}
\]

## 15.1 节点生产缺口

定义过程 \(p\) 的相对生产缺口：

\[
\boxed{
\delta_p(t)
=
1-
\frac{u_p^{shock}(t)}
     {u_p^{base}(t)}
}
\]

当基准流率为零时，该时刻不计算相对缺口。

---

## 15.2 冲击首次到达时间

给定阈值 \(\varepsilon>0\)，定义 shock 从源 \(i\) 到过程 \(j\) 的首次到达时间：

\[
\boxed{
T_{ij}^{arr}
=
\inf
\left\{
t-t_0:
\delta_j(t)\geq\varepsilon
\right\}.
}
\]

---

## 15.3 冲击幅度

可以定义峰值幅度：

\[
\boxed{
A_{ij}^{peak}
=
\max_{t\geq t_0}
\delta_j(t)
}
\]

或累计损失：

\[
\boxed{
A_{ij}^{cum}
=
\int_{t_0}^{T}
\left(
u_j^{base}(t)-u_j^{shock}(t)
\right)dt.
}
\]

离散时间下使用求和。

---

## 15.4 终端损失

终端需求未满足的累计量：

\[
\boxed{
L_m
=
\int_{t_0}^{T}
z_m(t)\,dt.
}
\]

全系统损失：

\[
\boxed{
L^{sys}
=
\sum_{m\in M_{terminal}}
L_m.
}
\]

---

# 16. 面向传播规律的标准化变量

为了比较不同规模、不同参数网络，优先使用无量纲或具有直接物理意义的变量。

## 16.1 库存覆盖时间

对物料 \(m\)，基准消耗率记为：

\[
\lambda_m^0.
\]

定义：

\[
\boxed{
B_m
=
\frac{q_m^0}{\lambda_m^0}
}
\]

其含义是：

\[
\text{当前库存能够支撑正常生产多久}.
\]

相比绝对库存量 \(q_m\)，\(B_m\) 更适合比较不同规模网络。

---

## 16.2 产能冗余

定义：

\[
\boxed{
\sigma_p
=
\frac{\bar c_p-u_p^0}{u_p^0}.
}
\]

\(\sigma_p\) 表示正常生产之外还剩多少相对 capacity slack。

---

## 16.3 路径物理时间

对路径 \(\pi\)，可以定义初步的有效传播长度：

\[
\boxed{
L_{\pi}^{eff}
=
\sum_{r\in\pi}
\left(
\tau_r+B_r
\right)
}
\]

其中 \(\tau_r\) 表示该段生产/运输 lead time，\(B_r\) 表示相关下游库存的 depletion buffer。

该量是 Q2 要通过 motif 分析和随机网络实验检验的候选传播变量，而不是预先假定成立的最终定律。

---

# 17. Q2 的最小研究单元

第一阶段优先研究四类 motif：

## 17.1 Chain

\[
P_1\rightarrow M_1\rightarrow P_2
\]

用于研究：

\[
\text{delay}
+
\text{inventory buffer}
\rightarrow
\text{arrival time}.
\]

---

## 17.2 Fork

\[
M
\rightarrow
\begin{cases}
P_1\\
P_2
\end{cases}
\]

用于研究共享库存下的 rationing 与并行传播。

---

## 17.3 Assembly

\[
M_A,M_B,M_C
\rightarrow
P
\]

用于研究多个必要投入中的 bottleneck 传播。

---

## 17.4 Multi-source Merge

\[
M_A^{(1)},M_A^{(2)}
\rightarrow
P
\]

用于研究供应份额、多源冗余与 shock amplitude 的关系。

---

# 18. 当前核心模型明确不包含的内容

以下内容暂不进入 Q2 baseline：

- 价格；
- 成本；
- 信用；
- 谈判能力；
- 主动换供应商；
- 动态采购优化；
- 安全库存优化；
- 补货策略优化；
- 企业博弈；
- 战略性囤货；
- endogenous demand response；
- 恢复资源优化；
- 网络重构。

这些可以在得到基础传播规律以后逐层加入。

---

# 19. 架构总结

Q2 baseline 的最小完整模型为：

\[
\boxed{
G=(M,P,E)
}
\]

加静态参数：

\[
\boxed{
\Theta=
\left(
a,\,
b,\,
\bar q,\,
\bar c^P,\,
\bar c^T,\,
\tau^P,\,
\tau^T,\,
u^0
\right)
}
\]

加动态状态：

\[
\boxed{
S(t)=
\left(
Q(t),W(t),H(t),L(t)
\right)
}
\]

加固定运行规则：

\[
\boxed{
\text{capacity constraint}
+
\text{input constraint}
+
\text{proportional rationing}
+
\text{fixed terminal demand}
}
\]

于是：

\[
\boxed{
(G,\Theta,S_0,\text{shock})
\Longrightarrow
\text{unique trajectory}
}
\]

并可从轨迹中测量：

\[
\boxed{
T^{arr},
\quad
A^{peak},
\quad
A^{cum},
\quad
L^{sys},
\quad
T^{recovery}
}
\]

从而把问题 2 具体化为：

\[
\boxed{
\text{哪些局部结构与物理参数组合}
\Longrightarrow
\text{哪些稳定的冲击传播规律？}
}
\]

研究顺序建议为：

\[
\boxed{
\text{motif analytic laws}
\rightarrow
\text{random-DAG simulation}
\rightarrow
\text{cross-network scaling laws}
\rightarrow
\text{theoretical verification}
}
\]
