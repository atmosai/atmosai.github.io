# GFS数据多线程下载


近期由于NCEP GFS数据源网络通信协议由[HTTP](https://zh.wikipedia.org/zh-hans/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)更新为[HTTPS](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)，基于axel的多线程下载工具均出现了 Too many  redirects的问题。这是由于旧版axel无法支持新的HTTPS协议，导致在获取链接时出现了重定向的问题。

{{% admonition note "Note" %}}

上述问题仅出现在基于HTTP协议的GFS数据源，基于ftp的GFS数据源不受影响。

{{% /admonition %}}

```bash
axel -n 4 https://nomads.ncep.noaa.gov/pub/data/nccf/com/gfs/prod/gfs.2019021100/gfs.t00z.pgrb2.0p25.f000
```

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g080vt3v0rj31wm03wwf7.jpg)

通过更新axel到最新版本即可解决上述问题。关于axel的发布见[axel github](https://github.com/axel-download-accelerator/axel)。从axel [发布页](https://github.com/axel-download-accelerator/axel/releases)下载最新版本axel-2.16.1，比如axel-2.16.1.tar.gz 下载完成后，解压：

```bash
tar -zxvf axel.2.16.1.tar.gz
```

编译安装

```bash
cd axel.2.16.1.tar.gz

./configure --prefix=/directory/to/install
make 
make install
```

其中 `configure` 配置命令中 `--prefix` 选项指定安装目录，如果不指定安装目录，会安装到系统默认目录。

`make` 命令会在当前文件夹内进行编译，到此步就已经完成了axel的编译，编译后的axel就在当前目录的某个文件夹内，

`make install` 命令会将编译完成后的文件移动到 `--prefix` 指定的路径。**如果没有指定安装目录，并且没有root权限，**执行`make`之后，无需执行 `make install`。

编译安装结束之后，将新编译的axel路径添加的环境变量中，然后source更新环境变量，查看axel版本

```bash
axel --version
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g080z51q5mj30so0h8wgt.jpg)

安装完成后，可以测试一下：

```bash
axel -n 4 https://nomads.ncep.noaa.gov/pub/data/nccf/com/gfs/prod/gfs.2019021100/gfs.t00z.pgrb2.0p25.f000
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0810j24koj31xe0juae6.jpg)

当然还是可以使用之前的HTTP协议链接，axel会自动处理这种问题。


