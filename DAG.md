# 供应链网络架构

## 1. 总体结构

供应链网络表示为一张带静态物理属性、并可沿统一时间轴运行的有向无环图（DAG）。整个架构由四个层次组成：

\[
\boxed{
\text{单源生产 DAG}
\longrightarrow
\text{多源生产 DAG}
\longrightarrow
\text{镶嵌静态物理属性的 DAG}
\longrightarrow
\text{随时间演化的动力学系统}
}
\]

网络中只保留两类节点：

\[
\boxed{
M=\text{Material-State-at-Location / 可用物料库存状态}
}
\]

\[
\boxed{
P=\text{Production Process / 生产流程}
}
\]

\(M\) 表示某种物料在某个位置、某种状态下的独立可用库存，可积累并等待后续调用；\(P\) 表示生产工艺，用于完成物料转换。

所有有向边都具有运输行为。只要物料沿一条边从上游对象移动到下游对象，该边就保存运输能力和运输 delay。与生产流程 \(P\) 相连的边还同时承担投入或产出的计量关系：

\[
\boxed{
M\rightarrow P:
\text{运输}+\text{投入系数}
}
\]

\[
\boxed{
P\rightarrow M:
\text{产出系数}+\text{运输}
}
\]

\[
\boxed{
M\rightarrow M:
\text{纯运输}
}
\]

当一个 \(P\) 的同一种产出需要供给多个下游生产流程时，对应的 \(P\rightarrow M\) 分支边还保存配给比例，用于描述该产出在各下游分支之间的分配。

当前核心范围包括网络结构、静态物理属性和动力学。用于控制网络结构的生成参数、经济策略、采购策略、优化和恢复决策暂不纳入核心 DAG。

---

## 2. 单源生产网络

最基本的生产单元为：

\[
\boxed{
M^{in}\rightarrow P\rightarrow M^{out}
}
\]

其中 \(M^{in}\) 是进入生产流程之前的可用输入库存，\(M^{out}\) 是生产输出经过对应运输边后进入的可用输出库存。

当两个生产流程连续连接时，标准结构固定为：

\[
\boxed{
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
}
\]

完整地写入前后生产单元后：

\[
\boxed{
M_i^{in}
\rightarrow
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
\rightarrow
M_j^{out}
}
\]

两个相邻生产流程之间保留两个独立的 \(M\)。

第一个库存：

\[
M_{ij}^{(1)}
\]

表示上游 \(P_i\) 完成生产以后，通过 \(P_i\rightarrow M_{ij}^{(1)}\) 运输边到达的生产后库存。

第二个库存：

\[
M_{ij}^{(2)}
\]

表示经过中间物流以后、位于下游生产端之前的可用库存。

因此局部物理过程为：

\[
\boxed{
P_i
\xrightarrow{\text{output transport}}
M_{ij}^{(1)}
\xrightarrow{\text{pure transport}}
M_{ij}^{(2)}
\xrightarrow{\text{input transport}}
P_j
}
\]

三条边全部具有运输能力和运输 delay。

现实物流中即使包含卡车、铁路、港口、海运、清关、配送中心或其他中间环节，也不继续增加新的 \(M\)。这些内部物流细节统一折叠进当前既有边的运输参数中。尤其是两个库存之间的：

\[
M_{ij}^{(1)}\rightarrow M_{ij}^{(2)}
\]

承担两端库存之间的完整中间物流过程。

图上的边长可以用于辅助表达空间事实。生产节点与库存节点画得很近，可以表示二者位于同一厂区或相邻位置；两个 \(M\) 画得很近，可以表示两个库存设施在现实中距离较短；跨区域物流可以画得更长。图形边长提供空间直觉，真正进入动力学计算的运输时间由每条边自己的 \(\tau_e^{trans}\) 给出。

一个生产流程可以同时需要多个输入。例如：

\[
M_A\rightarrow P,\qquad
M_B\rightarrow P,\qquad
M_C\rightarrow P
\]

也可以产生多个输出：

\[
P\rightarrow M_D,\qquad
P\rightarrow M_E
\]

输入和输出的数量关系分别由边上的投入系数与产出系数确定。

单源表示某个所需组件在当前网络中只有一个真实供应来源，不限制一个生产流程只能具有一种输入或一种输出。

---

## 3. 多源、分支与配给

多源网络沿用同一套 \(P\)、\(M\) 和边语义，不增加新的节点类型。

