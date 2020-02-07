# 基于f90tohtml创建模式源码调用图


气象数值模式中涉及到的源代码非常庞大，而且各种模块非常的多，虽然体系非常清晰，并对不同类别的模块进行了分类，但是仍然无法清晰的知道每一个模块之间的调用流程。为了熟悉模式的调用流程，方便向模式中添加内容或者出现问题的时候更容易检查问题，可以创建模式的调用流程图，从而熟悉模式模块之间的调用。

以WRF模式为例，用 f90tohtml 创建模式调用流程图。

### 安装 f90tohtml

先下载 [f90tohtml](https://code.google.com/archive/p/f90tohtml/downloads) 然后解压到指定文件夹：

    tar -zxvf f90tohtml.tar.gz
    
然后进入 f90tohtml 文件夹中，更改 f90tohtml 中的 perl 的路径，如果路径和系统中的perl所在路径一致则无需更改(用 which perl 查看系统中 perl 的路径)。

    cd f90tohtml
    vi f90tohtml
    
然后更改 f90tohtmll 中的 $path_f90tohtml 为解压后的 f90tohtml 所在的路径。

执行以下命令更改 f90tohtml 的权限

    chmod u+x f90tohtml

添加 f90tohtml 路径到 .bashrc 中

    vi ～/.bashrc
    export PATH=$PATH:/f90tohtmlpath/
    
进入 f90tohtml 目录下的 examples 目录，然后编辑 d2ps_prepare.pl 和 crm_prepare.pl ，改变其中的 $the_path 为f90tohtml/examples/d2ps 所在路径。修改 d2ps.f2h 中的 $dir_html，这是用来指定要创建的  d2psbrowser 路径。注意：不要自行创建此目录。   

执行以下命令，然后生成一些测试文件

     perl d2ps_prepare.pl

然后执行以下命令

     f90tohtml d2ps.f2h

以上都是安装以及测试操作，下面开始生成WRF模式的源码调用流程图。

### 创建WRF调用流程图

执行以下命令进入 nwp_codes 目录，然后修改 wrf_prepare.pl 中 $the_path 为WRF源码路径  ： /path/of/wrf/../WRFV3/

执行以下命令生成 .ls文件

    perl wrf_prepare.pl

修改 wrf.f2h  中的 $dir_html 为转换后的输出目录

**注意：修改 $dir_html 时，文件夹应该是不存在的，否则生成文件时会出错。**

然后执行以下命令生成WRF模式调用流程图

    f90tohtml wrf.f2h

文中以 WRFV3.9.1 为示例，创建了WRF模式的调用流程图，在主页 [model](http://www.i-lightning.cn/model/) 标签下可查看生成后的效果。

生成后的页面提供了正则表达式的搜索，可以更方便的进行检索操作。


