# 基于Xarray的Grid格式文件处理


[Xarray](http://xarray.pydata.org/en/stable/index.html)是专门为科学计算开发的用于处理多维数组的开源Python程序包，通过对多维数组进行标记，提供更加方便的操作方式，这一点和地球科学中的NetCDF数据格式非常相似。

### 安装

首先安装xarray和cfgrib等所需要的库，命令如下

```bash
conda install -c conda-forge xarray cfgrib eccodes
```



### 数据处理

Xarray中提供了不同的后端，可用于处理Grib格式数据。以下以`cfgrib`后端为例处理GFS全球预报的Grib格式文件：

```python
import xarray as xr

fn = 'gfs.t00z.pgrb2.0p25.f000'

rh = xr.open_dataset(fn, engine='cfgrib', backend_kwargs={'filter_by_keys':{'typeOfLevel': 'isobaricInhPa', 'paramId=157'}})
pre = rh['isobaricInhPa'].data

gh = xr.open_dataset(fn, engine='cfgrib', backend_kwargs={'filter_by_keys':{'typeOfLevel': 'isobaricInhPa', 'paramId':156}})

gh['gh'].sel(isobaricInhPa=pre)
```

> 注意：使用cfgrib后端处理GFS全球预报的grib格式文件时，由于部分变量的维度大小不同，一次读取多个变量可能会出现问题。比如在GFS的grib格式数据中，位势高度和温度的`isobaricInhPa`维度大小为33，而相对湿度的`isobaricInhPa`维度大小为31。使用如下命令读取时会出现问题：仅提取部分变量，然后给出跳过变量的提示。读取GFS的grib数据时，仅读取位势高度和温度变量。

```python
data = xr.open_dataset(fn, engine='cfgrib', backend_kwargs={'filter_by_keys':{'typeOfLevel':'isobaricInhPa'}})
```

正是由于GFS的grib格式文件中维度类型太多，所以如果直接读取会出现问题。因此需要使用`backend_kwargs`参数进行筛选。如果不知道该提供什么参数进行筛选，可以先执行如下语句：

```python
data = xr.open_dataset(fn, engine='cfgrib')
```

会触发`DatasetBuildError`错误，并给出可能的筛选关键词。



除了直接提取变量所有层之外，也可以单独提取某一层，比如：

```python
data = xr.open_dataset(fn, engine='cfgrib', backend_kwargs={'filter_by_keys':{'typeOfLevel':'isobaricInhPa', 'level':850}})
```

提取单层不会出现上述问题，只要是相同维度类型的变量可以一次性全部提取，并返回xarray.Dataset类型数据，然后借助xarray的强大功能可以实现很方便的数组操作。

​    

### 更新记录

2019.07.10 初次更新


