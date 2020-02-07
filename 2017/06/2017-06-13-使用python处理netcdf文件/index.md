# 如何处理netCDF文件


NetCDF文件是自描述的**二进制**数据格式。所谓自描述就是自带属性信息，这和一般的雷达基数据格式不同，一般的雷达数据也是二进制的，但不是自描述的，而是需要额外的数据格式文档来说明数据格式，而NetCDF文件中包含了描述变量和维度的元数据信息。通常包含以下三个部分：

* **维度**

* **变量**

* **属性**

维度部分记录的是每个变量的维度名及长度，而变量包含了维度，属性（如数据单位）信息及变量的值。属性部分包含了一些额外信息，比如文件创建者等。

很多工具都可以处理NetCDF文件，比如MATLAB，Python，NCL，GrADS，CDO，NCO，Panoply等等。这里主要讲一下如何利用MATLAB，Python，NCL处理NetCDF文件。

## Python

python中有多个库提供了处理NetCDF文件的功能，比如专门处理nc数据的netCDF4-python，scipy，osgeo，PyNIO(Linux)等。

### netcdf4-python

使用 netCDF4-python处理nc数据是非常方便的，而且其提供了非常多的功能，可以进行nc文件的读写操作。

```python
# 加载库
import netCDF4 as nc

data = nc.Dataset("wrfout_v2_Lambert.nc", "r")

# 输出文件中变量
print(data.variables.keys())

# 读取变量
lon = data.variables["XLONG"]
lat = data.variables["XLAT"]
sst = data.variables["SST"]

## 通过指定索引获取变量部分数据
# lon = data.variables["XLONG"][1, :, :]
# lat = data.variables["XLAT"][1, :, :]
# sst = data.variables["SST"][1, :, :]
```

### scipy

scipy 库中的io模块同样提供了 netcdf 文件处理方法，其所使用的外部模块和 netCDF4-python 使用的相同，都不需要使用 Unidata 提供的 netcdf C库。

```python
import scipy.io as spio

data = spio.netcdf_file("wrfout_v2_Lambert.nc", "r")
# data = spio.netcdf.netcdf_file("wrfout_v2_Lambert.nc", "r")

# 输出文件中变量信息
print(data.variables.keys())

dict_keys(['SFROFF', 'DN', 'SH2O', 'TMN', 'UDROFF', 'RDY', 'ITIMESTEP', 'HGT', 'RDX', 'PSFC', 'W', 'V', 'MU', 'QVAPOR', 'SMOIS', 'CF1', 'MAPFAC_U', 'HFX', 'DNW', 'SINALPHA', 'QFX', 'SNOWC', 'PB', 'CFN1', 'VEGFRA', 'MAPFAC_V', 'EPSTS', 'XLONG', 'F', 'XICE', 'COSALPHA', 'E', 'P_TOP', 'ZNW', 'QRAIN', 'SST', 'TSLB', 'RDNW', 'XLAND', 'RAINC', 'SNOW', 'U', 'FNM', 'LANDMASK', 'MAPFAC_M', 'ZNU', 'ZETATOP', 'PHB', 'SNOWH', 'TH2', 'Q2', 'RDN', 'QCLOUD', 'DZS', 'V10', 'RESM', 'TSK', 'CF3', 'RAINNC', 'XLAT', 'GLW', 'ISLTYP', 'P', 'PH', 'T', 'CANWAT', 'IVGTYP', 'CFN', 'CF2', 'MUB', 'LU_INDEX', 'Times', 'FNP', 'SWDOWN', 'PBLH', 'GRDFLX', 'T2', 'U10', 'LH', 'ZS'])

# 读取变量数据，获取变量数据的方式和 netCDF4-python 相同
lon = data.variables["XLONG"]
lat = data.variables["XLAT"]
sst = data.variables["SST"]
```

## MATLAB

matlab中提供了处理netcdf文件的包，但是只有2011年之后的版本内置了改包。

读取数据之前，可以先查看以下文件中包含了哪些信息：

```matlab
ncinfo('wrfout_v2_Lambert.nc');
```

![](https://ws1.sinaimg.cn/large/006tNc79ly1fzr153iigmj30az069wei.jpg)

数据信息为结构体，其中包含了各维度信息，包含的变量及属性等信息。Format 表示文件格式为 classic netcdf文件。

知道变量信息之后就可以读取变量了：

```matlab
lon = ncread('wrfout_v2_Lambert.nc', 'XLONG');
lat = ncread('wrfout_v2_Lambert.nc', 'XLAT');
sst = ncread('wrfout_v2_Lambert.nc', 'SST');
```

读取数据之后就可以进行分析和画图了。

## NCL

ncl处理netcdf文件的方法同样非常简单，这里仅简单介绍一下：

```ncl
data = addfile("wrfout_v2_Lambert.nc", "r")

lon = data->XLONG
lat = data->XLAT
sst = data->SST

; 当然也可以通过索引获取部分数据
lon = data->XLONG(1, :, :)
lat = data->XLAT(1, :, :)
sst = data->SST(1, :, :)
```



