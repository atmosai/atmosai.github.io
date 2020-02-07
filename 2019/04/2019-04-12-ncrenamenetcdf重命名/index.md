# ncrename:NetCDF重命名


  `ncrename`可以重命名netCDF维度，变量，属性和组。每个对象都有一个旧名称和新名称。所有的新名称必须是独一无二的。每一个旧名称必须存在于输入文件中，除非旧名称之前以`.`开头。
在重命名之前，不会检查旧名称是否存在。因此，如果旧名称没有以`.`开头，当旧名称不存在时，`ncrename`将终止重命名。
`.`表示旧名称存在与否是可选的，如果存在就替换，不存在就跳过。新名称不应以`.`开头。

  {{% admonition warning "Warning" %}}

  从2007年至今，netCDF库(v4.0.0–4.6.2)具有bug，导致无法正确对netCDF4文件中的坐标变量，维度和组重命名。netCDF3文件的重命名没有问题，如果netCDF4重命名频繁出现问题，可转换为netCDF3>，重命名后再转换为netCDF4。
  {{% /admonition %}}

  * `-a`：重命名属性

    重命名单个变量的属性

    ```bash
    ncrename -a u@long_name,largo_nombre in.nc
    ```

    重命名全局属性

    ```bash
    ncrename -a global@Convention,Conventions in.nc
    ```

  * `-d`：维度重命名

    ```bash
    ncrename -d lon,longitude in.nc
    ```

  * `-g`：重命名组

    ```bash
    ncrename -g g8,g20 in_grp.nc
    ```
  * `-v`：重命名变量

    ```bash
    ncrename -v p,pressure -v .t,temperature in.nc
    ```


