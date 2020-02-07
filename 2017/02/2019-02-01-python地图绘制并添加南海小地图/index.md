# Python地图绘制并添加南海小地图


在添加南海小地图时使用了matplotlib中的小工具，即嵌入定位器。可以放大一个图的局部，并绘制在这个图上，从而展示某一块区域。

```python
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt
from mpl_toolkits.axes_grid1.inset_locator import zoomed_inset_axes
from mpl_toolkits.axes_grid1.inset_locator import mark_inset
import netCDF4 as nc
import numpy as np

import shapefile
from matplotlib.path import Path
from matplotlib.patches import PathPatch

def basemask(cs, ax, map, shpfile):

    sf = shapefile.Reader(shpfile)
    vertices = []
    codes = []
    for shape_rec in sf.shapeRecords():
        if shape_rec.record[3] >= 0:  
            pts = shape_rec.shape.points
            prt = list(shape_rec.shape.parts) + [len(pts)]
            for i in range(len(prt) - 1):
                for j in range(prt[i], prt[i+1]):
                    vertices.append(map(pts[j][0], pts[j][1]))
                codes += [Path.MOVETO]
                codes += [Path.LINETO] * (prt[i+1] - prt[i] -2)
                codes += [Path.CLOSEPOLY]
            clip = Path(vertices, codes)
            clip = PathPatch(clip, transform = ax.transData)    
    for contour in cs.collections:
        contour.set_clip_path(clip)          
    
shpfile = u'E:\MATLAB\shp\中国行政区_包含南海九段线'
data = nc.Dataset('E:\python\scripts\sss\pres.mon.ltm.nc')

pres = data.variables['pres'][0,14:35,28:57]

lat = data.variables['lat'][:]
lon = data.variables['lon'][:]
lons, lone = lon[28], lon[57]
lats, late = lat[35], lat[14]
nx = pres.shape[1]
ny = pres.shape[0]

fig = plt.figure()
ax = fig.add_subplot(111)

map = Basemap(llcrnrlon = lon1, llcrnrlat = lat1, urcrnrlon = lon2, urcrnrlat = lat2,
              projection = 'cyl', ax = ax)

map.drawparallels(np.arange(lats, late + 1, 5), labels = [1, 0, 0, 0])
map.drawmeridians(np.arange(lons, lone + 1, 10), labels = [0, 0, 0, 1])              
              
x, y = map.makegrid(nx, ny)
# 转换坐标
x, y = map(x, y[::-1])

map.readshapefile(shpfile, 'China')

cs = map.contourf(x, y, pres, vmin = int(pres.min()), vmax = int(pres.max()))  
basemask(cs, ax, map, r'E:\MATLAB\shp\bou2_4p')               
        
axins = zoomed_inset_axes(ax, 0.8, loc = 3)
axins.set_xlim(lons + 38, lons + 52)
axins.set_ylim(lons, lons + 20)

map2 = Basemap(llcrnrlon = lons + 38, llcrnrlat = lats, urcrnrlon = lons + 52, urcrnrlat = lats + 20, ax = axins)                      
              
map2.readshapefile(shpfile, 'China')              
cs2 = map2.contourf(x, y, pres, vmin = int(pres.min()), vmax = int(pres.max()))

basemask(cs2, axins, map2, r'E:\MATLAB\shp\bou2_4p')

mark_inset(ax, axins, loc1=2, loc2=4, fc = "none", ec = "none")

plt.show()
```

* 添加南海小地图时先使用 `zoomed_inset_axes` 创建嵌入定位器

  `zoomed_inset_axes` 接受了三个参数: `ax`需要添加子图的axes，`0.8`表示放大等级，`loc`表示子图放置位置

* `mark_inset` 用于标记放大区域

* `basemask` 函数用于进行地图的白化 

{{% admonition note "Note" %}}

绘图时要设置 vmin 和 vmax 参数，而且两个图的值要设置为一致，否则画出的图可能导致相同区域配色不一致。

{{% /admonition %}}

![](https://ws1.sinaimg.cn/large/006tNc79ly1fzr2j69a8qj30hs0dx3zc.jpg)