### 多个上游生产流程供给一个下游生产流程

若 \(P_1\) 和 \(P_2\) 都向同一个下游 \(P_D\) 提供生产所需物料，可以写成两条独立来源路径：

\[
P_1
\rightarrow
M_{1D}^{(1)}
\rightarrow
M_{1D}^{(2)}
\rightarrow
P_D
\]

\[
P_2
\rightarrow
M_{2D}^{(1)}
\rightarrow
M_{2D}^{(2)}
\rightarrow
P_D
\]

每条路径拥有独立库存编号、库存容量和运输参数。多个上游供给一个 \(P_D\) 时，不需要在上游 \(P\rightarrow M\) 边上增加“向 \(P_D\) 配给多少”的额外比例；下游生产对各类输入的需求由 \(M\rightarrow P\) 边上的投入系数直接定义。

例如：

\[
\boxed{
2A+3B\rightarrow C
}
\]

对应：

\[
a_{A,p}=2,\qquad
a_{B,p}=3
\]

以及：

\[
b_{p,C}=1
\]

其含义是生产流程 \(P\) 每运行一个单位，需要消耗 \(2\) 单位 \(A\) 和 \(3\) 单位 \(B\)，并产生 \(1\) 单位 \(C\)。

如果 \(A\) 与 \(B\) 分别来自不同的上游生产流程，来源路径仍然保持独立；最终进入 \(P\) 的数量关系继续由：

\[
2:3
\]

这一投入配方决定。

### 一个生产流程供给多个下游生产流程

若同一个生产流程 \(P_i\) 的同一种产出需要供给多个下游 \(P_1,\ldots,P_k\)，则每条对应的 \(P_i\rightarrow M\) 分支边除了产出系数和运输属性外，还保存配给比例：

\[
\boxed{
\rho_{ie}
}
\]

例如：

\[
P_i
\rightarrow
M_{i1}^{(1)}
\rightarrow
M_{i1}^{(2)}
\rightarrow
P_1
\]

\[
P_i
\rightarrow
M_{i2}^{(1)}
\rightarrow
M_{i2}^{(2)}
\rightarrow
P_2
\]

若两条路径分配的是 \(P_i\) 的同一种输出物料，则：

\[
\rho_{i1}\ge 0,\qquad
\rho_{i2}\ge 0
\]

并通常满足：

\[
\boxed{
\rho_{i1}+\rho_{i2}=1
}
\]

一般情况下，对同一输出物料的全部下游分支集合 \(\mathcal B(p,m)\)：

\[
\boxed{
\sum_{e\in\mathcal B(p,m)}\rho_e=1
}
\]

若 \(P_i\) 每运行一个单位可产生 \(b_{pm}\) 单位物料 \(m\)，则分配到分支 \(e\) 的名义产出量为：

\[
\boxed{
\rho_e\,b_{pm}
}
\]

配给比例用于同一种产出在多个下游之间的分流。若 \(P\) 同时产生不同种类的联产品，则不同物料的数量首先由各自的产出系数 \(b_{pm}\) 定义；配给比例只在某一种具体产出继续向多个下游分支时使用。

当前静态网络中 \(\rho_e\) 作为给定配给比例保存。若后续研究调度或主动控制，可以进一步扩展成随时间变化的 \(\rho_e(t)\)。

### 库存实例不自动合并

每个 \(M\) 都是一个独立库存实例，并具有唯一编号：

\[
\boxed{
id_m
}
\]

即使两个库存具有完全相同的：

\[
(material,\ state,\ location)
\]

它们仍然可以保持为：

\[
M^{(1)},\qquad M^{(2)}
\]

并分别拥有：

\[
q_{m_1}(t),\qquad q_{m_2}(t)
\]

以及各自的库存容量、上下游连接和冲击历史。

因此，\(material\)、\(state\) 和 \(location\) 用于描述库存的物理属性，\(id_m\) 用于确定库存实例身份。只有现实中本来就是同一个物理库存实例时，网络才使用同一个 \(M\) 节点。

同一个产品可以对应多张生产 DAG。若两套生产实现长期独立、耦合很少，可以分别建网：

\[
\boxed{
\text{Product identity}
\neq
\text{Production-network identity}
}
\]

---

## 4. 静态物理属性

静态层保存生产网络能够正确运行所需要的基本物理信息。

当前静态网络写为：

