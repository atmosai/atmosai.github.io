# 提高matplotlib的出图效率


在数值预报后处理中经常需要批量出图，而基于matplotlib的图形渲染速度较慢，而提高出图的速度通常可通过两个方面来解决：

* 多进程进行绘图
* 图形渲染调整



### 多进程

在python中使用多进程方法加速批量出图是非常方便的。但这需要电脑有多个核，当然对于现代电脑和服务器而言已经不再是问题。

可选择`deco`和`multiprocessing`工具解决此问题。`deco`是对`multiprocessing`的封装，使用更加简单方便。

示例：

```python
from deco import *

@concurrent(processes=4) # We add this for the concurrent function
def process_lat_lon(lat, lon, data):
  #Does some work which takes a while
  return result

@synchronized # And we add this for the function which calls the concurrent function
def process_data_set(data):
  results = defaultdict(dict)
  for lat in range(...):
    for lon in range(...):
      results[lat][lon] = process_lat_lon(lat, lon, data)
  return results
```

第一个函数使用装饰器`@concurrent`，第二个函数使用了装饰器`@synchronized`，第二个函数中调用了第一个函数。第二个函数的装饰器是可选的，但最好使用装饰器进行封装。

第一个装饰器中给定了一个参数`processes`：表示进程数，如果没有给定，则使用所有的cpu。



### 图形渲染

以数值预报模式的批量出图过程中的气象要素空间分布为例。气象要素的空间分布必然涉及到地理信息的处理，比如添加海岸线、省市边界线、江流河海等。对于空间分布图而言，上述的地理信息是不变的。因此在批量出图时，相同地理范围的图可以使用相同的背景图。以温度的空间分布为例，这里所说的背景图是除了温度的空间分布外的海岸线、省市边界线、轴的标注等信息。



在绘图的时候都是按照图层进行先后叠加的，而叠加后的图层是可以删除的。批量出图时只需要将会变的信息清空，然后在背景图上叠加新的信息即可。这样，就能节省绘制地图的时间，每次只需要绘制一次地图即可。想想如果需要批量生成的图数量很多的话，这样就能节省很多时间。



#### 删除图层操作

对于`matplotlib.contour`类函数而言，删除操作如下：

```python
con = ax.contourf(lon, lat, temp)

for coll in con.collections:
    coll.remove()
```

#### 线或者文本操作

```python
lines = ax.plot(a, y)

l = lines.pop(0)
l.remove()
```

对于文本操作而言，以设置标题为例：

```python
at = ax.set_title('Test')
at.remove()
```

会出现以下错误信息：

```python
NotImplementedError: cannot remove artist
```

搜索了很久没找到解决办法，也就没有尝试。

但可以通过更新文本的方式覆盖原先的文本信息，比如：

```python
ax.set_title(None)
```

这样就能解决上述问题了。当然也可以使用如下方式：

```python
ax.set_visible(False)
```

​    

### 测试对比

整个循环批量出图需要对9个变量，输出4725张图。以下性能测试分析仅选取一个变量，绘制7张图。

#### 单核对比

```bash
time kernprof -l plot.py
```

```bash
real	0m31.441s
user	1m22.964s
sys	0m1.092s
```

```bash
 Line #      Hits         Time  Per Hit   % Time  Line Contents
 ==============================================================
     63         7     250148.0  35735.4      0.8      fig, ax = plt.subplots(figsize=(12, 9))
     64         7        273.0     39.0      0.0      m = Basemap(llcrnrlon=lon[0,0], llcrnrlat=lat[0,0],
     65         7        163.0     23.3      0.0                  urcrnrlon=lon[-1,-1], urcrnrlat=lat[-1,-1],
     66         7         14.0      2.0      0.0                  projection='lcc', resolution='l',
     67         7         15.0      2.1      0.0                  lat_1=30, lat_2=60,
     68         7   15828248.0 2261178.3     52.2                  lat_0=33.5, lon_0=106)
     69         7         43.0      6.1      0.0      shp = m.readshapefile('shps/cnhimap', 'china',
     70         7    3270262.0 467180.3     10.8                    linewidth=1.5, color='k', ax=ax)
     71         7         28.0      4.0      0.0      hb  = m.readshapefile('shps/hb', 'hebei',
     72         7     348692.0  49813.1      1.1                            linewidth=1.5, color='blue', ax=ax)
     73         7      55085.0   7869.3      0.2      x, y = m(lon, lat)
     74
     75         7        107.0     15.3      0.0      con = m.contourf(x, y, aqid, np.arange(rangs[0], rangs[-1]+1, 1),
     76         7         16.0      2.3      0.0                       vmin=rangs[0], vmax=rangs[-1], norm=aqi_norm,
     77         7    2528715.0 361245.0      8.3                       cmap=aqi_cmap, extend=extend, ax=ax)
     78
     79         7         29.0      4.1      0.0      m.drawparallels(yticks, labels=[1,0,0,0], linewidth=0.5,
     80         7     464004.0  66286.3      1.5                      ax=ax, fmt=lat2str, fontdict=dict(fontsize=FT))
     81         7         25.0      3.6      0.0      m.drawmeridians(xticks, labels=[0,0,0,1], linewidth=0.5,
     82         7     317093.0  45299.0      1.0                      ax=ax, fmt=lon2str, fontdict=dict(fontsize=FT))
     83         7      23429.0   3347.0      0.1      m.drawcoastlines()
```

