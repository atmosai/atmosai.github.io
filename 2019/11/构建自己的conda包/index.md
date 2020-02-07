# Conda包管理器:如何构建个人conda包


熟悉Python的都知道管理各种库有时候是多么痛苦的一件事！而有了conda包管理器，让库的安装和管理变得方便了不少。开发过程中，为了解决包的安装和管理问题，也方便用户安装，可以构建自己的conda包，并上传到Anaconda Cloud。



### 所需文件

* `meta.yaml` ：包含与需要创建的库相关的信息，比如：库的名称、版本、依赖、Github或者源码链接等。
* `build.sh`：OSX和Linux系统上用于安装库的脚本
* `bld.bat`：Windows上用于安装库的脚本



### 操作流程

此处以`pycwr`为例，`pycwr`的Github链接为：https://github.com/YvZheng/pycwr.git



{{% admonition note "Note" %}}

注意：在开始之前，可先执行以下命令，关闭conda包的自动上传功能

```bash
conda config --set anaconda_upload yes
```

{{% /admonition %}}

​    

1) 执行以下命令，将`pycwr`的源码下载到本地

```bash
git clone https://github.com/YvZheng/pycwr.git
```

2) 进入需要创建的库的目录中，创建`meta.yaml`、`build.sh`和`bld.bat`文件

首先创建`meta.yaml`文件，示例内容如下：

```yaml
package:
    name: pycwr
    version: 0.2

build:
    skip: True # [py<35]

source:
    git_url: https://github.com/YvZheng/pycwr.git

requirements:
    build:
        - python
    run:
        - python
        - 'numpy <=1.15.0'
        - 'scipy >=1.1.0'
        - 'matplotlib >=2.2.3'
        - 'netcdf4 >=1.5.3'
        - 'cartopy >=0.17*'
        - 'pandas >=0.23.4'
        - 'easydict >=1.9'
        - 'pyproj >=1.9.6'
        - 'xarray >=0.13*'
        - 'pyqt >=5.10*'
        
about:
    home: https://github.com/YvZheng/pycwr
    license: MIT
```

* `package`：给出了此包的名称和版本号
* `build`：`skip` 参数表示是否跳过一些指定条件，`#`号后的`[py<35]`表示跳过python版本低于3.5的版本，也可以跳过指定平台，比如win、linux、osx等。
* `source`：给出了包源码的链接，此处是github地址
* `requirements`：要构建的包的依赖，此部分依赖可通过github源中的`requirements.txt` 文件或 `setup.py` 获取
* `about`：给出了包遵循的协议以及说明文档等信息



然后创建`build.sh` 和 `bld.bat` 文件，`build.sh` 内容如下：

```bash
#!/bin/bash

$PYTHON $SRC_DIR/setup.py install --single-version-externally-managed --record=record.txt
```



{{% admonition note "Note" %}}

注意：可使用`bash -x -e`  运行 `build.sh` 脚本，`-x` 表示每执行一个命令就 `echo` 输出，`-e` 表示出现错误就退出。

{{% /admonition  %}}



{{% admonition note "Note" %}}

注意：`build.sh` 脚本里的 `$SRC_DIR` 环境变量表示 `conda build` 构建时存储`pycwr`源码的目录，如果不给定此变量的话，可能会出现 `setup.py not found` 的错误，从而导致无法顺利构建conda包。

{{% /admonition  %}}

​    

`bld.bat`的内容如下：

```bat
"%PYTHON%" setup.py install --single-version-externally-managed --record=record.txt
if errorlevel 1 exit 1
```

​    

{{% admonition note "Note" %}}

注意：在`bld.bat` 脚本中，最好在每一行命令之后都加上`if errorlevel 1 exit 1`，更有利于监控构建过程。

{{% /admonition  %}}

​    

{{% admonition note "Note" %}}

注意：`bld.bat` 脚本中没有需要特别注意的，出问题的是 git，如果给定的源码链接是 Github的源码链接，可能会出现 `Error: git is not installed in root environment.` 的错误。需要在Windows上安装git，或者取消`source`字段信息。

{{% /admonition  %}}



3) 上述文件创建完成后，在`pycwr` 目录下执行以下命令构建包

```bash
conda build .
```



