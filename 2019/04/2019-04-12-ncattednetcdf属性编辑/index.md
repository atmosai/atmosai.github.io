# ncatted:NetCDF属性编辑


  `ncatted`可以编辑netCDF文件的属性。`ncatted`可以添加，创建，删除，更改和重写属性。`ncatted`可以将每个操作应用到文件中的每个变量。`ncatted`主要用于写属性。如果仅是读取属性信息，
可以使用`ncks -m -M`。

  因为`ncatted`会重复使用，所以可能会增加`history`全局属性的大小，可以使用`-h`选项放置将操作记录自动添加到输出文件的`history`全局属性。

  当使用`ncatted`改变`_FillValue`属性时，会改变相应的缺失值。如果文件内部缺失值为浮点数形式，比如1.0e36，在不同的机器之间，可能会产生不兼容的缺失值。这使得`ncatted`从不同的机器改
变文件中的缺失值时能够将文件联系起来而不会损失信息。更过信息见[Missing Values](http://nco.sourceforge.net/nco.html#Missing-Values)。

  为了使用`ncatted`，必须要理解描述属性变化的结构的含义。*att_dsc*是`-a`选项的参数，每个*att_dsc*都包含了5个元素，即`att_dsc = att_nm, var_nm, mode, att_type, att_val`

  * `att_nm`：属性名，从V4.5.1开始，接受正则表达式。
  * `var_nm`：变量名，也接受正则表达式，`global`和`group`也有各自的含义。
  * `mode`：编辑模式缩略名
    * `a`：添加，如果不存在，则创建
    * `c`：创建，如果不存在，则创建，否则，不改变已存在的属性
    * `d`：删除，如果不存在，则不执行删除操作。如果`att_nm`留空，和指定变量相关的所有属性都将被删除。当选择了删除模式时，`att_type`和`att_val`将是多余的，可以留空。
    * `m`：更改，如果有，则更改，没有，则不执行更改操作。
    * `n`：如果存在，则添加到已有属性值之后，否则，不指定操作。
    * `o`：重写，默认模式。
  * `att_type`：属性类型缩略名
    * `f`：浮点数，即netCDF的`NC_FLOAT`类型
    * `d`：双精度浮点数，`NC_DOUBLE`
    * `i, l`：整数，`NC_INT`
    * `s`：短整型，`NC_SHORT`
    * `c`：字符，`NC_CHAR`
    * `b`：字节，`NC_BYTE`
    * `ub`：无符号字节，`NC_UBYTE`
    * `us`：无符号短整型，`NC_USHORT`
    * `u, ui, ul`：无符号整型，`NC_UINT`
    * `ll, int64`：整型，`NC_INT64`
    * `ull, uint64`：无符号整型，`NC_UINT64`
    * `sng, string`：字符串，`NC_STRING`
  * `att_val`：属性值

### 示例

  ```bash
  ncatted -a isotope,'^H2O*',c,s,'18' in.nc # 为变量名中包含H2O的添加 isotope 属性
  ncatted -a '.?_iso19115$','^H2O*',d,, in.nc  # 删除变量中包含H2O的属性值以_iso19115结尾的属性
  # Overwrite units attribute of specific 'lon' variable
  ncatted -O -a units,/g1/lon,o,c,'degrees_west' in_grp.nc
  # Overwrite units attribute of all 'lon' variables
  ncatted -O -a units,lon,o,c,'degrees_west' in_grp.nc
  # Delete units attribute of all 'lon' variables
  ncatted -O -a units,lon,d,, in_grp.nc
  # Overwrite units attribute with new type for specific 'lon' variable
  ncatted -O -a units,/g1/lon,o,sng,'degrees_west' in_grp.nc
  # Add new_att attribute to all variables
  ncatted -O -a new_att,,c,sng,'new variable attribute' in_grp.nc
  # Add new_grp_att group attribute to all groups
  ncatted -O -a new_grp_att,group,c,sng,'new group attribute' in_grp.nc
  # Add new_grp_att group attribute to single group
  ncatted -O -a g1_grp_att,g1,c,sng,'new group attribute' in_grp.nc
  # Add new_glb_att global attribute to root group
  ncatted -O -a new_glb_att,global,c,sng,'new global attribute' in_grp.nc
  ```



* 删除指定全局属性

  ```bash
  ncatted -a global_att,global,d,, filename
  ```

* 从一个文件拷贝全局属性

  ```bash
  ncks -A -x one_file another_file
  ```

* 重命名全局属性

  ```bash
  ncrename -a .global@.Convention,Conventions in.nc 
  ```

​    

### 参考链接

1. https://sourceforge.net/p/nco/discussion/9830/thread/c42c68fc/
2. https://linux.die.net/man/1/ncatted
3. http://nco.sourceforge.net/nco.html#xmp_att_glb_cpy
4. http://nco.sourceforge.net/nco.html#ncrename-netCDF-Renamer

​    

### 更新记录

* 2019.04.12 初版

* 2020.01.17 添加更多操作

