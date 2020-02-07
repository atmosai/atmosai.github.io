# Cpython加密python代码



人生苦短，我用Python。虽然python上手快，开发快，但是python的代码天生就能被人很容易看懂，如果碰到想要保密的部分就显得很尴尬了。



这部分介绍使用Cpython加密python代码，先直接给出代码，以后再补充详细内容：

以下代码为 `setup.py` 文件代码，在创建 `.c`  和 ` .so` 文件时，在linux中默认采用的是`gcc`编译器，如果想要指定编译器，可以设置 `CC` 环境变量，但是只改变 `CC` 环境变量的话在编译时采用的是 `icc`，但是在链接时使用的仍然是 `gcc`。因此还需要设置 `LDSHARED` 环境变量。

```python
 import os
 from distutils.core import setup
 from Cython.Build import cythonize

 os.environ['CC']='icc'
 os.environ['LDSHARED']='icc -shared'
 os.environ['CXX']='icpc'
 os.environ['FC']='ifort'

 setup(name='test',
       ext_modules=cythonize('*.py'),
       )
```



`cythonize('*.py')` 表示将当前目录下所有 `.py` 脚本编译链接为 `.so` 文件，从而对python代码进行加密。



然后在命令行中执行以下语句编译链接对应的`.so`文件

```bash
python setup.py build_ext --inplace
```



更详细的内容以后再补充...

​    

### 参考链接

1. https://stackoverflow.com/questions/37904377/can-cython-be-compiled-with-icc

​    

### 更新

2019-12-26 初次更新






