# 基于netcdf库的nc文件读写


NetCDF库提供了两种语言的函数API，一种是C，另一种是Fortran，其中又分为F77和F90两种方式的接口。通过函数开头的字符可以区分函数接口，C语言的函数接口以`nc_`开头，F77函数接口以`nf_`开头，F90函数接口以`nf90_`开头。

## 函数概览

NetCDF库的函数操作分为几个类别，以下以C语言API为例，Fortran的API类似，可能函数的参数有些区别。

### 文件和数据I/O函数

nc文件I/O操作包括文件的读写以及从内存中获取数据的函数，涉及上述操作时，还有一些辅助函数：比如控制打开文件对象定义模式，来操作文件的函数，以及查询函数(查询变量数，变量维度，全剧属性以及记录维度(这里所说的记录维度即是无限维度，即`unlimited dimension`)等等)。

> 如果是打开已有文件，对已有文件进行编辑时，如添加新变量，维度，属性等信息，需要进入定义模式，然后修改完成后，为了保证文件中的内容是最新的，可使用`nc_sync`/`nf_sync`等函数更新文件。

> NetCDF库的I/O操作函数除了能够接受文件之外，也可以是URL，但需要DAP支持。

### 维度操作函数

NetCDF库中提供的维度函数主要用于定义nc文件中数据的形状。定义/添加数据集维度，需要进入**定义模式**。每个维度都有一个名称和长度信息。在NetCDF文件中，维度通常分为**记录维度**/**无限维度**和**非记录维度**(常规维度)，

* **记录维度**/**无限维度**：维度的长度是无限制的，变量在此维度可以不断增加，即通常时间维是**记录维度**
* **非记录维度**：维度的长度是固定不变的，通常空间维度是非记录维度

> netCDF classic 和 64位文件，最多只能有一个记录维度，但在netCDF4文件中可以有多个记录维度。

netCDF文件的维度通常不能超过`NC_MAX_DIMS`所定义的值，变量的维度不能超过`NC_MAX_VAR_DIMS`的值，并且`NC_MAX_VAR_DIMS`不能超过`NC_MAX_DIMS`。

> 通常，维度的长度和名称是固定的，名称可以在定义模式中改变，但是维度的长度(记录维度除外)是不能改变的。通过`nc_rename_dim`函数可重命名维度名。

相关操作函数如下：

* `nc_def_dim`：定义新维度
* `nc_inq_dim`：查询维度长度和名称
* `nc_inq_dimid`：根据名称查询维度ID
* `nc_inq_dimlen`：查询维度长度
* `nc_inq_dimname`：查询维度名称
* `nc_inq_ndims`：查询维度数
* `nc_inq_unlimdim`：查询无限维度ID
* `nc_rename_dim`：重命名维度

### 变量操作函数

NetCDF库中提供了大量关于变量的操作函数，大致分为以下几类：

* **变量定义函数**：用于定义/添加新变量
* **变量数据获取函数**：此类函数可从变量中提取数据，此类函数提供了针对不同的数据类型的函数
* **变量查询函数**：此类变量用于查询文件中变量的信息
* **变量数据I/O函数**：用于写数据到变量，包括标量，向量，矩阵，数组，字符串/文本等操作函数
* **变量功能函数**：比如重命名，字符串资源释放，变量缓存等

