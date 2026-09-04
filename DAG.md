# 供应链网络架构

## 1. 总体结构

供应链网络由以下四个层次构成：

\[
\boxed{
\text{单源生产 DAG}
\;\longrightarrow\;
\text{多源生产 DAG}
\;\longrightarrow\;
\text{镶嵌静态物理属性的 DAG}
\;\longrightarrow\;
\text{随时间演化的动力学系统}
}
\]

当前范围只包括网络结构、静态物理属性和动力学。用于控制网络结构的生成参数、策略、优化与决策暂不纳入。

---

# 2. 单源生产网络

最基本的生产网络为：

\[
\boxed{
M\rightarrow P\rightarrow M
}
\]

其中：

\[
\boxed{M=\text{Material-State-at-Location / 可用物料库存状态}}
\]

\(M\) 表示某种物料在某个位置、某种状态下的可用库存，可积累并等待后续生产调用。

\[
\boxed{P=\text{Production Process}}
\]

\(P\) 表示生产工艺，用于完成物料转换。

因此，基本生产链可以写成：

\[
\text{原材料库存}
\rightarrow
\text{生产}
\rightarrow
\text{中间品库存}
\rightarrow
\text{生产}
\rightarrow
\text{成品库存}
\]

或：

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

一个生产工艺可以同时需要多个输入。例如：

\[
M_A\rightarrow P,\qquad
M_B\rightarrow P,\qquad
M_C\rightarrow P
\]

单源表示某个所需组件当前只有一个真实供应来源。

---

# 3. 多源生产网络

多源结构不增加新的基本节点类型，只改变现实供应链映射为 DAG 时的分叉与合并方式。

核心规则为：

\[
\boxed{
\text{来源差异仍影响未来}
\Rightarrow
\text{保持分叉}
}
\]

\[
\boxed{
\text{来源差异不再影响未来}
\Rightarrow
\text{在最早未来等价点合并}
}
\]

“影响未来”包括来源差异是否继续改变：

- 后续可行路径；
- 工艺要求；
- 兼容关系；
- 物流条件；
- 约束条件；
- 风险；
- 最终结果。

例如两个来源：

\[
M_A^{(1)},\qquad M_A^{(2)}
\]

如果其后续行为仍然不同，则继续保持为两条路径。

若从某一点开始：

\[
Future(M_A^{(1)})\equiv Future(M_A^{(2)})
\]

则可以合并为：

\[
M_A^{(1)},M_A^{(2)}
\rightarrow
M_A^{equiv}
\]

来源历史继续保存在 lineage 中，例如：

\[
supplier,\quad plant,\quad batch,\quad origin
\]

因此拓扑合并不会删除历史追溯信息。

同一个产品可以对应多张生产 DAG。若两套生产实现长期独立、耦合很少，可以分别建网。

\[
\boxed{
\text{Product identity}
\neq
\text{Production-network identity}
}
\]

多源规则属于“现实供应链 \(\rightarrow\) 静态网络”的映射规则，不属于底层动力学语法。

---

# 4. 静态物理属性

静态层只保存使生产网络能够正确运行的基本物理信息。

## 4.1 Material 节点 \(M\)

Material 节点定义为：

\[
\boxed{
m=(material,\ state,\ location)
}
\]

其中：

- \(material\)：物料种类；
- \(state\)：物料状态；
- \(location\)：物料所在位置。

Material 节点还保存库存容量：

\[
\boxed{\bar q_m}
\]

其中：

\[
\bar q_m=\text{该库存位置的最大允许存量}
\]

实际库存量：

\[
q_m(t)
\]

属于动态状态。

---

## 4.2 Production 节点 \(P\)

\(P\) 只表示生产工艺。

生产工艺至少保存：

\[
\boxed{
P:\left(\bar c_p,\tau_p^{prod}\right)
}
\]

其中：

\[
\bar c_p=\text{单位时间最大生产能力}
\]

\[
\tau_p^{prod}=\text{生产 delay}
\]

生产运行时的实际流率记为：

\[
u_p(t)
\]

并满足：

\[
0\le u_p(t)\le \bar c_p
\]

生产能力与生产 delay 表示两种不同物理限制：

\[
\bar c_p
\]

决定单位时间最多能够处理多少物料；

\[
\tau_p^{prod}
\]

决定物料进入生产以后，需要经过多久才能形成产出。

---

## 4.3 输入边 \(M\rightarrow P\)

输入边保存投入系数：

\[
\boxed{a_{mp}}
\]

表示每运行一个单位的生产工艺，需要多少单位输入物料。

例如：

\[
2A+3B\rightarrow C
\]

对应：

\[
a_{A,p}=2,\qquad
a_{B,p}=3
\]

---

## 4.4 输出边 \(P\rightarrow M\)

输出边保存产出系数：

\[
\boxed{b_{pm}}
\]

表示每运行一个单位的生产工艺，会产生多少单位输出物料。

对于：

\[
2A+3B\rightarrow C
\]

有：

\[
b_{p,C}=1
\]

若生产地点和下游库存地点不同，运输信息也存放在该连接上。

包括：

\[
\boxed{
\bar c_e^{trans},\qquad
\tau_e^{trans}
}
\]

其中：

\[
\bar c_e^{trans}
=
\text{运输能力}
\]

\[
\tau_e^{trans}
=
\text{运输 delay}
\]

---

# 5. Delay

Delay 是网络的核心静态物理参数之一。

生产 delay：

\[
\boxed{\tau_p^{prod}}
\]

运输 delay：

\[
\boxed{\tau_e^{trans}}
\]

