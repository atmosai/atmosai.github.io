# 资料同化基本术语


**数据同化(Data Assimilation)**：在地球物理学领域，数据同化就是利用**物理特性**以及**时间演变定律**的一致性约束将**观测信息**加入到**模式状态**中的一种**分析技术**。简单的说：数据同化就是利用一系列约束条件将观测信息加到模式中，更改模式的初始状态和观测更为接近，来达到更好的预报效果。

数据同化通常分为两种形式：

* `序列化同化(sequential assimilation)`：这种形式的同化通常应用于实时同化系统，因为所采用的数据是过去一段时间到分析时刻的所有观测
* `非序列化/回顾性同化(non-sequential/retrospective assimilation)`：这种形式的同化通常是应用于再分析数据的生成系统，因为这种方式所采用的观测数据包含了来自"未来"的数据。比如NCEP的FNL再分析数据，想生成2019.04.17 18UTC时刻的再分析数据，就要用到2019.04.17 18UTC之后以及之前的观测数据



![](https://github.com/bugsuse/blogpic/blob/master/2019/04/18/DA_fig1.jpg?raw=true)

<center><font size=2 color=gray>同化预报系统框架</font></center>



**状态向量x(State Vector)**：预报模式中，用于表示大气状态的矩阵。

**真实状态(true state)(x_t)**：模式所能呈现的可能最好的状态，由于模式分辨率低于真实情况，分析是无法完全和真实状态一样的。**注意**：真实状态是未知量，而且是不可知量。

**观测向量y(Observation Vector)**：由一系列观测数据所构成的矩阵。

**控制向量(Control Variable)**：分析过程中所涉及到的变量。

**观测空间(Observation Space)**：控制变量空间(Control Variable Space)之一。主要是对观测数据进行处理。

**分析空间(Analysis Space)**：控制变量空间之一。主要是根据背景状态向量和观测进行分析。

**观测算子H(forward/observation operator)**：由于相对于模式数据而言，观测数据很少且分布不规则，为了对比观测和状态向量而引入，是将模式状态空间函数应用于观测空间，从而称为**观测算子**

**变化量(innovation)**：观测和将背景场作用于观测算子后的差，即`y - H(xb)`，这里的x是背景场向量

**观测增量(Observation Increment)**：观测向量和背景向量的差，即`y - x_b`，而观测向量和背景向量之间进行比较需要利用观测算子进行变换，即`y - H(x_b)`，因此**观测增量**和**innovation**是相同的

**分析增量(Analysis Increment)**：分析向量和背景向量的差，即`X_a - X_b`，**分析增量为观测增量和最优权重的乘积**

**分析残差(Analysis Residuals)**：观测向量和将分析场作用于观测算子后的差，即`y - H(xa)`

**背景误差(Background Error)**：由于物理过程的简化等因素所造成的误差，即`x_b - x_t`

**表示性误差(representativeness errors)**：由于**模式离散化**(model discretization)所造成的误差，因为模式的分辨率和实际情况也存在差异，也会存在误差。

**观测误差(Observation Error)**：由于观测设备问题所导致的误差(**设备误差**(instrumental errors))，在分析过程的数学表达上，通常将**表示性误差**也划到观测误差中。即`y - H(x_t)`

**分析误差(Analysis Error)**：分析向量和真实状态向量之间的差，即`x_a - x_t`

**协方差(Covariance)**：衡量变量之间的总体误差

**观测误差协方差矩阵(Observation Error Covariance Matrix)**：观测变量之间的误差矩阵

**背景误差协方差矩阵(Background Error Covariance Matrix)**：模式变量之间的误差矩阵。对该矩阵的合理估计尤为重要。因为矩阵的大小和形状控制了分析增量的影响函数。**形状**决定了从观测到分析格点的信息的传播，**大小**决定了观测权重和背景权重的大小，当背景误差越大时，观测的权重越大。

**分析误差协方差矩阵(Analysis Error Covariance Matrix)**：分析变量之间的误差矩阵



![](https://github.com/bugsuse/blogpic/blob/master/2019/04/18/DA_fig2.jpg?raw=true)

<center><font size=2 color=gray>控制变量空间及及计算过程</font></center>



​    

### 更新记录

2019.04.18 初次更新相关术语


