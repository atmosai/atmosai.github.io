# 基于多普勒雷达数据的风场反演


本文记录利用python根据国内S波段雷达径向速度数据反演风场。

编程语言：[python](https://www.python.org/)
库：[pyart](https://github.com/nasa/SingleDop)，[matplotlib](https://matplotlib.org/)，[SingleDop](https://github.com/nasa/SingleDop)

其中pyart用于处理S波段雷达数据(ARM-DOE提供的pyart本身不支持国内S波段雷达数据，可下载更新后的[pyart](https://github.com/bugsuse/pyart))，SingleDop用于风场反演(NASA开源的根据观测或模拟多普勒雷达数据反演风场的库)，matplotlib进行图形绘制。

SingleDop支持PyART输出对象，在进行风场反演是非常方便。在处理PyART不支持的雷达数据时只需要转换为PyART对象即可。详细的参数介绍可以查看SingleDop函数帮助或者阅读源码。

本次以一次超级单体龙卷为例，利用SingleDop反演风场。下图为实际观测的多普勒径向速度，可以看出非常明显的正负速度对，即出现了中气旋的特征。

![雷达低层径向速度](https://github.com/bugsuse/blogpic/blob/master/2018/04/29/radial_speed.png?raw=true)
<center><font size=2>雷达低层径向速度</font></center>

下图为反演的风场结果，从反演风场发现气旋式切变相对较弱，这是由于在使用SingleDop进行风场反演时参数调整的问题。不同的参数设置会导致反演结果出现非常大的差异。所以在进行风场反演时可能需要进行多次的参数调整，由于计算量非常大，所以这是个非常耗时的过程。

![反演风场](https://github.com/bugsuse/blogpic/blob/master/2018/04/29/wind_stream.png?raw=true)
<center><font size=2>反演风场</font></center>

下图为反演风场填充等值线，可以更加直观的看出正负速度对的存在。

![反演风场填充等值线](https://github.com/bugsuse/blogpic/blob/master/2018/04/29/wind_contourf.png?raw=true)
<center><font size=2>反演风场填充等值线</font></center>

不贴代码了，本文代码见[基于雷达数据反演风场](https://github.com/bugsuse/blogpic/blob/master/2018/04/29/radar_radial.ipynb)。

关于参数的调整提供的notebook脚本中给了一个示例，其余的参数调整可调整试试。



