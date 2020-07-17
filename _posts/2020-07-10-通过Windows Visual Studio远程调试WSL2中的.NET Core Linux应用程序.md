最近两天在Linux中调试.NET Core应用程序，同时我发现在Linux中调试.NET Core应用程序并不容易。一直习惯在Visual Studio中进行编码和调试。现在我想的是可以简单快速的测试.NET Core应用在Linux。所以通过本篇文章我们能了解到如何在Windows中使用Visual Studio进行远程调试我们部署在Linux中的应用程序，从而我们可以去发现或者说去调试在中会产生的一些问题。

# Windows中的Linux：Hello WSL

子系统从这里我不做过多的介绍了，大家有兴趣的话可以从 [https://docs.microsoft.com/en-us/windows/wsl/about](https://docs.microsoft.com/en-us/windows/wsl/about) 中了解一下。

第一步从windows开启wsl

![](https://imgkr.cn-bj.ufileos.com/34ef9501-8690-4d1c-9c48-8f0ab3083f5c.png)

我们可以搜到他并打开

![](https://imgkr.cn-bj.ufileos.com/09fe1297-6247-49ae-ae81-385a2ff04a99.png)

打开后我们可以看到如下内容

![](https://imgkr.cn-bj.ufileos.com/b09581ea-befa-4734-9f45-da76b262d6a0.png)

因为一会我需要对他进行调试所以我这边选择的是DEBUG

![](https://imgkr.cn-bj.ufileos.com/14481c50-2cee-4f2f-8339-98b4982ffa48.png)

通过上面一波操作后我们需要做的是在WSL提示符下，输入dotnet并加上我们的应用程序集名称

![](https://imgkr.cn-bj.ufileos.com/6e4923f9-8ed2-4aeb-bbbe-6065451503ed.png)

现在我们已经将我们的应用程序发布到了linux中如下所示

![](https://imgkr.cn-bj.ufileos.com/cb745520-8acb-4f1b-9b9f-9ead687b790d.png)


# 如何附加到正在运行的Linux应用程序

正如上面所述，我想要做的是在Visual Studio中调试Linux应用程序，那么下面我们来看一下附加

![](https://imgkr.cn-bj.ufileos.com/53819263-b019-4307-90cb-8a362c6c7c52.png)

SSH连接类型将与具有以下通信架构的WSL一起使用：

![](https://imgkr.cn-bj.ufileos.com/b07aa9fe-f49c-4cf3-a20e-6b8fbdfc5c25.png)

我们需要安装vsdbg调试器，然后通过SSH通到将命令发送到Linux调试器。

1. 默认情况下，SSH服务器与WSL一起安装。但是，我无法使整个管道都可以使用，因此必须卸载并重新安装它：

```
sudo apt-get remove openssh-server

sudo apt-get install openssh-server
```

2. 更改SSH配置，以允许 Visual Studio所需的用户名/密码类型的安全性，如果不知道如何有效地使用vi来简单地编辑文件，请安装nano

```
sudo apt-get install nano
```

3. 在/etc/ssh/sshd_config中，更改PasswordAuthentication设置

```
sudo nano /etc/ssh/sshd_config

PasswordAuthentication yes
```

4. 重启SSH服务器

```
sudo service ssh start
```

5. 安装解压缩才能获取vsdbg

```
sudo apt-get install unzip

curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v latest -l ~/vsdbg
```

现在我们可以选择SSH作为连接类型，同时需要点击“刷新”按钮将这些信息填充，如下所示：

![](https://imgkr.cn-bj.ufileos.com/db8da056-56d3-4d33-b3cf-e414a47f1713.png)

单击“刷新”按钮后，底部的列表应包含在WSL中运行的Linux进程。

![](https://imgkr.cn-bj.ufileos.com/b6d337ae-4d3b-46a6-96d6-6f4746862804.png)

选择.NET Core应用程序，然后单击附加选择托管调试器:

![](https://imgkr.cn-bj.ufileos.com/2e1b36ae-520e-4ce1-af4a-76cdc67d52ba.png)

当我们在代码中设置断点之后，并且触发我们设置的断点就会达到如下效果:

![](https://imgkr.cn-bj.ufileos.com/92465e2a-4932-417f-a5fb-6f226806a64d.png)

通过上面内容来说我们以达到了我们预期的想法，我们可以通过Visual Studio借助WSL进行调试Linux应用程序。当然对于这一块我也在寻找更便捷的方式，当然我还发现一个 [	
.NET Core Debugging with WSL 2](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.Dot-Net-Core-Debugging-With-Wsl2&ssr=false#overview).


# Reference

https://www.hanselman.com/blog/RemoteDebuggingANETCoreLinuxAppInWSL2FromVisualStudioOnWindows.aspx

https://devblogs.microsoft.com/devops/debugging-net-core-on-unix-over-ssh/

https://medium.com/criteo-labs/wsl-visual-studio-attaching-launching-a-linux-net-core-application-on-my-window-10-ab21c179702d

https://github.com/Microsoft/MIEngine/wiki/Offroad-Debugging-of-.NET-Core-on-Linux---OSX-from-Visual-Studio
