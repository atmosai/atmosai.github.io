# WRF模式高维数据可视化


vis5d是专门开发用来进行5维数据可视化的工具，接触WRF模式看到的第一个高维可视化图就是由vis5d完成的。vis5d无法直接处理netcdf格式文件，需要借助第三方工具将nc文件处理成vis5d可以接受的格式，比如.v5d格式。

早些时候ARWPost可以将WRF模式结果处理成v5d的格式，但是后来更新将此功能删除了，这是因为出现了更好的高维数据可视化工具，这些在之前也都介绍过一些。除了ARWPost之外，我们还可以利用wrf2vis5d工具进行格式转换。

wrf2vis5d是专门进行WRF和vis5d格式转换的工具，其依赖于vis5d，在编译wrf2vis5d之前，要先安装vis5d。

### vis5d安装

从官网下载安装包，然后解压:

```bash
tar -zxvf vis5d-5.1.tar.Z
```

编译:

```bash
make
```

![](/img/2019/03/15/vis5d1.png)

一般可以选择

```bash
make linux-opengl
```

执行上述编译操作后，开始分块编译，根据错误提示，找到 `lui5` 文件夹，修改其中的 Makefile中对应的 `linux`部分编译选项。

![](/img/2019/03/15/vis5d2.png)

可以更改`gcc`为`icc`，并删除`-m486`选项，然后继续编译，碰到类似问题继续更高相应的选项。或者批量修改。

**需要修改 `lui5`, `src`, `util`,`import`中的 Makefile，对应`linux`或者`linux-opengl`的编译选项。**

![](/img/2019/03/15/vis5d5.png)

### wrf2vis5d安装

编译完成之后，就可以编译wrf2vis5d：

```bash
tar -zxvf wrf2vis5d.tar.gz
cd WRF2VIS5D/
```

修改Makefile中部分编译选项并拷贝/链接`netcdf.inc`到WRF2VIS5D路径下：

```bash
LIBNETCDF = -L/usr/local/netcdf/lib -lnetcdf -lm    # netcdf lib路径
LIBVIS5D = /usr/local/vis5d/src/binio.o /usr/local/vis5d/src/v5d.o   # vis5d库路径
INCLUDE = -I/usr/local/netcdf/include -I./   # netcdf include路径
FC = f90   # 编译器选项
FCFLAGS = -g -C -free
```

然后执行编译选项:

```bash
make
```

![](/img/2019/03/15/vis5d4.png)

`wrf_v5d_input`为 `wrf_to_v5d`的参数控制文件，类似WRF模式的namelist文件：

```bash
-1 ! number of times to put in vis5d file, negative means ignore the times
2000-01-24_18:00:00
U      ! variable list for vis5d file,  indent one space to skip
V      ! first five in list are special variables (diagnosed)
W
 THETA
 TK
TC
QVAPOR
QCLOUD
 QRAIN
 RAINC
 TSK
end_of_variable_list
wrfout_d01_000000  ! data files to pull fields from
end_of_file_list
-1 ! specify v5d vertical grid  0=cartesian, -1=interp to z from lowest h, >1 list levels (z)  desired in vis5d file
1 1.
2 2.
3 3.
4 4.
5 5.
6 6.
7 7.
8 8.
9 9.
10 10.
```

`end_of_variable_list` 和 `end_of_file_list` 之间为需要处理的netcdf文件，可以是多个，顺序列出即可。不需要处理的变量可删除或者**变量名前留空格**。

执行以下命令进行格式转换

```bash
wrf_to_v5d wrf_v5d_input wrf.v5d
```

然后就可以将wrf.v5d作为vis5d的输入，进行高维数据可视化处理了。下图是处理结果。

![](/img/2019/03/15/v5d.gif)


