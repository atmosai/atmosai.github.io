# 如何在Fortran中调用Python



Python是机器学习领域不断增长的通用语言。拥有一些非常棒的工具包，比如`scikit-learn`，`tensorflow`和`pytorch`。气候模式通常是使用Fortran实现的。那么我们应该将基于Python的机器学习迁移到Fortran模型中吗？数据科学领域可能会利用HTTP API(比如Flask)封装机器学习方法，但是HTTP在紧密耦合的系统(比如气候模式)中效率太低。因此，可以选择直接从Fortran中调用Python，直接通过RAM传递气候模式的状态，而不是通过高延迟的通信层，比如HTTP。

### 正文

有很多方法可以实现通过Python调用Fortran，但是从Fortran调用Python的方法却很少。从Fortran调用Python，可以看作是将Python代码嵌入到Fortran，但是Python的设计并不是像嵌入式语言Lua。可以通过以下三种方法实现从Fortran调用Python：

* Python的C语言API。这是最常用的方式，但需要实现大量的C封装代码。
* 基于Cython。Cython用于从Python中调用C语言，但也可以实现从C调用Python。
* 基于CFFI。CFFI提供了非常方便的方法可以嵌入Python代码。

无论选择哪种方法，用户都需要将可执行Fortran文件链接到系统Python库，比如通过添加`-lpython3.6`到Fortran模式的Makefile文件。下面通过`Hello World`示例演示如何通过Fortran调用Python。Fortran代码保存在`test.f90`文件，如下：

```fortran
! test.f90
program call_python
  use, intrinsic :: iso_c_binding
  implicit none
  interface
     subroutine hello_world() bind (c)
     end subroutine hello_world
  end interface

  call hello_world()

end program call_python
```

首先导入Fortran 2003内部定义的和C语言类型互通的模块`iso_c_binding`。但使用CFFI时，我们不需要写任何C代码，CFFI会生成C类型的打包接口。下一行则定义了一个C函数`hello_world`接口，这可以在C语言中实现，但是这里我们使用Python和CFFI。最后，调用`hello_world`。

为了使用`hello_world`，我们需要构建`CFFI`标注，并保存在`builder.py`中，此代码用于创建可以链接Fortran程序的动态库：

```python
import cffi
ffibuilder = cffi.FFI()

header = """
extern void hello_world(void);
"""

module = """
from my_plugin import ffi
import numpy as np

@ffi.def_extern()
def hello_world():
    print("Hello World!")
"""

with open("plugin.h", "w") as f:
    f.write(header)

ffibuilder.embedding_api(header)
ffibuilder.set_source("my_plugin", r'''
    #include "plugin.h"
''')

ffibuilder.embedding_init_code(module)
ffibuilder.compile(target="libplugin.dylib", verbose=True)
```

首先，我们导入`cffi`包，并且声明了**外部函数接口**(FFI)对象。这看起来似乎比较奇怪，这只是CFFI实现这种目的的方式。下一步，`header`字符串中包含了需要调用的函数接口的定义。`module`字符串中包含了真正需要执行的Python程序。装饰器`@ffi.def_extern`用于标记`hello_world`函数。`my_plugin`用于获取`ffi`对象。`def_extern`装饰器用于处理C类型，指针等。然后，`ffibuilder.embedding_api(header)`定义了API，`embedding_init_code`定义了Python代码。

看起来比较奇怪的是在字符串中定义Python代码，但CFFI需要以这种方式将Python代码构建为共享库对象。`ffibuilder.set_source`来设置源代码信息(?)。

然后执行以下语句创建共享库`libplugin.dylib`：

```bash
python builder.py
```

然后使用下列命令编译Fortran程序:

```bash
gfortran -o test -L./ -lplugin test.f90
```

以上是在Mac OSX上创建的共享库，如果在Linux上，共享库应该以`.so`结尾。如果一切没有问题，那么就可以执行文件了:

```bash
./test
hello world
```

以上演示了如何使用CFFI从Fortran中调用Python程序，而不需要写任何C程序。



### FAQ

#### 必须将所有Python代码写入`header`字符串吗？

不需要这样。你可以直接在不同的Python模块中定义Python代码(比如`my_module.py`)，然后在`module`字符串的开头导入即可。比如，`builder.py`可以改为如下形式：

```python
...

module = """
from my_plugin import ffi
import my_module

@ffi.def_extern()
def hello_world():
    my_module.some_function()

"""
...
```

这将在Python中使用可导入的形式使用Python程序。在添加到Fortran中之前，你也可以通过`python -c "import my_module"`测试一下。如果失败了，你可能需要将包含`my_module`模块的路径添加到Python的`sys.path`变量中。



#### 如何传递Fortran数组给Python

