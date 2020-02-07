# Mac安装QGIS


想要使用ArcGIS做一些东西，但是无奈ArcGIS没有对应的Mac版本，又不想安装虚拟机在Windows上运行，最后找到了开源的QGIS，使用了QGIS 3.4.1，没想到安装过程就碰到了许多坑。

此文纯粹记录Mac安装QGIS 3.4.1的过程。

从QGIS官网下载Mac安装包，然后安装时，第一个问题：

* **提示 QGIS required python3.6**

  这个问题比较棘手，因为QGIS 3.4.1只支持Python3.6，而且还不认Anaconda版本的Python，似乎是只认clang编译的Python3.6，只能重新安装一个版本了，使用下列命令先去除系统中python的链接，然后安装Python3.6

  ```bash
  brew unlink python
  brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/f2a764ef944b1080be64bd88dca9a1d80130c558/Formula/python.rb
  ```

  安装完成之后，记下安装的Python具体版本号，然后要将安装的Python链接到指定目录。这里我安装的具体版本号是 3.6.5_1，然后执行一下命令(具体安装路径取决于你的系统实际安装路径)：

  ```bash
  ln -s /usr/local/Cellar/python/3.6.5_1/Frameworks/Python.framework/Versions/3.6 /Library/Frameworks/Python.framework/Versions/
  ```

解决了上述问题之后，就可以顺利安装QGIS了。安装完成之后打开QGIS，如果中间没有出现任何问题，那么恭喜你，后面的内容你就不用看了。

我在安装完成之后打开QGIS时，弹窗出现以下问题，找不到对应的库：

* **Couldn't load plugin 'processing'          ModuleNotFoundError: No module named 'osgeo'**

  这个问题也是比较棘手的，这里不做过多解释，osgeo是地学的一个常用库，但是这里并不是说直接用pip安装osgeo就行了，而是要**安装gdal**，安装gdal的时候出现了找不到 cpl* 头文件的问题，这里需要先安装mac上的gdal，而不是直接安装python的 gdal，应为python的 gdal需要一些头文件。

  我出现的问题是，即使安装了mac的gdal，也把头文件所在的路径添加到环境变量中，但是依然提示找不到头文件，最后发现Mac安装好的GDAL路径中有预编译的 python gdal，直接使用以下命令安装即可：

  ```
  /usr/local/Cellar/python/3.6.5_1/bin/pip3 install /Library/Frameworks/GDAL.framework/Versions/Current/Resources/GDAL-2.3.2-cp36-none-macosx_10_9_x86_64.whl
  ```

本以为这就结束了，没想到后面又出现了很多库找不到(心塞==)。所幸后面缺失的库安装都比较顺利，只需要根据提示，缺少什么库使用pip安装即可。

```
Couldn't load plugin 'processing' 
ModuleNotFoundError: No module named 'yaml' 
```

```
Couldn't load plugin 'processing' 
ModuleNotFoundError: No module named 'owslib' 
```

执行以下命令安装即可：

```bash
/usr/local/Cellar/python/3.6.5_1/bin/pip3 install pyyaml
/usr/local/Cellar/python/3.6.5_1/bin/pip3 install owslib
```

```
ModuleNotFoundError: No module named 'pygments' 
```

如果安装库时出现类似以下库的问题，只需要指定 --user 选项即可：

```bash
Could not install packages due to an EnvironmentError: [Errno 13] Permission denied: '/usr/local/bin/pygmentize'
Consider using the `--user` option or check the permissions.
```

```bash
/usr/local/Cellar/python/3.6.5_1/bin/pip3 install --user pygments
```

完成上述所有库缺失的问题之后，再打开基本就没问题了。

至此，安装完成。


