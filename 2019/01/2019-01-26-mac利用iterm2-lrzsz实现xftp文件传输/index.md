# Mac利用iTerm2+lrzsz实现文件快速传输


无论是在windows还是unix系统，在远程服务器和本地之间传输文件时，总是还需要手动去打开xftp或者其他ftp工具来进行文件传输，每次都这样操作就很麻烦。如果没有安装类似工具，只能通过scp或者sftp等命令行工具，而每次都可能需要输出一长串命令。为了节省时间，可以利用lrzsz工具直接进行命令行传输。

### 安装

lrzsz的安装极为方便，在unix平台可以直接通过系统的包管理器进行安装，比如centos可以通过如下命令进行安装:

```bash
   yum install lrzsz 
```

Mac也可以通过安装Homebrewer等包管理器，然后通过这些包管理器进行安装。安装命令如下：

```bash
   brew install lrzsz 
```

在Mac上安装完成之后，基于iTerm2使用`sz`和`rz`进行文件传输时，还需要进行一些配置。需要设置Zmodem协议，安装链接见这里[iTerm2-zmodem](https://github.com/mmastrac/iterm2-zmodem)。

* 下载 `iterm2-send-zmodem.sh` 和 `iterm2-recv-zmodem.sh` 放到 `/usr/local/bin` ，然后赋予两个文件可执行权限 `chmod a+x iterm2*zmodem.sh`
<br>

* 打开 iTerm2，选择`Preferences`，选择`Profiles`，选择`Advanced`，选择`Triggers`部分`Edit`按钮，然后选择添加新trigger，具体操作流程可见[How to Create a Trigger](https://www.iterm2.com/documentation-triggers.html)
<br>
  

  <div align=center><img src="https://github.com/bugsuse/blogpic/blob/master/2019/01/26/fig1.jpg?raw=true" width="100%" height="100%"></div>


* 将如下内容添加到相应的选项

```bash
    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
    Instant: checked

    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
    Instant: checked
```

操作如下图所示

<div align=center><img src="https://github.com/bugsuse/blogpic/blob/master/2019/01/26/fig2.jpg?raw=true" width="100%" height="100%"></div>


<br>
### 操作示例

从远程服务器传输文件到本地，在远程服务器终端执行以下命令：

```bash
sz filename
```

![](https://github.com/bugsuse/blogpic/blob/master/2019/01/26/fig3.jpg?raw=true)


<br>
从本地上传文件到远程服务器，直接在远程服务器终端执行以下命令，即打开图形化界面，选择需要传输的文件即可：

```bash
rz
```

![](https://github.com/bugsuse/blogpic/blob/master/2019/01/26/fig4.jpg?raw=true)


<br>
以上为简单的传输示例，`rz` 和 `sz` 命令还支持一系列选项，更多操作待发掘。
<br>

