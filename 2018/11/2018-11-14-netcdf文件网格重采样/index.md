# NetCDF文件网格数据重采样


无论是应用再分析数据作为模式输入还是后处理过程需要，对不同分辨率的NetCDF文件数据进行重新采样是非常必要的。

最近由于项目原因，需要对模式结果进行网格重塑，期间了解了不少工具，发现nco和cdo在进行NetCDF操作时功能非常的强大，而且效率也比较高。本文将从其中一个方面记录对NetCDF文件的重采样过程。

从概念上来说，将网格插值为新分辨率是一种简单操作。可以看作是一种稀疏矩阵乘法，即 $s W=t$，矩阵W则是从网格 s 映射到网格 t 所使用的权重。有很多库可以计算不同网格映射时的权重矩阵，而且也可以进行矩阵乘法操作。Oasis coupler使用 [MCT](https://www.mcs.anl.gov/research/projects/mct/)库在耦合模型中进行网格转换。[ESMF](https://www.earthsystemcog.org/projects/esmf/)也提供了很多函数执行插值操作，下面提到的是其中一种计算权重的命令行工具。

针对大量文件的操作，权重的计算显得尤为重要。如果文件的网格是相同的，需要插值的新网格也是固定的，那么权重只需要计算一次即可。

### 使用 ESMF_RegridWeightGen 创建权重文件

ESMF_RegridWeightGen能够直接从CF-NetCDF文件中读取网格信息然后直接生成重塑(regridding)权重。你也可以使用每个文件中的缺失值字段为源网格和目标网格定义**掩膜(mask)**。

对于高分辨率文件来说，创建权重矩阵需要使用大量的资源。最好的方式是使用并行运算。以下示例将7600x3600原数据插值到1536x1152。

```bash
#!/bin/bash
#PBS -l ncpus=16
#PBS -l mem=64
#PBS -l walltime=30:00
#PBS -l wd
 
module load esmf openmpi
ulimit -s unlimited
mpirun ESMF_RegridWeightGen -t GRIDSPEC -m neareststod  \
    -s SOURCE.nc --src_missingvalue SOURCE_FIELD \
    -d TARGET.nc --dst_missingvalue TARGET_FIELD \
    -w weights.nc --netcdf4
```

上述示例中使用PBS作业管理系统使用16个核提交作业进行计算， module load 加载当前所安装的一些软件，module是linux中的软件管理工具，在集群管理中非常方便。

`ulimit -s unlimited` 则是解除内存限制，防止出现 segamentation fault的错误。


**ESMF_RegridWeightGen** 参数：

- `-t`：定义了网格类型
  - `GRIDSPEC`：从CF-NetCDF文件中读取网格类型
  - `SCRIP`：读取SCRIP格式中的网格
- `-m`：网格重塑(regridding)方法
  - `bilinear`：双线性插值
  - `patch`：高阶插值
  - `neareststod`：映射源网格中与目标网格点中最邻近的点(仅应用一个点到目标格点)
  - `nearestdtos`:   映射源网格中与目标网格点中最邻近的点(可以映射多个源网格点到一个目标点)
  - `conserve`:  传统映射方法 (需要额外边界信息来计算网格面积) 
- `-s`：`SOURCE.nc`：需要regrid的样本文件
- `-src_missingvalue SOURCE_FIELD`:  `SOURCE.nc` 中定义源网格的mask，(应该通过`_FillValue` 或 `missing_value` 属性进行设置)
- `-d TARGET.nc`: 目标网格文件
- `-dst_missingvalue TARGET_FIELD`:  `TARGET.nc` 文件中定义目标网格 mask (应该通过`_FillValue` 或 `missing_value` 属性进行设置)
- `-w weights.nc`:  网格重塑权重文件
- `--netcdf4`:  输出为NetCDF4格式

完整参数列表以及源网格格式信息参见 [ESMF_RegridWeightGen documentation](http://www.earthsystemmodeling.org/esmf_releases/public/ESMF_6_3_0rp1/ESMF_refdoc/node3.html#SECTION03020000000000000000)

### Regridding 掩膜字段

如果源网格或者目标网格需要进行掩膜操作，那么可能目标网格中的一些点不会出现在源网格中，例如其中一个文件中有湖但是另一个文件中没有。如果这样的话 ESMF_RegridWeightGen 将会出错并将错误信息输出到日志文件。为了避免出现这种情况，可以使用 `neareststod` 方法插值或者使用 `--ignore`选项忽略缺失值错误，然后在插值完成之后将缺失点添加到文件中。

### 使用 ncks 进行网格重塑

生成权重文件之后，便可以使用 ncks 进行数据转换了：

```bash
ncks --map weights.nc input.nc output.nc
```

ncks 是 nco 中的一个命令行工具，使用之前需要先安装 nco。除了自行编译 nco 之外，可以通过 Anaconda 的 conda 包管理器进行安装。