[stack overflow page](https://stackoverflow.com/questions/16276268/how-to-pass-a-numpy-array-into-a-cffi-function-and-how-to-get-one-back-out)回答了此问题。下面是一个示例，将代码定义在一个模块文件中，比如`my_module.py`：

```python
# my_module.py

# Create the dictionary mapping ctypes to np dtypes.
ctype2dtype = {}

# Integer types
for prefix in ('int', 'uint'):
    for log_bytes in range(4):
        ctype = '%s%d_t' % (prefix, 8 * (2**log_bytes))
        dtype = '%s%d' % (prefix[0], 2**log_bytes)
        # print( ctype )
        # print( dtype )
        ctype2dtype[ctype] = np.dtype(dtype)

# Floating point types
ctype2dtype['float'] = np.dtype('f4')
ctype2dtype['double'] = np.dtype('f8')


def asarray(ffi, ptr, shape, **kwargs):
    length = np.prod(shape)
    # Get the canonical C type of the elements of ptr as a string.
    T = ffi.getctype(ffi.typeof(ptr).item)
    # print( T )
    # print( ffi.sizeof( T ) )

    if T not in ctype2dtype:
        raise RuntimeError("Cannot create an array for element type: %s" % T)

    a = np.frombuffer(ffi.buffer(ptr, length * ffi.sizeof(T)), ctype2dtype[T])\
          .reshape(shape, **kwargs)
    return a

```

`asarray`函数使用CFFI的`ffi`对象转换指针`ptr`为给定形状的numpy数组。可以使用如下形式在`builder.py`中的`module`字符串中调用：

```python
module = """
import my_module

@ffi.def_extern()
def add_one(a_ptr)
    a = my_module.asarray(a)
    a[:] += 1
"""
```

`add_one`也可以定义在`my_module.py`中。最后，我们需要定义与函数相关的头文件信息，并且添加到`builder.py`的`header`字符串中：

```python
header  = """
extern void add_one (double *);
"""
```

最后，在Fortran中以如下形式调用：

```fortran
program call_python
  use, intrinsic :: iso_c_binding
  implicit none
  interface
    subroutine add_one(x_c, n) bind (c)
        use iso_c_binding
        integer(c_int) :: n
        real(c_double) :: x_c(n)
    end subroutine add_one
  end interface
  
  real(c_double) :: x(10)

  print *, x
  call add_one(x, size(x))
  print *, x

end program call_python
```

这一部分，我们介绍了如何在Fortran中嵌入Python代码块，以及如何传递数组给Fortran或从Fortran传递数组给Python。然后，有些方面还是不太方便。



#### 必须要在三个不同的区域定义python函数签名吗

任何要传递给Fortran的Python函数，都必须要要在三个区域进行定义。

* 首先，必须在`header.h`中进行C头文件声明
* 然后，执行函数必须要在`builder.py`的`module`字符串中，或一个外部模块中
* 最后，Fortran代码中必须包含定义子程序的`interface`块(接口块)

这对于改变Python函数来说就显得有些麻烦。比如，我们写了一个Python函数：

```python
def compute_precipitation(T, Q):
    ...
```

必须要在所有区域进行声明。如果我们想添加一个垂直涡度`W`作为输入参数，我们必须要修改`builder.py`以及调用Fortran的程序。显而易见，对于大的工程来说，这就变得极为麻烦。

对于一般通信而言，采用了一小部分fortran/python代码封装器。主要依赖于一个外部python模块：

```python
# module.py
import imp

STATE = {}


def get(key, c_ptr, shape, ffi):
    """Copy the numpy array stored in STATE[key] to a pointer"""

    # wrap pointer in numpy array
    fortran_arr = asarray(ffi, c_ptr, shape)
    
    # update the numpy array in place
    fortran_arr[:] = STATE[key]
    

def set(key, c_ptr, shape, ffi):
    """Call python"""
    STATE[key] = asarray(ffi, c_ptr, shape).copy()


def call_function(module_name, function_name):

    # import the python module
    import importlib
    
    mod = importlib.import_module(module_name)
    
    # the function we want to call
    fun = getattr(mod, function_name)
    
    # call the function
    # this function can edit STATE inplace
    fun(STATE)
```

全局变量`STATE`是一个包含了函数需要的所有数据的Python字典。`get`和`set`函数的功能主要就是将Fortran数组传递给`STATA`或者从`STATE`中取出Fortran数组。如果这些函数使用了Fortran/CFFI封装器，那么可以使用如下方式从Fortran中调用Python函数`cumulus.compute_precipitation(state_dict)`：

```fortran
call set("Q", q)
call set("T", temperature)
call set("ahother_arg", 1)

call call_function("cumulus", "compute_precipitation")

call get("prec", prec)
```

如果需要传递更多的数据给`compute_precipitation`，那么需要添加更多的`call set`命令，这可能会改变Python函数的执行。我们就不需要改变`builder.py`中的任何代码。



### 结论

上面描述了如何传递Fortran数据给Python函数，然后再获取计算输出。为了解决频繁更改接口的问题，我们将fortran数据放到了Python模块的字典中。通过调用给定的名称来获取数据，并且将计算结果也存储到相同的字段中，然后，Fortran代码通过索引字典中正确的关键词来获取结果。Cython中使用了类似的架构，但CFFI更为方便。最重要的是，从C语言中调用Cython需要导入`Python.h`头文件，还要运行`Py_initialize`和`init_my_cython_module`函数。然而，CFFI会在后台完成这些操作。

这篇文章只是起到一个简单的指示性作用，有很多问题都没有讨论，比如如何传递Fortran字符给Python。更多的代码信息，见[Github](<https://github.com/nbren12/call_py_fort>)。



​    

### 参考链接

1. <https://www.noahbrenowitz.com/post/calling-fortran-from-python/>