两种 delay 均与 capacity 分开。

Capacity 描述：

\[
\text{单位时间最多能够处理多少}
\]

Delay 描述：

\[
\text{一批物料进入某个阶段以后，多久才能进入下一状态}
\]

例如：

\[
\tau_p^{prod}=3h,
\qquad
\tau_e^{trans}=8h
\]

若物料在 10:00 进入生产，则：

\[
10{:}00
\rightarrow
13{:}00
\]

完成生产。

随后经过运输：

\[
13{:}00
\rightarrow
21{:}00
\]

最终在 21:00 进入下游可用库存。

因此一段物料传播过程为：

\[
\boxed{
\text{available stock}
\xrightarrow{\text{production}}
\tau_p^{prod}
\xrightarrow{\text{transport}}
\tau_e^{trans}
\rightarrow
\text{next available stock}
}
\]

运输完成以前，相关物料属于连接上的在途状态，不计入下游库存。

---

# 6. 运输与中间库存

当前结构中，\(P\) 只表示生产，运输放在连接上。

例如：

\[
P_A
\rightarrow
M_{AB}
\rightarrow
P_B
\]

其中：

\[
M_{AB}
\]

表示已经完成必要运输、当前可以被 \(P_B\) 调用的中间库存。

若 \(P_A\) 的产出需要经过运输才能到达 \(M_{AB}\)，则：

\[
P_A
\xrightarrow[\bar c_{AB}^{trans}]{\tau_{AB}^{trans}}
M_{AB}
\rightarrow
P_B
\]

在运输完成以前，物料属于：

\[
\boxed{\text{in-transit state on edge}}
\]

运输完成后才进入：

\[
q_{AB}(t)
\]

因此运行中的物料可以自然分布在三个位置：

\[
\boxed{
\begin{array}{ccc}
P & E & M\\
\text{生产中} & \text{运输中} & \text{库存中}
\end{array}
}
\]

---

# 7. 当前最小静态语法

当前静态网络可以写成：

\[
\boxed{
G^S=(M,P,E^{MP},E^{PM};\Theta)
}
\]

其最小物理参数为：

\[
\boxed{
\begin{array}{lll}
M
&:&
(material,\ state,\ location,\ \bar q_m)
\\[2mm]
P
&:&
(\bar c_p,\ \tau_p^{prod})
\\[2mm]
M\rightarrow P
&:&
a_{mp}
\\[2mm]
P\rightarrow M
&:&
b_{pm},\ \bar c_e^{trans},\ \tau_e^{trans}
\end{array}
}
\]

静态层定义生产系统能够怎样运行。

---

# 8. 动力学系统

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

---

## 8.1 库存

库存状态为：

\[
\boxed{
q_m(t)
}
\]

并满足：

\[
0\le q_m(t)\le \bar q_m
\]

其中：

- \(q_m(t)\)：当前可用库存；
- \(\bar q_m\)：静态库存容量。

---

## 8.2 生产流率

生产工艺的实际运行强度为：

\[
\boxed{
u_p(t)
}
\]

满足：

\[
0\le u_p(t)\le \bar c_p
\]

\(\bar c_p\) 是静态生产能力，\(u_p(t)\) 是动态生产流率。

---

## 8.3 生产中物料

物料进入生产过程以后，从上游可用库存中退出。

在：

\[
\tau_p^{prod}
\]

结束以前，该部分物料处于生产中状态。

因此：

\[
\boxed{
\text{input at }t
\Rightarrow
\text{production output at }t+\tau_p^{prod}
}
\]

---

## 8.4 在途物料

生产完成后，如果下游库存位于其他地点，物料进入运输过程。

经过：

\[
\tau_e^{trans}
\]

后，物料才进入下游库存。

因此：

\[
\boxed{
\text{production completed}
\Rightarrow
\text{in transit}
\Rightarrow
\text{available downstream inventory}
}
\]

---

# 9. 动态历史

静态网络和动力学规则运行后，会自然产生 trajectory / event history。

包括：

- 各节点库存随时间变化；
- 各生产过程实际流率；
- 各批物料的生产开始与完成时间；
- 在途物料数量；
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

---

# 10. 冲击传播

冲击可以通过生产能力或运输能力变化进入系统。

例如：

\[
\bar c_p:
100
\rightarrow
20
\]

或：

\[
\bar c_e^{trans}:
1000
\rightarrow
300
\]

网络拓扑保持不变，运行时的有效能力发生变化。

冲击传播主要受到三类物理因素影响：

\[
\boxed{
\text{Capacity}
+
\text{Inventory}
+
\text{Delay}
}
\]

其中：

- Capacity 决定局部生产或运输缺口的大小；
- Inventory 决定下游能够维持运行的时间；
- Delay 决定影响到达下游的时间。

典型传播过程为：

\[
\boxed{
\text{local shock}
\rightarrow
\text{inventory depletion}
\rightarrow
\text{production reduction}
\rightarrow
\text{delayed downstream propagation}
}
\]

库存和 delay 会使冲击以有限速度沿供应链传播。

---

# 11. 当前不纳入核心模型的内容

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
- 恢复与优化策略。

这些内容可以在后续分别进入经济层、策略层、控制层或网络生成层。

---

# 12. 架构概括

该模型是一张以可用物料库存 \(M\) 和生产工艺 \(P\) 为核心的生产 DAG。

多源结构通过未来相关性决定分叉，通过未来等价决定合并。

静态物理层包含：

\[
\boxed{
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
}
\]

这些动态状态共同构成供应链的时间演化，并形成冲击传播过程。
