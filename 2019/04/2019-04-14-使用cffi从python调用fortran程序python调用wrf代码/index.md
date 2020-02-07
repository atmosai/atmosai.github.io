# 使用CFFI从Python调用Fortran程序:Python调用WRF代码


### 为什么要使用Fortran代码

* Fortran语言是已经几乎被遗忘的语言，但其在科学领域却仍为活跃，尤其是大气科学领域。
* 如果效率优先是需要考虑的重要因素，那么可以尝试使用Fortran，虽然Fortran在这方面的作用相较于其他语言经常被高估(=_=)。



### 为什么从Python调用Fortran

* 使用Python比使用Fortran写模块更简单
* 测试方面，使用Python环境更友好



### CFFI：面向Python的C语言外部函数接口

* 从Python调用已编译的C和Fortran代码
* 使用回调机制从C和Fortran中调用Python程序
* 试图支持PyPy和CPython



### 操作步骤

#### 创建共享库

首先创建Fortran程序的共享库

* 可以使用`-shared`选项编译你的代码

  ```bash
  gfortran -shared  fortran_code.f90 -o libfortran.so  -fPIC
  ```

  不同的fortran编译器在库中可能会创建不同的函数签名，因此在使用之前应该使用`nm`命令检查名称。

* 也可以使用`iso_c_binding`模块对Fortran函数和子程序进行封装，此方法需要额外的代码，但这会：

  * 强制Fortran编译器生成正确的C签名
  * 允许传递参数作为值
  * 允许嵌入代码处理Fortran特定参数类型的转换
  * 有助于确保正确的参数类型



#### 函数封装器

在这个示例中，我们会创建一个封装器，虽然可能并不需要这样。

`fortran_rapper.f90`应该导入想要封装的Fortran模块，并且从`iso_c_binding`中调用需要使用的`c_types`：

```fortran
use your_fortran_module
use iso_c_binding, only: c_int, c_double
```

你需要定义一个C函数/子程序—打包需要使用的Fortran函数/子程序：

```fortran
subroutine c_wrapper_your_subr(arg1, arg2, arg3) bind(c)
```

然后需要声明所有需要使用的变量，`arg1`是值，`arg2`和`arg3`是指针：

```fortran
integer(c_int), intent(in), value :: arg1  
real(c_double), intent(inout) :: arg2, arg3(1:10)
```

现在就可以从Fortran模块`your_subr`中调用上述函数/子程序了：

```fortran
call your_subr(arg1, arg2, arg3)
```

最后一步，创建共享库封装Fortran模块，

```bash
gfortran -c -fPIC fortran_code.f90 -o fortran_code.o
gfortran -shared -fPIC  fortran_code.o fortran_wrapper.f90 -o libfortran.so
```



#### 从Python调用Fortran函数/子程序

有了共享库之后，就可以从Python中调用子程序了。

首先，使用外部函数解释器FFI加载库

```python
from cffi import FFI
ffi = FFI()
lib = ffi.dlopen("libfortran.so")
```

对于那些需要从Python调用的函数/子程序，需要提供C签名给CFFI：

```python
ffi.cdef("void c_wrapper_your_subr(int arg1, double arg2, double arg3[]);")
```

调用子程序之前需要定义参数，如果只是传递值，可以简单的赋值：

```python
arg1 = 1
arg2 = 3.14
```

因为`arg3`是一个数组，你应该创建一个CFFI对象存储Numpy数组指针

{{% admonition note "Note" %}}

NumPy数组的存储方式应该和Fortran数组相同。

{{% /admonition %}}

```python
numpy_array = numpy.arange(10, order='F')
arg3  = ffi.cast("double*", numpy_array.__array_interface__['data'][0])
```

现在就可以调用Fortran子程序了

```python
lib.c_wrapper_your_subr(arg1, arg2, arg3)
```



### 调用WRF微物理方案

为了测试C++版的微物理方案，需要使用WRF中广泛使用的微物理方案来进行对比。

