# ESMF:模式间耦合及插值工具


ESMF是用于耦合天气，气候和相关模式的工具，具有稳定，并行化和可扩展的插值(remapping)包，通常用于生成插值权重。ESMF可以处理大多数网格和选项：常规矩形网格，非结构网格(即网格大小不同的网格结构，比如ISCCP的网格)和一系列散点；区域或全球网格；2D或3D；极地(pole)或掩膜(masking)选项。ESMF也可以从NetCDF文件中读取网格信息，比如Climate and Forecast (CF) GridSpec 和 [UGRID](https://github.com/ugrid-conventions/ugrid-conventions) conventions 文件。

### ESMF_RegridWeightGen

ESMF中提供了大量的工具可用于处理模式间耦合的问题或者网格数据的插值等问题。ESMF_RegridWeightGen就是包含的ESMF发行版中，用于从文件生成插值权重的工具。

此工具支持串行和并行，其接受两个网格文件作为输入，然后生成一个权重文件。支持矩形和非结构网格。网格文件可以是四种不同的格式：

- 用于SCRIP输入的SCRIP格式文件
- 遵从CF源数据标准的GRIDSPEC格式
- 自定义的ESMF非结构网格格式

ESMF_RegridWeightGen 支持大量的选项用于控制插值权重的生成。目前支持五种类型的插值方法：bilinear, higher-order patch, first-order conservative,及另外两种 nearest neighbor法。有多个选项可用于处理极地周围(around the pole)的插值，也有部分参数可用于处理无法插值到目标网格的情况。

对于传统的一阶插值方法，此工具支持掩膜(masking)和用户自定义面积(user-supplied area)操作。

此外，ESMF_RegridWeightGen 还提供了一些参数用于检查所产生的权重。这个选项可以利用权重(weight)以及计算出的插值误差(interpolation error)和conservation error(如果使用conservative插值方法)针对分析场执行插值(regridding)。这些误差同时也会输出。

### Regridding

Regridding(Remapping, Interpolation)是为了保证原始数据的质量而处理网格数据的过程。针对不同的问题需要进行不同的转换。尤其是当地球系统模式间(比如陆面和大气，或者不同数据集间的可视化操作等过程)进行数据交流时，插值是必要的。

Regridding主要分为两个部分：

- 生成插值权重矩阵，描述原始网格中的点在目标网格中的权重
- 原始网格数据和权重矩阵相乘，从而生成目标网格数据

不同的插值方法适用于不同的问题。基本的双线性插值是线性插值的二维变体。higher order patch recovery是一个二级多项式插值方法(second degree polynomial regridding method)，使用最小二乘法计算多项式。此方法比双线性插值得到的目标网格结果要好。还有两种最邻近插值法，可以将一个网格点的数据映射到另一个网格最邻近的点。这对于外推或者分类场来说非常有用。

一阶传统插值方法计算了从源网格到目标网格的字段的积分，使用源网格和目标网格的重叠部分来确定合适的权重。二阶传统插值方法使用的是源的梯度，从而给出比一阶算法更平滑的结果。所有的方法本质上都是数据和插值权重矩阵间的矩阵乘法操作。


本文仅对 ESMF 和 ESMF_RegridWeightGen 进行简要介绍，更多的使用操作后续会进行介绍。

### 参考链接：
1. https://www.earthsystemcog.org/projects/regridweightgen/
2. http://www.earthsystemmodeling.org/esmf_releases/public/ESMF_7_1_0r/ESMF_refdoc/node3.html#SECTION03020000000000000000