\[
\boxed{
G^S=
\left(
M,\,
P,\,
E^{MP},\,
E^{PM},\,
E^{MM};
\Theta
\right)
}
\]

其中：

\[
E^{MP}\subseteq M\times P
\]

表示库存到生产流程的投入运输边；

\[
E^{PM}\subseteq P\times M
\]

表示生产流程到库存的产出运输边；

\[
E^{MM}\subseteq M\times M
\]

表示库存之间的纯运输边。

所有边组成：

\[
E=E^{MP}\cup E^{PM}\cup E^{MM}
\]

并统一具有运输能力和运输 delay。

### Material 节点 \(M\)

Material 节点至少保存：

\[
\boxed{
M_m:
\left(
id_m,\,
material_m,\,
state_m,\,
location_m,\,
\bar q_m
\right)
}
\]

其中：

\[
id_m=\text{库存实例唯一编号}
\]

\[
material_m=\text{物料种类}
\]

\[
state_m=\text{物料状态}
\]

\[
location_m=\text{物料所在位置}
\]

\[
\bar q_m=\text{该库存实例的最大允许存量}
\]

实际库存量：

\[
\boxed{
q_m(t)
}
\]

属于动态状态，并满足：

\[
\boxed{
0\le q_m(t)\le \bar q_m
}
\]

初始库存为：

\[
\boxed{
q_m(0)
}
\]

必要时，\(M\) 还可以保存库存所属企业、仓储类型、设施编号等附加静态信息；这些字段不会改变库存节点的基本物理意义。

### Production 节点 \(P\)

\(P\) 表示生产工艺。生产流程至少保存：

\[
\boxed{
P_p:
\left(
id_p,\,
process_p,\,
location_p,\,
\bar c_p,\,
\tau_p^{prod}
\right)
}
\]

其中：

\[
id_p=\text{生产流程实例编号}
\]

\[
process_p=\text{生产工艺或生产活动}
\]

\[
location_p=\text{生产位置}
\]

\[
\bar c_p=\text{单位时间最大生产能力 / 额定产出率}
\]

\[
\tau_p^{prod}=\text{生产 delay}
\]

生产运行时的实际流率记为：

\[
\boxed{
u_p(t)
}
\]

并满足：

\[
\boxed{
0\le u_p(t)\le \bar c_p
}
\]

生产能力决定单位时间最多能够完成多少生产活动；生产 delay 决定一批投入进入生产以后，需要经过多久才能形成产出。

同一家公司内部多个 \(P\) 可能共享企业级总产量限制、设备限制或其他联合约束。当前核心 DAG 不额外表述这类公司级聚合产量约束，各个 \(P\) 只保存自身局部生产参数。

### 输入边 \(M\rightarrow P\)

任意输入边：

\[
e=(m,p)\in E^{MP}
\]

都保存：

