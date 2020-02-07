# ncrcat:NetCDF记录合并


  合并一些列输入文件的记录变量。默认情况下，最终的记录维度长度是输入文件所有记录维度长度之和。`ncrcat`可以从标准输入接受大量文件。

  输入文件的大小可以是多变的，但是每个文件必须要有一个记录维度。记录坐标应该是单调的。

  `ncrcat`无法解包数据，只能简单的从输入文件拷贝数据和元数据到输出文件。这意味着对于所有输入文件的给定变量而言，使用打包规则压缩的数据必须使用相同的打包参数(即`scale_factor`和`add_ffset`，否则连接后数据集无法正确解包。

  对于输入文件打包参数(`packing parameters`)不同的情况，通常按照如下方式处理：

  * 使用`ncpdq`解包数据
  * 使用`ncrcat`连接解包数据
  * 使用`ncpdq`重新打包结果

### 示例

  * 合并多个文件

    ```bash
    ncrcat in1.nc in2.nc in3.nc in4.nc out.nc
    # 或
    ncrcat in[1-4].nc out.nc
    ```

  * 选择性合并文件

    假设85.nc，86.nc，87.nc每个文件`time`维度包含12个记录，想获取1985年12月到1986年2月的数据：

    ```bash
    ncrcat -d time,11,13 85.nc 86.nc 87.nc 8512_8602.nc
    ncrcat -F -d time,12,14 85.nc 86.nc 87.nc 8512_8602.nc    # 索引按照Fortran索引形式
    ```

    如果仅想获取某个月份的数据，比如只想获取这三年3月份的温度数据
    ```bash
    ncrcat -F -d time,3,,12 -v temperature 85.nc 86.nc 87.nc 858687_03.nc
    ```
    

