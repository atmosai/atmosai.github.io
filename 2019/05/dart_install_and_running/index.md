# DART资料同化模块


DART(Data Assimilation Research Testbed)是一个数据同化的集合工具，是由NCAR提供支持的集合数据同化社区版工具，其支持提供了大多数模式数据同化接口。

### 安装

DART的编译依赖于`make`和`mkmf`两个工具，make是一个非常常用的工具，其记录了各源码文件间的依赖关系，通常由用户自定义。`mkmf`是一个perl脚本，用来生成`make`输入文件和示例`namelist`文件`input.nml.program_default`。

`mkmf`需要两个输入文件，第一个模版文件定义了特定F90编译器所需要的命令，也包含了DART系统所需要的预编译工具指向的目录。此模版需要根据所使用的系统进行修改。第二个模版文件是DART所需要的`path_names`文件，无需更改。`mkmf`使用上述两个文件生成`Makefile`，成功之后即可使用make进行编译。

#### 构建和自定义`mkmf.template`文件

`DART/build_templates`目录下有很多模版文件，文件后缀为系统和编译器名称。

#### 自定义`path_names_*`文件

每个模式的`work`目录下都提供了一些`path_names_*`文件，比如`DART/models/lorenz_63/work`，这些文件一般不需要自定义修改。

### 构建Lorenz_63 DART项目

所有的DART程序都可以使用相同的方式编译。每个模式目录中都有一个`work`目录，其中包含了用于构建可执行程序的文件。下例展示了如何为lorenz_63创建两个可执行程序：`preprocess`和`obs_diag`。`preprocess`的构建和运行是用于创建源码支持特定的观测。按照如下方式编译：

```bash
cd DART/models/lorenz_63/work
./mkmf_preprocess
make
./preprocess         # 用于生成../../../assimilation_code/modules/observations/obs_kind_mod.f90文件，否则会报错
./mkmf_obs_diag
make
```

目前`work`目录下已经有一些DART可执行文件，Lorenz_63模式有7个`mkmf_xxxxxx`文件，具体描述如下：

```

Program	                    Purpose
preprocess	                 creates custom source code for just the observations of interest
create_obs_sequence	         specify a (set) of observation characteristics taken by a particular (set of) instruments
create_fixed_network_seq  	 specify the temporal attributes of the observation sets
                             perfect_model_obs	spinup, generate "true state" for synthetic observation experiments, ...
filter	                     perform experiments
obs_diag	                   creates observation-space diagnostic files to be explored by the MATLAB® scripts.
obs_sequence_tool	           manipulates observation sequence files. It is not generally needed (particularly for low-order models) but can be used to combine observation sequences or convert from ASCII to binary or vice-versa. Since this is a specialty routine - we will not cover its use in this document.
```

`quickbuild.csh`脚本可用于创建所有的可执行文件，而且可以提供选项构建mpi版本。

```bash
cd DART/models/lorenz_63/work
./quickbuild.csh -nompi
```

#### 简单运行测试

`DART/models/lorenz_63/work`目录下提供了一些输入文件，可用于进行简单的测试：使用20个集合成员同化50天逐6小时的观测。运行`perfect_model_obs`和`filter`可以得到已知结果的对比结果。

初始条件和观测序列文件为ASCII格式，不存在移植的问题，但是从ASCII转换到机器二进制可能会存在一些问题。对于高度非线形模式，初始条件的微小差异会导致模式结果的不同。

`Manhattan`发行版使用netCDF文件作为输入文件格式，使用`ncgen`可进行转换。当转换完成后，执行`perfect_model_obs`和`filter`即可：

```bash
ncgen -o perfect_input.nc perfect_input.cdl
ncgen -o filter_input.nc filter_input.cdl
./perfect_model_obs
./filter
```

以下是输出文件：

```
																			from executable "perfect_model_obs"
perfect_output.nc   			a netCDF file containing the model trajectory ... the 'truth'
obs_seq.out								The observations (harvested as the true model was advanced) that were assimilated.
 
																				from executable "filter"
preassim.nc								A netCDF file of the ensemble model states just before assimilation. This is the prior.
filter_output.nc					A netCDF file of the ensemble model states after assimilation.
obs_seq.final							The observations that were assimilated as well as the ensemble mean estimates of the 'observations' - for comparison.
 
																						from both
dart_log.out							The run-time log of the experiment. This grows with each execution and may safely be deleted at any time.
dart_log.nml							A record of the input settings of the experiment. This file may safely be deleted at any time.

```

如果修改了`input.nml`文件中的设置，输出文件可能会发生变化。

[DART/documentation/tutorial](https://www.image.ucar.edu/DAReS/DART/Manhattan/documentation/tutorial/index.html) 文档是学习DART和集合数据同化的非常有用的资料。

### 构建DART-WRF

DART也提供了WRF模式的同化模块，进入`DART/models/wrf/work` 目录可以按照`lorenz_63` 的构建方式进行编译，编译之前需要更改`build_templates/mkmf.template` 文件，使用intel编译器，并行库使用impi，编译选项更改为：

```
MPIFC = mpiifort
MPILD = mpiifort
FC = ifort
LD = ifort

FFLAGS  = -O2 $(INCS) 
```

> 编译前执行 `echo $NETCDF` 查看是否设置了netcdf环境变量，如果未设置，需要在 `mkmf.template` 文件中显示给定netcdf的路径

```
INCS = -I$(NETCDF)/include
LIBS = -L$(NETCDF)/lib -lnetcdff -lnetcdf
```

> 上述使用的是netcdfv4.3.0版本，如果使用的是比较老的netcdf版本，上述 `LIBS` 可能需要更改为
>
> LIBS = -L$(NETCDF)/lib -lnetcdf

然后进入`DART/models/wrf/work`目录，执行`quickbuild.csh`脚本，然后编译所有可执行程序，默认编译为并行版。

如果没有出现错误，则表示所有可执行程序都编译成功，最后会出现所有可执行程序均编译成功的提示。

### 更新记录
2019.05.15 更新DART安装及WRF部分编译