\[
\boxed{
e^{MP}:
\left(
a_{mp},\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
\]

其中：

\[
a_{mp}=\text{投入系数}
\]

表示每运行一个单位生产流程 \(P_p\)，需要多少单位来自库存 \(M_m\) 的输入物料。

同时：

\[
\bar c_e^{trans}=\text{该边单位时间最大运输能力}
\]

\[
\tau_e^{trans}=\text{该边运输 delay}
\]

因此 \(M\rightarrow P\) 同时表达“送入生产”和“送多少才能完成一个单位生产”。

对于：

\[
2A+3B\rightarrow C
\]

有：

\[
\boxed{
a_{A,p}=2,\qquad a_{B,p}=3
}
\]

### 输出边 \(P\rightarrow M\)

任意输出边：

\[
e=(p,m)\in E^{PM}
\]

至少保存：

\[
\boxed{
e^{PM}:
\left(
b_{pm},\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
\]

其中：

\[
b_{pm}=\text{产出系数}
\]

表示每运行一个单位生产流程 \(P_p\)，会产生多少单位送往库存 \(M_m\) 的输出物料。

对于：

\[
2A+3B\rightarrow C
\]

有：

\[
\boxed{
b_{p,C}=1
}
\]

若同一种生产输出从 \(P_p\) 分配给多个下游分支，则相应的 \(P\rightarrow M\) 分支边增加配给比例：

\[
\boxed{
e^{PM}_{branch}:
\left(
b_{pm},\,
\rho_e,\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
\]

其中 \(\rho_e\) 描述该输出分配到当前分支的比例。

因此，一个 \(P\rightarrow M\) 分支同时携带三类物理信息：

\[
\boxed{
\text{Production output relation}
+
\text{Allocation}
+
\text{Transportation}
}
\]

没有发生一对多分配时，\(\rho_e\) 可以省略。

### 库存边 \(M\rightarrow M\)

任意库存间运输边：

\[
e=(m_i,m_j)\in E^{MM}
\]

保存：

\[
\boxed{
e^{MM}:
\left(
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
\]

它表示两个独立库存实例之间的纯物流移动，不承担生产投入或产出系数。

若模型需要显式记录物理距离，也可以在所有边上增加：

\[
d_e
\]

得到：

\[
\boxed{
e:
\left(
\bar c_e^{trans},\,
\tau_e^{trans},\,
d_e
\right)
}
\]

\(d_e\) 用于表达实际空间尺度和辅助图形布局；动力学中的传播时间仍由 \(\tau_e^{trans}\) 直接给出。

当前最小静态物理信息可以概括为：

\[
\boxed{
\begin{array}{lll}
M
&:&
(id_m,\ material,\ state,\ location,\ \bar q_m)
\\[2mm]
P
&:&
(id_p,\ process,\ location,\ \bar c_p,\ \tau_p^{prod})
\\[2mm]
M\rightarrow P
&:&
(a_{mp},\ \bar c_e^{trans},\ \tau_e^{trans})
\\[2mm]
P\rightarrow M
&:&
(b_{pm},\ [\rho_e],\ \bar c_e^{trans},\ \tau_e^{trans})
\\[2mm]
M\rightarrow M
&:&
(\bar c_e^{trans},\ \tau_e^{trans})
\end{array}
}
\]

其中 \([\rho_e]\) 只在同一种输出从一个 \(P\) 向多个下游分支配给时使用。

---

## 5. Capacity、Delay 与运输状态

Capacity 和 delay 描述不同的物理限制。

生产能力：

\[
\boxed{
\bar c_p
}
\]

决定单位时间最多能够运行多少生产活动。

生产 delay：

\[
\boxed{
\tau_p^{prod}
}
\]

决定投入进入 \(P\) 后，需要经过多久才能形成生产输出。

每一条边都有运输能力：

\[
\boxed{
\bar c_e^{trans}
}
\]

决定单位时间最多能够沿该边移动多少物料。

每一条边也都有运输 delay：

\[
\boxed{
\tau_e^{trans}
}
\]

决定物料进入该边以后，需要多久才能到达边的另一端。

因此：

\[
\boxed{
\text{Capacity}
=
\text{单位时间最多能够处理或运输多少}
}
\]

\[
\boxed{
\text{Delay}
=
\text{已经进入某个过程的物料多久以后才能到达下一状态}
}
\]

对一段：

\[
M_A\rightarrow P\rightarrow M_B
\]

如果：

\[
\tau_{A,P}^{trans}=1h,\qquad
\tau_p^{prod}=3h,\qquad
\tau_{P,B}^{trans}=2h
\]

则一批物料从 \(M_A\) 发出，到最终进入 \(M_B\) 的最短物理时间为：

\[
\boxed{
1h+3h+2h=6h
}
\]

运输完成以前，物料属于相应边上的在途状态，不计入目标库存。

因此运行中的物料可以分布在：

\[
\boxed{
\text{库存中}
+
\text{运输中}
+
\text{生产中}
}
\]

三个基本物理状态中。

---

## 6. 动力学系统

静态网络确定以后，引入统一时间轴：

\[
\boxed{
t:0\rightarrow T
}
\]

并给定初始状态：

\[
\boxed{
S_0
}
\]

最基本的初始状态包括：

\[
q_m(0)
\]

必要时还可以包含初始 WIP 和初始在途物料。

随着时间推进，系统产生动态状态：

\[
\boxed{
S(t)
}
\]

### 库存

每个库存实例具有独立库存轨迹：

\[
\boxed{
q_m(t)
}
\]

满足：

\[
\boxed{
0\le q_m(t)\le\bar q_m
}
\]

库存只在运输真正到达 \(M\) 后增加；物料从 \(M\) 发出进入任意出边时，相应数量退出当前可用库存。

若 \(In(m)\) 和 \(Out(m)\) 分别表示库存节点 \(m\) 的入边和出边，在固定运输 delay、无损运输的连续流近似下：

\[
\boxed{
\dot q_m(t)
=
\sum_{e\in In(m)}
f_e\!\left(t-\tau_e^{trans}\right)
-
\sum_{e\in Out(m)}
f_e(t)
}
\]

其中 \(f_e(t)\) 是时刻 \(t\) 进入边 \(e\) 的实际运输流率。

### 运输流与在途物料

任意边的实际运输流率为：

\[
\boxed{
f_e(t)
}
\]

满足：

\[
\boxed{
0\le f_e(t)\le\bar c_e^{trans}
}
\]

物料在时刻 \(t\) 进入边 \(e\)，在：

\[
t+\tau_e^{trans}
\]

到达下游节点。

若运输无损，固定 delay 边上的在途量可写为：

\[
\boxed{
z_e(t)
=
\int_{t-\tau_e^{trans}}^{t}f_e(s)\,ds
}
\]

因此 \(P\rightarrow M\)、\(M\rightarrow M\) 和 \(M\rightarrow P\) 都可以存在各自独立的在途状态。

### 生产流率与生产中物料

生产流程实际运行强度为：

\[
\boxed{
u_p(t)
}
\]

并满足：

\[
\boxed{
0\le u_p(t)\le\bar c_p
}
\]

对输入边 \(M_m\rightarrow P_p\)，每运行一个单位 \(P_p\) 需要：

\[
a_{mp}
\]

单位输入。多输入生产必须同时满足相应投入配方。

例如：

\[
2A+3B\rightarrow C
\]

若某一时段计划运行：

\[
u_p(t)=10
\]

则需要已经完成输入运输并可供该生产流程使用的：

\[
20
\]

单位 \(A\) 和：

\[
30
\]

单位 \(B\)。

物料真正进入生产以后，在：

\[
\tau_p^{prod}
\]

结束以前属于 WIP / 生产中状态：

\[
\boxed{
\text{production input at }t
\Rightarrow
\text{production output at }t+\tau_p^{prod}
}
\]

生产完成后的数量由输出系数给出。

如果：

\[
b_{p,C}=1
\]

则运行 \(10\) 个生产单位产生：

\[
10C
\]

随后这些产出进入相应的 \(P\rightarrow M\) 运输边。

### 一对多配给的动态含义

若同一种产出通过多条 \(P\rightarrow M\) 分支供给多个下游，静态配给比例决定各分支获得的名义份额。

设生产流程 \(P_p\) 对物料 \(C\) 的产出系数为：

\[
b_{p,C}
\]

两个分支的配给比例为：

\[
\rho_1,\qquad\rho_2
\]

且：

\[
\rho_1+\rho_2=1
\]

则生产完成后对应两个分支的名义释放量为：

\[
\boxed{
\rho_1 b_{p,C}u_p(t-\tau_p^{prod})
}
\]

和：

\[
\boxed{
\rho_2 b_{p,C}u_p(t-\tau_p^{prod})
}
\]

每个分支随后继续受到自身 \(P\rightarrow M\) 边运输能力与运输 delay 的约束。

如果某条分支的运输能力、目标库存容量或其他物理条件不足以承载给定配给，实际可行生产流率会受到相应约束。后续若引入主动调度，可以通过调整 \(\rho_e(t)\) 改变分配。

---

## 7. 动态历史与冲击传播

静态网络和动力学规则运行后，会产生 trajectory / event history，包括：

- 各库存实例随时间变化的 \(q_m(t)\)；
- 各生产过程实际流率 \(u_p(t)\)；
- 各条边实际运输流率 \(f_e(t)\)；
- 各批物料的生产开始与完成时间；
- 各条边上的在途物料数量；
- 一对多分支的实际配给结果；
- 冲击后的产能变化；
- 冲击传播路径；
- 恢复过程。

整体关系为：

\[
\boxed{
\text{Static Network}
+
\text{Dynamics}
\longrightarrow
\text{Trajectory / Dynamic History}
}
\]

动态历史属于运行结果，不属于静态拓扑。

冲击可以通过生产能力或任意边的运输能力变化进入系统。例如：

\[
\bar c_p:100\rightarrow20
\]

或：

\[
\bar c_e^{trans}:1000\rightarrow300
\]

也可以通过运输 delay 增大进入系统：

\[
\tau_e^{trans}:8h\rightarrow20h
\]

网络拓扑可以保持不变，运行时的有效物理参数发生变化。

对于标准结构：

\[
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
\]

若中间运输能力下降，则可能出现：

\[
q_{M_{ij}^{(1)}}(t)\uparrow
\]

以及：

\[
q_{M_{ij}^{(2)}}(t)\downarrow
\]

第一个库存逐渐积压并可能达到：

\[
q_{M_{ij}^{(1)}}(t)=\bar q_{M_{ij}^{(1)}}
\]

从而反向约束上游生产；第二个库存逐渐耗尽并可能达到：

\[
q_{M_{ij}^{(2)}}(t)=0
\]

从而限制下游生产。

一个上游 \(P\) 同时供给多个下游时，配给比例还会改变冲击在不同分支上的分布。即使总产出冲击相同，不同的：

\[
\rho_e
\]

也会产生不同的下游库存耗尽时间和生产下降幅度。

冲击传播主要受到：

\[
\boxed{
\text{Capacity}
+
\text{Inventory}
+
\text{Delay}
+
\text{Allocation}
}
\]

共同影响。

其中：

- Capacity 决定局部生产或运输的最大吞吐；
- Inventory 决定网络可以吸收短期缺口或积压多长时间；
- Delay 决定物理影响沿生产和运输过程传播所需要的时间；
- Allocation 决定一个上游产出在多个下游分支之间如何分布。

典型传播过程可以写为：

\[
\boxed{
\text{local shock}
\rightarrow
\text{transport / production flow change}
\rightarrow
\text{inventory accumulation or depletion}
\rightarrow
\text{production reduction}
\rightarrow
\text{delayed downstream propagation}
}
\]

---

## 8. 当前不纳入核心模型的内容

以下内容暂不进入核心静态语法：

- 供应商合作等级；
- 信用；
- 采购偏好；
- 谈判能力；
- 风险评分；
- 安全库存策略；
- 补货点；
- 目标库存；
- 工作班次与排班；
- 成本和价格；
- 用于生成或控制网络结构的参数；
- 生产与采购决策策略；
- 恢复与优化策略；
- 同一企业内部多个 \(P\) 之间的公司级总产量限制；
- 共享设备、共享劳动力等企业级联合资源约束。

这些内容可以在后续分别进入经济层、策略层、控制层或网络生成层，不改变当前核心 DAG 的基本物理语法。

---

## 9. 架构概括

该模型是一张以可用物料库存 \(M\) 和生产流程 \(P\) 为核心、以全部有向边承担物流行为的生产 DAG。

连续生产流程之间采用固定结构：

\[
\boxed{
P-M-M-P
}
\]

其中两个 \(M\) 是两个独立库存实例，中间不继续增加库存节点。

所有边都保存：

\[
\boxed{
\text{Transport Capacity}
+
\text{Transport Delay}
}
\]

输入边：

\[
\boxed{
M\rightarrow P
}
\]

额外保存投入系数：

\[
\boxed{
a_{mp}
}
\]

输出边：

\[
\boxed{
P\rightarrow M
}
\]

额外保存产出系数：

\[
\boxed{
b_{pm}
}
\]

当一个 \(P\) 的同一种产出供给多个下游时，相应 \(P\rightarrow M\) 分支再保存：

\[
\boxed{
\rho_e
}
\]

用于表达配给比例。

库存间：

\[
\boxed{
M\rightarrow M
}
\]

只承担纯运输。

\(P\) 保存生产能力、生产 delay 及生产流程自身的信息；\(M\) 保存唯一编号、物料、状态、位置、最大库存容量等信息。属性相同的库存实例不自动合并，各自拥有独立库存状态和动态历史。

当前静态物理层最终包含：

\[
\boxed{
\text{Inventory Identity}
+
\text{Inventory Capacity}
+
\text{Production Capacity}
+
\text{Production Delay}
+
\text{Transport Capacity}
+
\text{Transport Delay}
+
\text{Input/Output Coefficients}
+
\text{Allocation Ratios}
}
\]

时间推进后，系统产生：

\[
\boxed{
\text{Inventory}
+
\text{WIP}
+
\text{In-transit Flow}
+
\text{Production Flow}
+
\text{Transport Flow}
}
\]

这些动态状态共同构成供应链的时间演化，并用于研究局部生产冲击、物流冲击、库存缓冲、多源投入和一对多配给条件下的因果传播。
