---
layout: post
title: "在 Mac 下安装 mysql for python"
date: 2015-04-10 19:49:33 +0800
comments: true
tags: [Python, Mysql]
---

在 Mac 中使用 XAMPP 环境下，如何来为 python 安装 mysql

<!--more-->

停下手头的，写点东西。

关于如何能在 Mac 下为 python 安装 mysql，真是折腾了一会。之前在 windows 下装过，很顺利，这次到 Mac 下居然这么坎坷了。特来记录一下过程，共享学习。

我的环境：Mac Yosetime 10.10.2 + python 2.7 + XAMPP

凡是在搜索“python mysqldb 安装 mac”等类关键字时，得到的教程基本都是一致的，教程如下，我开始也是照着这个来的但是不成功，原因是我用的是 XAMPP（下文具体讲）。

在下面的网址下载mysqldb模块：
```
http://sourceforge.net/projects/mysql-python/
```

在mac os x直接双击解压,命令行进入解压后的目录， 执行
```
python setup.py build
```

如果有
```
sh: mysql_config: command not found
```

提示，我们需要编辑下mysql的路径，打开setup_posix.py

找到:
```
mysql_config.path = "mysql_config"
```

改为：
```
mysql_config.path = "/usr/local/mysql/bin/mysql_config"
```

再执行一次 build：
```
python setup.py build
```

build 没问题了，然后执行：
```
sudo python setup.py install
```

安装成功后，在命令行输入 python 进入 python 环境，如果报下面的错误：
```
ImportError: dlopen(/Library/Python/2.7/site-packages/MySQL_python-1.2.4b4-py2.7-macosx-10.8-intel.egg/_mysql.so, 2): Library not loaded: libmysqlclient.18.dylib
  Referenced from: /Library/Python/2.7/site-packages/MySQL_python-1.2.4b4-py2.7-macosx-10.8-intel.egg/_mysql.so
  Reason: image not found
```

解决方法，建立一个软链就可以了
```
sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib /usr/lib/libmysqlclient.18.dylib
```

Over~如果你的 mysql 是手动安装的话，也许上面的教程就没有问题了，但关键是在于，我在 `python setup.py build` 的时候总是告诉我 mysql_config 命令找不到。根本原因是我这里的 mysql 是在 xampp 中的，所以 `mysql_config` 命令是在 `/Applications/XAMPP/xamppfiles/bin/mysql_config` 这里的，找到原因了，再打开 setup_posix.py

找到:
```
mysql_config.path = "mysql_config"
```

这次改为：
```
mysql_config.path = "/Applications/XAMPP/xamppfiles/bin/mysql_config"
```

之后再build：
```
python setup.py build
```

发现就成功了，并且没有报任何错误，另外还有同样的一点，在设置软链接时，也需要做相同的修改：
```
sudo ln -s /Applications/XAMPP/xamppfiles/bin/libmysqlclient.18.dylib /usr/lib/libmysqlclient.18.dylib
```

其余部分按照刚才教程之后的步骤操作就可以了。

总结：本次为 python 安装 mysql 屡屡失败的根本原因是在于自己过于相信网上的教程了，并不是说教程不对，而是说自己太不过脑子了，直接照抄并不思考为什么，刚才的错误只要稍微思考一下就能知道错误原因在哪里了，也不用浪费那么多时间了。

好了，继续了。

###个人github:  [http://github.com/icodeu](http://github.com/icodeu)

###CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

###个人微信号：`qqwanghuan`  只为技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)