# 用Android手机做电脑的HTTP代理服务器
在手机端创建一个 HTTP 代理不但可以让电脑共享手机网络，还可以利用手机翻墙。

### 手机端

1.  在 Play Store 里选择 Termux 安装。  
    其它备选 app ： GNURoot Debian 等。
2.  打开 Termux 安装Python：

        $ apt install python

    Termux 默认安装的是 Python 3，自带包管理 pip 。如果没有也可以自行安装 pip 。
3.  安装 HTTP 代理脚本：

        $ pip install proxy.py

4.  运行 HTTP 代理：

        $ proxy.py

    默认端口是 8899。修改端口或其它功能可以查看相应帮助。
5.  打开 Android 手机的 USB 调试模式 （要首先打开开发者模式才能看到）。

### 电脑端

1.  现在用数据线把手机和电脑相连。
2.  在电脑端安装 Android 系统调试程序 adb 。（与操作系统有关，Linux 系统如 Ubuntu 可以直接从软件仓库安装）。
3.  在系统终端中输入：

        $ adb forward tcp:8899 tcp:8899

    将手机端口 8899 映射到电脑端口 8899 。注：每次重新连接后都需要输入该命令。
4.  修改系统或浏览器的代理服务器为 localhost:8899 （具体设置与操作系统和浏览器有关）。

现在电脑就可以使用手机网络上网了。如果你的手机安装了诸如 Shadowsocks 之类的代理服务，那么电脑也可以自动翻墙。