# f2py:连接fortran和python的桥梁


如果说 Python 能够让你就此起飞的话，那么使用 f2py 能让你在一定程度上飞的更高更远。

f2py 是用来连接 fortran 和 python 的 python 包，可以将 fortran 源程序转换为 python 可用的程序（windows下转换为*.pyd格式文件，linux下转换为*.so文件）。编译好后，使用时直接在python中导入即可。f2py 是 numpy 的一部分，当你安装了 numpy 时就已经包含 f2py 了，其可以被用来构建 Python C/API 扩展模块，从而更容易调用 FORTRAN77/90/95 子程序，FORTRAN77 common 数据块 或 FORTRAN90/95 module 模块。

将 fortran 程序转换为 python 可用的程序是非常必要的，尤其是在进行复杂数值计算和处理大量数据时，调用 fortran 程序比使用 python 要高效的多。更为重要的是，如果已经有了 fortran 程序，可以省下很多编写相应的 python 程序的时间。

下图分别是 python 版网格化程序(35行)和 fortran版程序(36行)的执行时间：

使用纯python 程序实现耗时 11.7 s，而使用 fortran 实现的程序耗时不到 1 ms，差距非常明显。而且上述使用的数据样本很少，当数据量变大时，两者之间的差异将更加明显。

![](https://ws1.sinaimg.cn/large/006tNc79ly1fzr1b19315j30hs02f3ys.jpg)

### Windows

如果直接使用 f2py 进行程序的转换，很可能会出现问题。当然，如果你已经配置好环境了的话是没问题的。

由于 f2py 的使用需要用到 c/c++ 编译器，fortran 编译器，因此，在使用之前要安装相应的编译器。


安装之前可以执行以下语句查看是否已经包含 fortran 编译器：

```bash
python.exe f2py.py -c --help-fcompiler
```

{{% admonition note "Note" %}}

可以切换到 python 所在目录下执行，或者指定完整路径。如果安装到了  Program Files (x86) 目录下，即使指定完整路径也会失败，因为路径中包含了空格。

{{% /admonition %}}

![](https://ws4.sinaimg.cn/large/006tNc79ly1fzr1ctagi5j30gx0jwtbz.jpg)

**红色框** 表示当前系统中安装的 fortran 编译器，**浅蓝色框** 表示 f2py 支持的 fortran 编译器，又分为当前系统**可用**和**不可用**的部分，黄色圆 以下表示当前系统不可用的 fortran 编译器。

查看支持的 c 编译器选项

```bash
python.exe f2py.py -c --help-compiler
```

如果使用 vc 的话，指定编译器为 msvc， 当然也可以使用 mingw32。

本文主要使用的 fortran 编译器是 gfortran， c 编译器选项是 mingw32 和 msvc，建议使用mingw32。

需要安装 MinGW [**注1**] 和 VC，文中编译时使用的是 VC2012。当然可以只安装mingw。

{{% admonition note "Note" %}}

**注**：目前gfortran对python3.5及以上的版本支持并不好，在使用3.x以上版本进行编译时，只有3.4.版本能够编译成功，使用3.5版本编译时失败。

{{% /admonition %}}

所有依赖安装完成之后，就可以进行编译了，编译如下图。

![](https://ws3.sinaimg.cn/large/006tNc79ly1fzr1gnre4aj30hs06mq3f.jpg)

### Linux

Linux 系统下只要安装了 python 和 numpy，并设置好了环境变量，可以直接使用，比在 windows 下使用要简单很多，不再赘述。

```fortran
subroutine gridize(lon_num, lat_num, lonse, latse, longi_size, longi, latit, step, flash)
    implicit none
    
    integer :: lon_num, lat_num
    integer :: longi_size
    real*8 :: lonse(lon_num), latse(lat_num)
    real*8 :: longi(longi_size)
    real*8 :: latit(longi_size)
    real*8 :: step
    integer, dimension(lon_num, lat_num) :: flash
    integer :: i, j, k
    
!F2PY intent(in) :: lonse
!F2PY intent(in) :: latse
!F2PY intent(in) :: longi
!F2PY intent(in) :: latit
!F2PY intent(in) :: step
!F2PY intent(out) :: flash
    
    flash = 0
    
    do j = 1, lat_num - 1
        do i = 1, lon_num - 1
            do k = 1, longi_size 
                if ((latse(j) .le. latit(k) .and. latit(k) .lt. latse(j) + step) & 
                .and. (lonse(i) .le. longi(k) .and. longi(k) .lt. lonse(i) + step)) then
                    flash(i, j) = flash(i, j) + 1
                end if
            end do
        end do    
    end do
    
end subroutine
```

改写好程序之后就可以编译使用了

```bash
f2py -m gridize -m gridize.f90
```

也可以通过--fcompiler 和 --compiler 来指定编译器。

<br>

### 参考链接:

1. https://sourceforge.net/projects/mingw-w64/
2. https://www.scivision.co/f2py-running-fortran-code-in-python-on-windows/

