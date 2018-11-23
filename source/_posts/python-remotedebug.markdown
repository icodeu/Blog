title: Python远程调试及frp内网穿透
date: 2018-11-22 22:00:00
tags: [Python, Pycharm, frp]
----

讲述利用Pycharm进行远程调试，及内网环境下如何打通内网和外网。

<!--more-->

# 先说Python远程调试

### 依赖pycharm-debug.egg

需要pycharm-debug.egg库，位置在`应用程序->右键PyCharm->显示包内容->Content/debug-eggs/pycharm-debug.egg`(还有一个pycharm-debug-py3k.egg，是用于python3.x版本的)。注意一定要用当前Pycharm包中的，否则有可能会出现版本不一致的问题。将其拷贝到项目目录，并加入到依赖，具体代码如下：

```
import sys
import os

WORK_SPACE = os.path.split(os.path.abspath(__file__))[0]
sys.path.append(WORK_SPACE + "/pycharm-debug.egg")
```

### 使用pycharm-debug.egg

只需要两行代码，具体如下：

```
import pydevd
pydevd.settrace('your-local-ip', port=your-local-port, stdoutToServer=True, stderrToServer=True)
```

注意ip和port都需要填写本机的，因为pycharm debug的原理是:`本机监听远端，远端主动连接本机`。

刚才的settrace是为了远端在执行这个代码时，主动连接your-local-ip:your-local-port，即只完成了`远程主动连接本机`，现在还需要配置一下`本机监听远端`，点击Edit Configurations...，，左侧 + -> Python Remote Debug，名字随意写，主要是 Local host name 配置成 your-local-ip，port 配置成 your-local-port(此时会发现上面提示的内容就是刚才的settrace)，点击OK保存即可。之后切换到remotedebug，点击Debug按钮，本机即可在your-local-ip监听your-local-port端口，如图
![python](http://qiniu.icodeyou.com/python-remotedebug-0.png)

![python](http://qiniu.icodeyou.com/python-remotedebug-1.png)


### 上传代码至远端并debug

先看一下最终的代码
![python](http://qiniu.icodeyou.com/python-remotedebug-2.png)

将代码上传到远端，直接 python Main.py 执行，远端会根据settrace主动连接本机，如果没问题的话，本机就可以进入debug了(记得先在本机打断点)。

需要注意一点，可能会弹出这个窗口，意味着虽然连接上了，但是远端和本地的文件没有映射上，解决办法是点击 Auto-detect
![python](http://qiniu.icodeyou.com/python-remotedebug-3.png)



# 再说内网穿透

如果本机和远端都是在一个网段内，即都是公网或都是局域网，那么上述方法就可以完成调试了；但如果远端是公网，本机是局域网，就要稍微麻烦点了，原因是公网机器无法直接连接上局域网机器(试想10.x.x.x的公网是肯定连不上192.x.x.x的)。

若想公网主动连接局域网，可以使用内网穿透技术。内网穿透依赖NAT的相关知识，关于NAT直接看图
![python](http://qiniu.icodeyou.com/python-remotedebug-4.png)

内网穿透原理
![python](http://qiniu.icodeyou.com/python-remotedebug-5.png)


结论如图所示：

- 公网不能直接访问内网ip

- 内网可以直接访问公网ip，因为有路由器的NAT转换，且连接一旦建立便可双向通信

- 内网可以先向"中间代理公网""注册"自己的ip和端口，由"中间代理公网"和内网ip直接通信，做端口转发

那么为了实现内网穿透，需要两个条件：

- 一台公网可访问的主机，假设公网ip为10.0.0.2

- 内网穿透软件，以frp为例

frp的地址见[github](https://github.com/fatedier/frp/releases)，在其release页面下载对应的版本。对于Linux它提供了很多版本，可以执行`lsb_release -a`查看机器具体版本，下载即可。

### frp的配置

看文档的话可以直接略过这一节。

#### frps服务端

即公网机器的配置，`vi frps.ini`

```
[common]
bind_port = 8018
```

代表公网的frps程序开放8018端口与内网的frpc通信，执行 `./frps -c ./frps.ini &` 启动frps，等待内网机器连接。
![python](http://qiniu.icodeyou.com/python-remotedebug-7.png)


#### frpc客户端

即内网机器的配置，`vi frpc.ini`

```
[common]
server_addr = 10.0.0.2
server_port = 8018

[ssh]
type = tcp
local_ip = 192.0.0.1
local_port = 22
remote_port = 6000
```

客户端将访问 10.0.0.2:8018 与公网 frps 建立连接，并告诉公网 frps: `所有访问你6000的都转发到我的22号端口来`，具体流程可参考上面内网穿透原理图。执行 `./frpc -c ./frpc.ini &` 启动frpc，即可完成内网穿透。
![python](http://qiniu.icodeyou.com/python-remotedebug-8.png)

那么至此所完成的功能是：`另一台公网主机10.0.0.1访问10.0.0.2:6000端口即可与内网192.0.0.1:22端口建立双向通信`。

内网穿透搞定了，接下来回到python远程调试，还是对于上面ip和端口的例子

- 假定本机是192.0.0.1

- 内网穿透"中间代理"是公网10.0.0.2

- 远程执行代码的是公网10.0.0.1


那么在本机的remote configuration中，Local host name应配置为192.0.0.1，Port配置为22。

那么在远程的10.0.0.1中，代码应为

```
pydevd.settrace('10.0.0.2', port=6000, stdoutToServer=True, stderrToServer=True)
```

这样，远端代码在settrace访问10.0.0.2:6000时，便可以与192.0.0.1:22建立双向通信。

至此，内网和公网环境下的python远程调试得以解决。

上述例子中有两台公网机器，分别为10.0.0.1和10.0.0.2，真的需要两台才可以完成内网穿透吗？


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)