通过性能分析结果可以看出：

**创建 map 占据了超过一半的时间，占比52.2%，而添加地图边界占比11.9%，添加轴标注占比2.5%。而这些都属于背景图的信息，只需要创建一次即可。**

将背景图信息的部分单独拿出来，只创建一次，每次在背景图上添加新图层，新的图存储后将添加的图层删除，然后重复利用。

以下是优化后代码的执行结果：

```bash
time kernprof -l plot_eff.py
```

```bash
real	0m14.141s
user	0m21.010s
sys	0m0.670s
```

相比于之前运行的`31s`，优化后的代码运行时间只有`14s`，速度提升了超过50%。

​    

#### 多核对比

多核并行运行采用`deco`工具，使用3个核进行测试。

```bash
time python plot.py
```

```bash
real	0m11.224s
user	0m55.686s
sys	0m1.610s
```

​    

测试单背景图的多核时出现了问题，`figure.canvas` 为 `NoneType`，导致出错：`AttributeError: 'NoneType' object has no attribute 'print_figure'`

猜测可能是只创建了一个`figure`对象，导致在使用多进程传递对象时出现了混乱，从而导致出现问题。而后对代码进行了改进，仅将创建`map`的代码放到了循环之外，只创建一次地图。毕竟创建地图的代码的时间占比就超过了50%，其余部分占比较低，改动此项仍能大幅节省画图时间。

以下是改进后的3个核的运行效率，相比于原始脚本而言，仍然提升了35%。

```bash
time python plot_eff.py
```

```bash
real	0m7.274s
user	0m20.875s
sys	0m0.857s
```

​    

### 注意事项

通过图形渲染流程来优化绘图时需要注意：`matplotlib`在绘图的时候如果使用`subplots`创建`Figure`对象，添加`colorbar`的时候，图形对象会进行自适应，删除`colorbar`之后`axes`的位置并不会自动适应到原始位置，此时如果添加新的图层和`colorbar`，会导致新的`Figure`对象中的`axes`的位置再次缩小。每重复一次删除/更新操作，`axes`的位置会缩小一些，重复越多，`axes`越小。

​    

{{% admonition note "Note" %}}

解决方法如下：可通过如下方式创建`Figure`图像，固定`contourf`的`axes`和`colorbar`的`axes`，这样每次删除/更新新图层时就不会出现上述问题。

```python
fig = plt.Figure(figsize=(12, 9))

ax = fig.add_axes([0.12, 0.11, 0.64, 0.77])
cax = fig.add_axes([0.78, 0.2, 0.022, 0.6])
```

{{% /admonition %}}

当然，`subplots`应该也有自适应的方式，但是尝试了很多方法都没有实现，暂时先放下了。尝试了更新`axes`的位置，然后更新图形：

```python
gp = ax.get_position()

ax.set_position(gp)
ax.autoscale()
ax.relim()
fig.canvas.draw()
```

但是并未解决上述问题。



### 参考链接

1. https://stackoverflow.com/questions/27345157/matplotlib-how-to-remove-just-one-contour-element-from-axis-with-other-plotted
2. https://stackoverflow.com/questions/4981815/how-to-remove-lines-in-a-matplotlib-plot
3. https://stackoverflow.com/questions/21565445/matplotlib-says-fig-canvas-is-none-so-i-cant-use-fig-canvas-draw