下面以WRF中Kessler方法为例，所有代码在[这里](https://github.com/djarecka/scientific-software-diary/tree/master/CFFI_calling_FfromPy)，WRF的微物理方案`module_mp_kessler.f90`可以从[这里](<http://www2.mmm.ucar.edu/wrf/users/download/get_sources.html>)下载。

* 因为WRF的微物理方案需要传递大量的参数，封装起来会比较长。所有的数组将直接传递指针，以下是`kessler_wrap.f90`的内容：

```fortran
module kessler_wrap
  use iso_c_binding, only: c_int, c_double
  use module_mp_kessler
  implicit none
contains
 
  subroutine c_kessler(t, qv, qc, qr, rho, pii ,dt_in, z, xlv, cp, &
                       EP2, SVP1, SVP2, SVP3, SVPT0, rhowater,     &
                       dz8w, RAINNC, RAINNCV,                      &
                       ids,ide, jds,jde, kds,kde,                  &
                       ims,ime, jms,jme, kms,kme,                  &
                       its,ite, jts,jte, kts,kte                   &
                      ) bind(c)
 
    real(c_double), intent(in), value :: dt_in,  xlv, cp, rhowater,    &
                                         EP2,SVP1,SVP2,SVP3,SVPT0
    integer(c_int), intent(in), value :: ids,ide, jds,jde, kds,kde,    &
                                         ims,ime, jms,jme, kms,kme,    &
                                         its,ite, jts,jte, kts,kte
    real(c_double),  intent(in)       :: rho(ims:ime,kms:kme,jms:jme), &
                                         pii(ims:ime,kms:kme,jms:jme), &
                                         dz8w(ims:ime,kms:kme,jms:jme),&
                                         z(ims:ime,kms:kme,jms:jme)
    real(c_double),  intent(inout)    :: t(ims:ime,kms:kme,jms:jme),   &
                                         qv(ims:ime,kms:kme,jms:jme),  &
                                         qc(ims:ime,kms:kme,jms:jme),  &
                                         qr(ims:ime,kms:kme,jms:jme),  &
                                         RAINNC(ims:ime,jms:jme),      &
                                         RAINNCV(ims:ime,jms:jme)
 
    call kessler(t, qv, qc, qr, rho, pii, dt_in, z, xlv, cp,   &
                 EP2, SVP1, SVP2, SVP3, SVPT0, rhowater,       &
                 dz8w, RAINNC, RAINNCV,                        &
                 ids,ide, jds,jde, kds,kde,                    &
                 ims,ime, jms,jme, kms,kme,                    &
                 its,ite, jts,jte, kts,kte)
 
    end subroutine c_kessler
end module kessler_wrap                                     
```

* 强制编译器使用双精度创建共享库

  ```bash
  gfortran -fdefault-real-8 -c -fPIC module_mp_kessler.f90 -o module_mp_kessler.o
  gfortran -fdefault-real-8 -shared -fPIC module_mp_kessler.o kessler_wrap.f90 -o libkessler.so
  ```

* 在`cffi_kessler.py`中，定义一个python函数处理所有需要的大气变量数组，以及数组维度。因为需要传递大量的数组给Fortran，因此必须要创建一个CFFI字典来存储合适的指针。变量`ims`，`ime`…用来在Fortran子程序中分配足够的内存给数组：

  ```python
  import numpy as np
  from constants_kessler import xlv, cp, EP2, SVP1, SVP2, SVP3, SVPT0, rhowater
  from cffi import FFI
  ffi = FFI()
   
  # function creates cdata variables of a type "double *" from a numpy array             
  # additionally checks if the array is contiguous                                       
  def as_pointer(numpy_array):
      assert numpy_array.flags['F_CONTIGUOUS'], \
          "array is not contiguous in memory (Fortran order)"
      return ffi.cast("double*", numpy_array.__array_interface__['data'][0])
    
  def kessler(nx, ny, nz, dt_in, variable_nparr):
      ffi.cdef("void c_kessler(double t[], double qv[], double qc[], double qr[], double rh\
  o[], double pii[], double dt_in, double z[], double xlv, double cp, double EP2, double SV\
  P1, double SVP2, double SVP3, double SVPT0, double rhowater, double dz8w[], double RAINNC\
  [], double RAINNCV[], int ids, int ide, int jds, int jde, int kds, int kde, int ims, int \
  ime, int jms, int jme, int kms, int kme, int its, int ite, int jts, int jte, int kts, int\
   kt);", override=True)
   
      # load a library with the C function                                          
      lib = ffi.dlopen('libkessler.so')
   
      # create cdata variables for each numpy array            
      variable_CFFI = {}
      for item in ["t", "qv", "qc", "qr", "rho", "pii", "z", "dz8w", "RAINNC", "RAI\
  NNCV"]:
          variable_CFFI[item] = as_pointer(variable_nparr[item])
   
      [ims, ime, ids, ide, its, ite] = [1, nx] * 3
      [jms, jme, jds, jde, jts, jte] = [1, ny] * 3
      [kms, kme, kds, kde, kts, kte] = [1, nz] * 3
      
      # call the C function                                                                        
      lib.c_kessler(variable_CFFI["t"], variable_CFFI["qv"], variable_CFFI["qc"],
                    variable_CFFI["qr"], variable_CFFI["rho"], variable_CFFI["pii"],
                    dt_in, variable_CFFI["z"], xlv, cp, EP2, SVP1, SVP2, SVP3, SVPT0,
                    rhowater, variable_CFFI["dz8w"], variable_CFFI["RAINNC"],
                    variable_CFFI["RAINNCV"], ids, ide, jds, jde, kds, kde,
                    ims, ime, jms, jme, kms, kme, its, ite, jts, jte, kts, kte)    
  ```

* 这是使用Python函数的一个最简单示例，在[这里]([https://github.com/djarecka/scientific-software-diary](https://github.com/djarecka/scientific-software-diary/tree/master/CFFI_calling_FfromPy))你可以发现更多的代码。

​    

### 延伸阅读

* [CFFI documentation]( <https://cffi.readthedocs.org/en/release-0.8/>)

* [CFFI example I used](<http://maurow.bitbucket.org/notes/calling_fortran_from_misc.html>)
* [WRF model](<http://www.wrf-model.org/index.php>)
* [Libcloudph++ library]([libcloudphxx.igf.fuw.edu.pl](http://libcloudphxx.igf.fuw.edu.pl/))
* [Fortran signatures](<http://stackoverflow.com/questions/5811949/call-functions-from-a-shared-fortran-library-in-python>)
* [All examples are on github](<https://github.com/djarecka/scientific-software-diary>)

​    

[原文链接](<http://scientific-software-diary.com/?p=29>)


