# WRF-Chem编译安装及运行指南


从WRF模式官网下载WRF，WRF-Chem和WPS等安装包，在编译模式之前确保所有的依赖已经安装完成。

## 编译问题
### 版本V3.5

下载WRF-Chem 3.5版本安装编译问题：

```bash
./write_decomp.exe: Command not found
```

[解决办法](https://ruc.noaa.gov/wrf/wrf-chem/KPP_yacc_flex_problems.pdf)如下：

* 修改`scan.l`中相应的行(可不修改，但以下两步必须执行)

* 删除 `chem/KPP/kpp/kpp-2.1/src/Makefile ` 中所有 `-ll` 编译选项
* 修改 `chem/KPP/util/write_decomp/Makefile ` 中 `integr_edit.exe $(MECH) ` 为 `./integr_edit.exe $(MECH) `

重新编译即可。

已测试版本V3.7.1未出现上述问题。
    
## 运行

模拟范围设置以及参数选择省略...

### WPS前处理

确定好模拟范围以及参数之后，准备气象场数据(如果是进行预报可以选择GFS/EC的初始场，如果是历史回顾/个例研究可以使用FNL等再分析数据)，然后执行

```bash
./geogrid.exe
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
./link_grib.csh data/...  # 链接初始场数据
./ungrib.exe
./metgrid.exe
```

### 运行WRF

* 链接WPS前处理过程生成的气象数据

  ```bash
  cd WRFV3/run
  ln -s .../WPS/met_em* .
  ```

* 不开启化学选项`chem_opt = 0`运行`real.exe`

  ```bash
  ./real.exe 
  ## 运行成功后可以生成 wrfbdy_d01 和 wrfinput_d0* 等文件
  ```

### 准备生物源

下载MEGAN源码和数据，编译后用于处理生物源，下载链接[戳这里](https://www.acom.ucar.edu/wrf-chem/download.shtml)：

编译完成后会生成 `megan_bio_emiss` 可执行文件，然后创建 `megan_bio_emiss.inp` 配置文件(配置文件名随意)，文件格式为namelist格式，如下：

```bash
&control
domains = 2,  # 总的模拟嵌套数
start_lai_mnth = 01, # 模拟起始的月份
end_lai_mnth   = 01, # 模拟结束的月份
wrf_dir   = '.../WRFV3/run',   # WRF路径
megan_dir = '.../wrf-chem/MEGAN/data', # megan生物源数据路径
/
```

编译完成并且准备好数据后，即可执行 `megan_bio_emiss < megan_bio_emiss.inp > bio.log`

运行成功后会生成如下文件`wrfbiochemi_d0* `

### 准备人为源(可选)

可以不提供人为源

### 使用化学选项(chem_opt)运行`real.exe`

设置namelist中`chem_opt`为需要的选项，然后重新运行`real.exe`

### MOZART方案数据准备(可选)

如果使用MOZART化学方案，可能还需要准备一些额外的数据，比如`exo_coldens`和`wesely`提供相应的数据，上述工具的下载链接见上述MEGAN部分。

#### 运行`exo_coldens`

运行方式与MEGAN类似，同样需要准备配置文件，格式为namelist格式，部分参数如下：

```bash
&control
domains = 2,   # 总的嵌套数
wrf_dir = '.../wrf-chem/WRFV3/run',
/
```

其余参数见README
    
#### 运行`wesely`

准备配置文件`wesely.inp`，部分参数如下：

```bash
&control
domains = 2,
wrf_dir = '.../wrf-chem/WRFV3/run',
/
```
    
### 准备化学初始和边界条件

基于全球化学模式的结果，利用MOZBC为初始模拟提供化学初始条件和边界条件。也可不利用MOZBC来提供初始和边界条件，而使用默认的初始值。

### 运行`wrf-chem`

上述步骤完成后，即可运行`wrf.exe`进行积分运算。

### 更新记录

2019.05.12  首次更新wrf-chem整体运行步骤


