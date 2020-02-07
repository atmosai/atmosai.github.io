# 基于python的WRF模式后处理


WRF模式是数值天气预报和大气模拟系统，其开发目的就是用语研究和实际应用。运行WRF模式时，可以利用多种初始场数据来驱动，然后配置好选项之后便可以模拟天气过程（说的好像很简单的样子==）。

个例模拟结束之后怎么办呢，我们怎么知道模拟的效果究竟如何呢？既然模拟了，那么就会有数据输出，想要检验模拟效果如何，就要对模拟数据进行处理。这就是所谓的后处理过程，也就是对模拟结果的处理，提取想要的数据，并进行分析及可视化的过程。

在地学系统中，尤其是大气科学领域，对WRF模式后处理主要使用的是GrADS和NCL，而GrADS同FORTRAN一样，属于历史悠久系列产品之一。而NCL是由NCAR开发的脚本语言，近些年来用户颇多，其主要应用与地学领域，其中包含了大量的气象相关脚本，很多气象要素的计算函数，以及气象常用图形的绘制函数，官网也给出了大量的示例，在气象领域中应用广泛。既然涉及气象领域，那么自然就少不了对WRF模式的支持，NCL中有一个脚本库，专门应用于WRF模式的后处理。

Python进行WRF模式后处理，主要使用三个库：matplotlib(python中最火的可视化库)，netCDF4(处理nc文件)，Basemap(处理地图投影)。

```python
from matplotlib import cm,colors
import matplotlib.pyplot as plt
from mpl_toolkits.basemap import Basemap
import numpy as np
import netCDF4 as nc

from wrfpost import sharexy

fip = "/run/media/storm/Flash/WRF/623/"
fin1 = "wrfout_d02_2016-06-23_06_00_00"
shpfn = '/home/storm/MATLAB/shp/bou2_4p'

ti = [0,6,12,18]
xs = 0;   xe = -1
ys = 0;   ye = -1
zs = 0;   ze = -1

data = nc.Dataset(fip + fin1, "r")
truelat = data.TRUELAT1
truelat = data.TRUELAT2
stalon  = data.CEN_LON
stalat  = data.CEN_LAT

# 国内雷达图配色 
collev = ["#FFFFFF","#98F5FF","#87CEEB","#00FF00",\
           "#008B00","#FFFF00","#FFA500","#FF8C00",\
           "#FF0000","#AF0000","#8B0000","#FF00FF",\
           "#8A2BE2","#360CF9"]
#  文献中雷达反射率配色
# ["#9808F0", "#51A5CE", "#3455A9", "#7FC500", "#54BE09", "#39932C",\
#  "#F7E731","#E6BC25", "#F78C29", "#E12C2D", "#A22429", "#4A1910",\
#  "#B85BC1", "#935CB6"]
cmaps = colors.ListedColormap(collev,'indexed')
cm.register_cmap(name = 'dbzcmap',data = collev,lut = 128)

fn = "Arial"
fs = 10
ld = 0.
nrows = 2
ncols = 2

fig,axes  =  plt.subplots(nrows = nrows, ncols = ncols, 
                          subplot_kw= dict(aspect = 'auto'))

for i, ax in zip(np.arange(0, nrows*ncols), axes.flat):
    dbz  = data.variables["COMPOSITE_REFL_10CM"][ti[i],xs:xe,ys:ye]
    lat  = data.variables["XLAT"][ti[i], xs:xe, ys:ye]
    lon  = data.variables["XLONG"][ti[i], xs:xe, ys:ye]

    map = Basemap(ax= ax, projection="lcc", llcrnrlon = lon[-1,0], llcrnrlat = lat[0,0],\
              urcrnrlon = lon[0,-1], urcrnrlat=lat[-1,-1], lat_0 = stalat,\
              lon_0 = stalon, resolution ="h")
    x, y = map(lon, lat)
# 添加经度，纬度坐标，海岸线，国界线
    labelsx, labelsy = sharexy(ax, nrows, ncols, i)
    map.readshapefile(shpfn, 'bou2_4p', linewidth = 1, color = 'k', ax = ax)
    map.drawmeridians(np.arange(int(lon[-1,0]), int(lon[0,-1])+1, 2), labels= labelsx, \
                    fontname = fn, fontsize= fs, linewidth = ld, ax = ax)
    map.drawparallels(np.arange(int(lat[0,0]), int(lat[-1,-1])+1), labels= labelsy, \
                    fontname = fn, fontsize= fs, linewidth = ld, ax = ax)
    con  = map.contourf(x, y, dbz, np.arange(0,71,5), cmap= cmaps)
    
    ax.set_adjustable('box-forced')  # 防止绘图和坐标区域不一致

fig.subplots_adjust(top = 0.9, bottom = 0.1, left = 0.12, right = 0.77, 
                    hspace = 0.05, wspace = 0.05)    
fig.colorbar(con, ax=axes.ravel().tolist(), pad = 0.01)   
#plt.savefig("panel.eps")    
plt.show()
```

除了上述三个库之外，python中有一个库和NCL 中的 WRF相关库具有几乎相同的功能，就是 wrf-python。wrf-python对上述过程进行了大量的封装，可以更加高效简便的进行WRF模式的后处理。