如果没有出现错误则表示构建成功，最后会给出如下信息：

```
# Automatic uploading is disabled
# If you want to upload package(s) to anaconda.org later, type:

anaconda upload /usr/local/tools/miniconda3/conda-bld/osx-64/pycwr-0.2-py37_0.tar.bz2

# To have conda build upload to anaconda.org automatically, use
# $ conda config --set anaconda_upload yes

anaconda_upload is not set.  Not uploading wheels: []
####################################################################################
Resource usage summary:

Total time: 0:02:32.4
CPU usage: sys=0:00:00.9, user=0:00:02.5
Maximum memory usage observed: 156.4M
Total disk usage observed (not including envs): 254.1K


####################################################################################
Source and build intermediates have been left in /usr/local/tools/miniconda3/conda-bld.
There are currently 2 accumulated.
To remove them, you can run the ```conda build purge``` command
```



4) 构建成功后即可上传到自己的Anaconda Cloud账号中

```bash
anaconda login  # 执行此命令，然后按照提示输出用户名和密码
anaconda upload /usr/local/tools/miniconda3/conda-bld/osx-64/pycwr-0.2-py37_0.tar.bz2
```

等待上传完成即可。至此，所有的步骤就都已经完成了。



除了构建包之外，也可以直接安装到本地，使用如下命令可以直接在本地安装：

```bash
conda install --use-local pycwr # 在 pycwr 源码上层目录执行此命令
```



### 后续

#### 全平台构建

上面呢就完成了，单个平台的conda包创建，如果想要创建多个平台，那么就需要重复上述操作，而且还要找不同的系统。想想都头大！不过conda提供了直接转换的操作，可以省去很多时间。使用如下命令即可转换到所有平台：

```bash
conda convert --platform all /usr/local/tools/anaconda3/conda-bld/osx-64/pycwr-0.2-py37_0.tar.bz2 -o allplatform/
```

`allplatform` 目录下不仅包含了`linux-32/64`、`win-32/64`、`osx-64`，还有其他`linux`平台构建好的源码包。



如果使用Github源码包存在问题，那么可以使用`PyPi` 源码链接。从而可以避免因为git导致的构建问题。

使用如下信息

```yaml
url: https://files.pythonhosted.org/packages/f8/5c/f60e9d8a1e77005f664b76ff8aeaee5bc05d0a91798afd7f53fc998dbc47/pycwr-0.2-py37.tar.gz
sha256: 5b94b49521f6456670fdb30cd82a4eca9412788a93fa6dd6df72c94d5a8ff2d7
```

代替`meta.yaml`中的git相关信息即可：

```yaml
git_url: https://github.com/Yvzheng/pycwr.git
```

​    

{{% admonition note "Note" %}}

注意：`url` 和 `sha256` 可从`pycwr` 的 `PyPi` 页面获取。

{{% /admonition  %}}

​    

{{% admonition note "Tips" %}}

如果想要节省一些时间，可以设置自动上传

```bash
conda config --set anaconda_upload yes
```

{{% /admonition %}}



#### 删除已上传包

如果想要删除之前已经上传的指定版本包，可使用如下命令：

```bash
anaconda remove USERNAME/PACKAGENAME/0.2
```

{{% admonition note "Note" %}}

`USERNAME` 表示你Anaconda cloud的用户名

`PACKAGENAME` 表示要删除的包名称

`0.2`：表示版本号

{{% /admonition %}}



去除版本号可以删除指定包的所有版本：

```bash
anaconda remove USERNAME/PACKAGENAME
```

​    

#### 复制别人的包

从某个通道拷贝已经创建的包到自己的Anaconda Cloud账户下：

```bash
anaconda copy conda-forge/glueviz/0.10.4 --to-owner bugsuse
```

表示从 `conda-forge` 通道拷贝 `0.10.4` 版本的 `glueviz` 到 `bugsuse` 的Anaconda cloud通道。



​    

### 参考链接

1. https://docs.anaconda.com/anaconda-cloud/user-guide/tasks/work-with-packages/#uploading-packages



​    

### 更新记录

2019-11-30：更新通过 `scratch` 构建 `conda` 包

2019-12-01：添加删除和复制操作

​    




