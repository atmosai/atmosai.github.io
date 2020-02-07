# 如何添加背景图


以下代码展示如何在图中添加背景图，

首先生成图片

```python
from datetime import datetime

import numpy as np
import pandas as pd

from matplotlib import cm, colors
import matplotlib.image as mpimg
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_context('talk', font_scale=1.3)

## 自定义colormap
colors_pm = ['#009966', '#FFDE33', '#FF9A32', '#CC0033', '#660099', '#7D0023']
levels = [0, 35, 75, 115, 150, 250, 350]

cmap_pm = colors.ListedColormap(colors_pm)  
norm = colors.BoundaryNorm(levels, cmap_pm.N)

s = np.repeat(np.arange(350), 365).reshape((350, 365))

fig, ax = plt.subplots(figsize=(18, 12))

ax.imshow(s, cmap=cmap_pm, alpha=0.5, norm=norm, aspect='auto',
          extent=[xlims[0], xlims[-1], levels[-1], levels[0]])

# imshow显示的y轴可能是相反的，因此需要反转y轴
ax.invert_yaxis()

## 去除 x和y轴的ticklabels以及tick 
ax.set_xticks([])
ax.set_yticks([])
ax.tick_params(dict(length=0))

fig.savefig('img.png', dpi=300, bbox_inches='tight')
```

<br>
<div align=center><img src="https://github.com/bugsuse/blogpic/blob/master/2018/12/04/fig1.jpg?raw=true"></div>


<br>
```python
## 读取图片
img = mpimg.imread('img.png')

date_range = pd.date_range(datetime(2015, 1, 1), datetime(2015, 12 ,31), freq='1d')
data = pd.DataFrame(np.random.random(365) * 300, index=date_range, columns=['pm2.5'])

s = np.repeat(np.arange(350), 365).reshape((350, 365))

fig, ax = plt.subplots(figsize=(18, 12))

# alpha 设置透明度，图片在图中的位置，主要由 extent 参数控制，transform 参数将坐标轴的范围归一化
ax.imshow(img, alpha=0.5, extent=[-.02, 1.02, -0.02, 1.02], transform=ax.transAxes)

## x轴时间需要另外设置
ax.plot(data['pm2.5'].values, color='k', linewidth=2) 

ax.set_yticks(levels)
_ = ax.set_ylabel('PM$_2.5$($\mu$g/m$^3$)', fontdict=dict(fontfamily='Times New Roman'))

```
<br>
<div align=center><img src="https://github.com/bugsuse/blogpic/blob/master/2018/12/04/fig2.jpg?raw=true"></div>

<br>
以上示例展示了如何在图中添加背景图，添加背景图时可以添加多个需要的背景图片，关键在于设置 `imshow` 函数的 `extend` 参数。
<br>

上面实现的功能，可以通过以下代码完成：
<br>

```python
from datetime import datetime

import numpy as np
import pandas as pd

from matplotlib import cm, colors
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_context('talk', font_scale=1.3) # 调整图的配置


## 自定义 colormap
colors_pm = ['#009966', '#FFDE33', '#FF9A32', '#CC0033', '#660099', '#7D0023']
levels = [0, 35, 75, 115, 150, 250, 350]

cmap_pm = colors.ListedColormap(colors_pm)  
norm = colors.BoundaryNorm(levels, cmap_pm.N)

## 创建 DataFrame
date_range = pd.date_range(datetime(2015, 1, 1), datetime(2015, 12 ,31), freq='1d')
data = pd.DataFrame(np.random.random(365) * 300, index=date_range, columns=['pm2.5'])

s = np.repeat(np.arange(350), 365).reshape((350, 365))

fig, ax = plt.subplots(figsize=(18, 12))

xlims = mdates.date2num(data.index.values) # 转换datetime为timestamp

## imshow 默认 aspect 为 equal，应设置为 auto 才能使上述 figsize 参数生效
ax.imshow(s, cmap=cmap_pm, alpha=0.5, norm=norm, aspect='auto',
          extent=[xlims[0], xlims[-1], levels[-1], levels[0]])

ax.invert_yaxis() # 反转 y 轴

ax.xaxis_date() # 设置x轴刻度格式

# 不能使用 plot_date，因为imshow 暂时不支持 datetime axes
ax.plot(data['pm2.5'], color='k', linewidth=2) 

date_format = mdates.DateFormatter('%m/%d')
ax.xaxis.set_major_formatter(date_format)

ax.set_yticks(levels)
_ = ax.set_ylabel('PM$_2.5$($\mu$g/m$^3$)', fontdict=dict(fontfamily='Times New Roman'))

```

<div align=center><img src="https://github.com/bugsuse/blogpic/blob/master/2018/12/04/fig3.jpg?raw=true"></div>



果然还是不喜欢多写字。
<br>

参考链接:<br>
1. https://stackoverflow.com/questions/23139595/dates-in-the-xaxis-for-a-matplotlib-plot-with-imshow


