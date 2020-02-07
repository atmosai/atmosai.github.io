# 记一次非常奇怪的WRF-CMAQ错误


近期在准备搭建一套WRF-CMAQ空气质量预报系统，从准备WRF模式运行开始就出现了`Segmentation Fault `的问题，

当时 `namelist.input` 中设置如下：

```namelist
sf_surface_physics     = 2, 2, 2
```

后改为

```
sf_surface_physics     = 1, 1, 1
```

WRF模式能够正常运行，**猜测可能是陆面方案的问题**。



本来以为WRF模式正常之后就可以，结果到运行MCIP/CMAQ，计算**干沉降速率**的时候也报错了！！都是出现类似以下错误信息：

```
 **********************************************************************
 *** SUBROUTINE: M3DRY
 ***   NEGATIVE OR UNDEFINED DRY DEPOSITION VELOCITY
 ***   POINT   =    90   57
 ***   SPECIES = SO2
 ***   Vd      =           NaN
 **********************************************************************
```

在运行MCIP时还可以通过更改干沉降速率的选项关闭干沉降速率计算解决此问题，但在运行CCTM时似乎没有提供此选项。只能排查出现此错误的原因了。

```
Numerical problems can occur in these algorithms because the WRF model input has LAI=0 for grids located over the ocean or in ice-bound regions. The m3dry.F algorithm includes divide-by-LAI without protection for LAI being zero. This can cause the program to generate Inf or NaN values.

The most recent version of MCIP (3.6) includes a correction for this problem: the MCIP file m3dry.F includes separate calculations for land and water, and the solution for water is used whenever vegetation (xveg) is zero. This appears to cover all occurrences of zero LAI. However the CMAQ algorithm still uses solutions that cannot handle zero LAI and are not bypassed.

The inline m3dry.F for bidirectional Hg also includes a call to a numerical solver (ATX) for soil concentrations with control statements that call for the solution for land grids only. (A separate solution is provided for grids representing water.) The solution for soil concentrations can fail to converge or generate negative values if leaf area index and other soil parameters are zero or near-zero. Normally, LAI is 0.1 or higher for land grids. However the WRF output includes a small number of grids that are flagged as 'land' grids even though LAI = 0 and the soil type category is 'water'. (These have been found for grids at the southern end of Hudsons Bay in Canada.)
```

回到MCIP，检查上述错误出现在`m3dry.ff90` 中，定位到计算干沉降速率的地方，发现是部分区域的`canopy wetness` 值为0导致计算过程中出现了`NaN`，下面一行是排查时输出结果，最后一个是 `xlai(90, 57)`的值：

```
          90          57  2.1474836E+09  0.0000000E+00  0.0000000E+00
```

说明WRF模式的结果中关于陆面的变量有问题，联系到之前运行WRF模式陆面方案存在问题，怀疑两者可能存在联系。最终还是回到WRF模式。



### 问题排查

重新选择`Noah`方案检查错误

```
sf_surface_physics     = 2, 2, 2
```

通过检查`rsl.out*` 输出内容发现错误信息如下：

```
Flerchinger USEd in NEW version. Iterations= 10
```

通过搜索发现可能是`wrfinput_d0*` 文件中的`SH2O`或` SMOIS ` 变量存在0或负值导致。然后检查这两个变量值发现，`SMOIS` 中部分区域的值为0。然后更改了`Vtable`文件，重新运行WPS和WRF模式，结果一切正常，问题解决！



​    

### 参考链接

1. http://forum.wrfforum.com/viewtopic.php?f=6&t=2531
2. http://blog.sina.com.cn/s/blog_3eac55fd0101748w.html
3. http://www-personal.umich.edu/~sillman/CMAQ_corrections_2010.htm
4. http://bbs.06climate.com/forum.php?mod=viewthread&tid=88671