关于变量操作的更多信息和相关命令的使用说明见[这里](https://www.unidata.ucar.edu/software/netcdf/docs/group__variables.html)

### 属性操作函数

netCDF文件的属性操作通常是给文件添加**全局属性**或者给变量添加属性。NetCDF库中的属性操作包括：

* **属性获取函数**：获取属性值
* **属性查询函数**：用于查询变量或者全局/组的属性信息
* **属性添加函数**：此类函数提供了大量的添加属性操作，而且针对不同的数据类型，提供了特定的函数
* **其它功能函数**：比如删除/重命名属性

### 组操作函数

NetCDF库中关于组的操作是在NetCDF4中添加的，不支持NetCDF3 classic和64-bit offset文件。所谓的组就是支持多个并排存在的数据集合(就是一个年级以前只有一个班，现在可以有多个班)。

NetCDF库中提供了组的创建，查询，重命名等函数，更多信息见官方文档。

### 错误处理函数

无论是C，F77还是F90的API，如果函数成功执行，都会返回0，否则返回对应错误的代码，然后可使用`nc_strerror`/`nf_strerror`等函数将错误代码转换为字符串信息。

> 错误处理是程序设计所必需的，这对于错误的排查是非常关键的，所以在程序中应该时刻考虑这些异常情况的处理。

### 用户自定义类型函数

NetCDF库支持用户自定义数据结构，并且提供了相当多的函数来自定义数据结构。更多信息见官方文档。

## 文件读取
### 读取已知名称的netCDF数据

使用NetCDF库API从已有文件中去读已知变量名称的数据时，通常按照如下步骤：

```C
nc_open / 打开已有文件 /
    nc_inq_dimid    / 获取维度ID
    / 其它操作
    nc_inq_varid    / 获取变量ID
    / 其它操作
    nc_get_att      / 获取属性值
    / 其它操作
    nc_get_var      / 获取变量值
    / 其它操作
nc_close / 关闭打开的文件   
```

如果使用Fortran的API接口，则改为如下命令：

```fortran
nf90_open               ! open existing netCDF dataset
     ...
   nf90_inq_dimid       ! get dimension IDs
     ...
   nf90_inq_varid       ! get variable IDs
     ...
   nf90_get_att         ! get attribute values
     ...
   nf90_get_var         ! get values of variables
     ...
nf90_close              ! close netcdf dataset
```

### 读取未知名称的netCDF数据

与已知变量名称不同时，如果不知道变量等信息的名称，那么需要调用查询函数获取关于netCDF对象的信息，操作步骤如下：

```fortran
nf90_open                 ! open existing netCDF dataset
  ...
nf90_inquire              ! find out what is in it
     ...
   nf90_inquire_dimension ! get dimension names, lengths
     ...
   nf90_inquire_variable  ! get variable names, types, shapes
        ...
      nf90_inq_attname    ! get attribute names
        ...
      nf90_inquire_attribute ! get other attribute information
        ...
      nf90_get_att        ! get attribute values
        ...
   nf90_get_var           ! get values of variables
     ...
nf90_close                ! close netcdf dataset
```

## 创建新文件

使用NetCDF库创建新nc文件，通常遵循如下步骤：

### 创建新的nc文件对象

可以使用如下函数创建新的nc文件：

* `nc_create`：C语言函数接口创建新nc文件函数
* `nf_create`：F77创建新nc文件函数
* `nf90_create`：F90创建新nc文件函数

### 定义维度变量

使用如下函数定义维度：

* `nc_def_dim`
* `nf_def_dim`
* `nf90_def_dim`

>  创建维度时，需要注意的是，维度分为记录维度和非记录维度，非记录维度是固定大小的维度，而记录维度时不知道大小的维度，比如时间维，通常是不知道要写入多少个时刻数据的，而空间维度，通常是可以固定大小。

### 定义新的变量
使用如下函数定义新变量：

* `nc_def_var`
* `nf_def_var`
* `nf90_def_var`

定义新变量时通常会添加变量的属性信息，变量的属性有整型，浮点型和字符串等类型，针对不同类型的属性，需要使用特定的函数添加属性：

* `nc_put_att_int`，`nf_put_att_int`，`nf90_put_att_int`
* `nc_put_att_text`，`nf_put_att_text`，`nf90_put_att_text`

### 结束维度和变量定义

当维度和变量定义完成之后，需要使用定义结束函数结束定义阶段：

* `nc_enddef`
* `nf_enddef`
* `nf90_enddef`

### 写入数据

写入数据时，需要根据所写的变量类型选择相应的函数，比如变量是标量，向量，矩阵还是数组，又或者是字符串。

### 关闭文件对象

当上述步骤已经完成，不需要再添加任何信息时，需要使用如下函数关闭打开的文件对象：

* `nc_close`
* `nf_close`
* `nf90_close`

> ⚠️：在创建新文件时，如果定义的维度有记录维度，那么要确保记录维度位于最左侧(⚠️：这里所说的最左侧是nc文件中变量的最左侧，但是在程序中定义变量的时候，使用`nc_def_var`等定义变量时，记录维度应该位于最右侧)，否则会出现 `NetCDF: NC_UNLIMITED in the wrong index` 的错误。

> 定义变量的维度顺序与添加数据时变量的维度信息大小要一致，否则可能会出现`NetCDF: Start+count exceeds dimension bound`的错误。

## 字符和数字转换

字符和数字间的转换，可以使用`write`语句，将变量看作**内部文件**。比如:

```fortran
character(len=4) :: stryear
character(len=2) :: strmonth
integer :: year, month

write(stryear, "(I4.4)") year
write(strmonth, "(I2.2)") month 
```

对于不足两位或者四位的，用0补齐。

> 进行数字和字符转换时，要注意定义的字符串的长度和转换数字为字符后的长度是否一致，如果长度不一致可能会出错:  “output statement overflows record”

## 参考链接
1. https://www.unidata.ucar.edu/software/netcdf/docs/modules.html
2. https://climate-cms.org/2018/10/12/create-netcdf.html
3. [http://people.sc.fsu.edu/~jburkardt%20/f_src/netcdf/netcdf.html](http://people.sc.fsu.edu/~jburkardt /f_src/netcdf/netcdf.html)
4. https://www.unidata.ucar.edu/software/netcdf/fortran/docs/
5. http://www.met.reading.ac.uk/~ross/Computing/netCDF.html
6. https://stackoverflow.com/questions/29463305/fortran-output-statement-overflows-record-error-when-naming-output-files

## 更新记录
2019.05.13 添加函数概览和创建新文件部分
2019.05.14 添加文件读取
2019.05.18 添加字符串和数字转换


