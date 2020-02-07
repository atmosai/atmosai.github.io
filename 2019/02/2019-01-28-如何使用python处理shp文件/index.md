# 如何使用Python处理shp文件


涉及到空间数据处理的时候，为了比较清晰方便的看出空间数据所处的区域，通常都需要将省市边界线加到地图中。

Python中也提供了大量的shp文件处理方法，有底层的一些库，也有一些封装比较完整的库。比如：

* [fiona](https://github.com/Toblerity/Fiona)：基于ogr的封装，提供了更简洁的API
* [pyshp](https://github.com/GeospatialPython/pyshp)：纯python实现的shape文件处理库，支持shp，shx和dbf文件的读写
* `ogr` ：gdal中的用于处理边界文件的模块
* [geopandas](https://github.com/geopandas/geopandas)：基于 `fiona` 进行了封装


## fiona

### 安装

```bash
pip install fiona
```

### 读取shp文件

```python
import fiona

shps = fiona.open('CHN_adm2.shp')
```

#### 获取shp文件中属性信息

  ```python
  >>> shps.schema 
  
  {'properties': OrderedDict([('ID_0', 'int:10'),
                ('ISO', 'str:3'),
                ('NAME_0', 'str:75'),
                ('ID_1', 'int:10'),
                ('NAME_1', 'str:75'),
                ('ID_2', 'int:10'),
                ('NAME_2', 'str:75'),
                ('HASC_2', 'str:15'),
                ('CCN_2', 'int:10'),
                ('CCA_2', 'str:254'),
                ('TYPE_2', 'str:50'),
                ('ENGTYPE_2', 'str:50'),
                ('NL_NAME_2', 'str:75'),
                ('VARNAME_2', 'str:150')]),
   'geometry': 'Polygon'}
  ```

#### 获取shp文件编码

  ```python
  >>> shps.encoding
  
  'utf-8'
  ```

#### 获取shp文件投影信息

  ```python
  >>> shps.crs
  
  {'init': 'epsg:4326'}
  
  >>> shps.crs_kwt
  
  'GEOGCS["GCS_WGS_1984",DATUM["WGS_1984",SPHEROID["WGS_84",6378137.0,298.257223563]],PRIMEM["Greenwich",0.0],UNIT["Degree",0.0174532925199433],AUTHORITY["EPSG","4326"]]
  ```

#### 获取shape子文件

```python
>>> shp = next(iter(shps))
```

#### 查看shape子文件信息

```python
>>> shp.keys()

dict_keys(['type', 'id', 'geometry', 'properties'])
```  
  

{{% admonition note "Note" %}}

`type`：表示shape子文件的类型

`id`：shape子文件的序号

`geometry`：包含shape子文件的类型和经纬度信息(字典类型)，包含了 `type` 和 `coordinates` 两个关键词

`properties`：shape子文件的属性信息

{{% /admonition %}}


```python
>>> shp.get('type')
>>> shp.get('id')
>>> shp.get('geometry')
```

    

shps 变量包含了一些方法可以获取shape文件中的每个边界，比如 `.next`，`.iterms`等。



{{% admonition warning "Warning" %}}

`.next` 方法将在 fiona 2.0版本中移除，可改用 next(iter(shps))进行单个迭代，或者使用 shps.iterms 进行循环迭代。

{{% /admonition %}}



fiona中提供了shp文件的读取方法，但是并没有提供可视化方法，如果使用fiona处理，还需要单独进行画图的操作。



### 写shp文件

构建shp文件的操作很少使用，但有时候可能需要从已有的shp文件中提取一个子区域。

```python
subshp = fiona.open('subshp.shp', mode='w', 
                    crs=shps.crs, crs_wkt=shps.crs_wkt, 
                    driver=shps.driver, 
                    encoding=shps.encoding, 
                    schema=shps.schema)
```

fiona默认的方式是读，只需要改为写，然后提供源文件中的一些信息即可。

```python
>>> shp = next(iter(shps))
>>> subshp.write(shp)  ## 写入字段
>>> subshp.flush()  ## 更新文件
>>> subshp.close()  ## 关闭文件
```

上述方法只是从源文件中随意选择了一个子区域，可以根据需要选择特定的区域，然后写入文件即可。

{{% admonition note "Note" %}}
⚠️：不要忘记更新文件。  
{{% /admonition %}}

## pyshp

###  安装

```python
pip install pyshp
```

### 文件读取

```python
import shapefile

shps = shapefile.Reader('CHN_adm2.shp')
```

读取后返回的 `shps` 中也包含了很多方法，其中 `.fields` 包含了shape文件中的一些字段信息，类似 fiona 中的 `.schema` 方法：

```python
>>> shps.fields

[('DeletionFlag', 'C', 1, 0),
 ['ID_0', 'N', 10, 0],
 ['ISO', 'C', 3, 0],
 ['NAME_0', 'C', 75, 0],
 ['ID_1', 'N', 10, 0],
 ['NAME_1', 'C', 75, 0],
 ['ID_2', 'N', 10, 0],
 ['NAME_2', 'C', 75, 0],
 ['HASC_2', 'C', 15, 0],
 ['CCN_2', 'N', 10, 0],
 ['CCA_2', 'C', 254, 0],
 ['TYPE_2', 'C', 50, 0],
 ['ENGTYPE_2', 'C', 50, 0],
 ['NL_NAME_2', 'C', 75, 0],
 ['VARNAME_2', 'C', 150, 0]]
```

```python
>>> shps.numRecords  # shape文件中包含了多少个记录数，即子文件数
```

#### 读取shape子文件

  ```python
  >>> shp = shps.shapeRecord()
  ```

#### 获取子文件属性信息

```python
>>> shp.record  # 返回列表

[49,
'CHN',
'China',
1,
'Anhui',
1,
'Anqing',
'',
0,
'',
'Dìjíshì',
'Prefecture City',
'安庆市',
'Ānqìng']
```

{{% admonition note "Note" %}}

类似 fiona 中获取shape子文件的属性信息，但fiona返回为字典。

{{% /admonition %}}

#### 获取子文件坐标信息

```python
>>> shp.shape.points # 包含了经纬度坐标
>>> shp.shape.bbox  # shape子文件范围
```

上述两个库，均可以进行shape文件的读写操作，但并没有提供可视化的方法。如果想看图的时候可以使用ArcGIS或者QGIS，导入文件即可。或者使用geopandas进行处理，geopandas提供了shape文件的处理和可视化，具有更为简便的API。

## geopandas

### 安装

```python
pip install geopandas
```

### 文件处理和可视化

```python
import geopandas

shps = geopandas.read_file('CHN_adm1.shp')

shps.plot()
```

![](https://ws1.sinaimg.cn/large/006tNc79ly1fzqwda3btnj30hs0dc752.jpg)

包括库导入也只需要3行代码即可。  

## 参考链接：
1. https://gis.stackexchange.com/questions/113799/how-to-read-a-shapefile-in-python
2. https://gis.stackexchange.com/questions/131716/plot-shapefile-with-matplotlib
3. https://gis.stackexchange.com/questions/97545/using-fiona-to-write-a-new-shapefile-from-scratch/97563

