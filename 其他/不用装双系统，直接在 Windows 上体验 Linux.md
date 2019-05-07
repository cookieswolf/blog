# 不用装双系统，直接在 Windows 上体验 Linux

「Microsoft Loves Linux！」

说出这句话的不是所谓的 IT 领域那些技术专家或者是意见领袖，而是时任微软 CEO 的萨蒂亚 · 纳德拉，在 2015 年的一次活动中，这位第三任微软 CEO 脱口而出的这句话，让这个曾经开源界最大敌人的微软，正式拥抱这个开源世界最大的操作系统：Linux。

![](https://cdn.sspai.com/2018/03/25/f289d70d25e33a8e41b2700a62fd03cb.jpg)

其实在云计算领域，微软很早之前就让其 Azure 支持多个流行的 Linux 发行版，但对于普通消费者而言，真正的变化发生在后面的 Windows 10：微软宣布将会在 Windows 10 内置 Linux，而采用的技术上并非是所谓的「虚拟化」技术——也就是说，这个子系统的 Linux 完全是原生运行在 Windows 10 上的。

![](https://cdn.sspai.com/2018/03/25/6a7cca29f55e4d5ee4d2de7747a3bab5.png)WSL 技术实现原理

而微软给这个 Linux 系统命名为：Windows Subsystem for Linux，而对于有些系统极客而言，这个名字实在太熟悉了，因为在 Windows 7 之前，微软也曾经内置过一个 UNIX 子系统，可以原生运行 UNIX 二进制程序，他的名字叫做：Windows Services for UNIX。

即便如此，对于很多普通用户而言，Windows Subsystem for Linux 也只是尝鲜的玩物罢了，但对于不少软件开发、系统极客而言，无需通过虚拟机以及双系统的形式体验 Linux ，并且可以实现系统级别的文件互操作，实在是太具有吸引力了。而今天我们就一起来体验探索一番。

## 如何启动 Linux 子系统

微软从 Windows 10 周年更新（build 14393）开始内置 Windows Subsystem for Linux 组件框架，只不过这项功能当时还只能称作是 Beta 版，而在 Windows 秋季创意者更新中，安装 Linux 子系统变的更为简单——可以直接通过 Microsoft Store 来下载子系统，而可选择的发行版也从最初的只有 Ubuntu 变成可以选择 Suse、Ubuntu、Debian、甚至是用来进行网络安全工作 Kail Linux。

![](https://cdn.sspai.com/2018/03/25/bc6f2007cf2b6b6d33a8d5e68431522f.png)目前 Microsoft Store 有多款 Linux 发行版可供选择

只不过如果你想要体验这些发行版还需要进行一些简单操作，毕竟 Windows Subsystem for Linux 组件框架并非是默认选中的。

首先我们需要确认自己的 Windows 10 版本，以下的操作方法只适用与 Windows 10 秋季创意者更新（Windows 10 build 16299）以上版本，如果你是 Windows 10 周年更新，安装 Linux 子系统的安装办法你可以检索「 Bash on Windows」自行探索安装方法。此外，系统必须是 64 位操作系统。

![](https://cdn.sspai.com/2018/03/25/c9a281638af54268938fd6a0b7b505d7.png)开启「适用于 Linux 的 Windows 子系统」

以上均确认后，打开 「控制面板」—> 「程序和功能」，在左边的「启用和关闭 Windows 功能」里面勾选「适用于 Linux 的 Windows 子系统」，然后点击确定（这一步有可能需要重启）。

![](https://cdn.sspai.com/2018/03/25/f700748a6460dfd13dd02d1249f21733.png)

接着打开 Microsoft Store，搜索喜欢的 Linux 发行版，这里我选择的是我比较熟悉的 Linux 发行版 Ubuntu，然后点击安装。对于初学者来说，Ubuntu/ Debian 系的发行版具有非常完善的包管理系统，方便新手快速上手。

![](https://cdn.sspai.com/2018/03/25/d4e3151daf4d5931584c5d32e80442e4.png)

安装完毕之后，你就可以在 Windows 开始菜单中找到「Ubuntu」这个应用了！换言之，现在你的 Windows 10 中就已经成功安装发行版为 Ubuntu 的 Linux 子系统。

## Ubuntu 子系统设置与基本命令

在开始菜单中打开 Ubuntu 后，Ubuntu 会进行较长时间的安装和初始化，之后会提示你设置 Linux 的用户名和密码，需要注意的是这个用户名和密码和 Windows 并不通用。

![](https://cdn.sspai.com/2018/03/25/1c9aabe7cd1a1ec4e723754ca310fcc0.png)

设置密码是非明文的，不会像 Windows 那样使用「*** 」替代，所以你只要盲打点击确认即可，建议密码使用复杂密码，有些发行版会有强制要求。

输入完成之后，系统会提示你如何提权操作，之后会自动以刚才新设置的用户名登录 Ubuntu。

我安装 Linux 第一件事就是查看内核版本以及系统系统版本，在 Ubuntu 下直接输入以下命令来查看内核版本号：

`uname -r`

这时系统会显示：4.4.0-43-Microsoft，这表示 Linux 内核版本为：4.4.0-43。

至于系统版本号，可以使用：`sudo lsb_release -a` 来查看，系统会输出：

![](https://cdn.sspai.com/2018/03/25/630d0d1b11254249aa73a088c91102e6.png)

这表示 Ubuntu 版本为 16.04。为 Ubuntu 的长期支持版本。

## 更换 Linux 子系统的软件源并更新软件

之前我说过使用 Ubuntu /debian 系最大的好处就是可以使用「软件源」进行软件安装，使用 Ubuntu 自带的 deb 包管理系统安装软件可以减少直接下载源码编译的麻烦，所以这里就要用到「apt-get」系列命令了。

因为默认的软件源是 Ubuntu 的官方源，我们可以选择替换为国内的软件源，比如说阿里云镜像的软件源。

在当前命令行下面输入：

`sudo-i`

提权后输入密码，使用 root 权限登录。然后接下来备份当前源，输入以下命令：

`cp /etc/apt/sources.list /etc/apt/sources.list.old`

不难看出管理源的文件就是 sources.list，我们选择编辑它，编辑器我这里选用的是 vim，所以命令是：

`vim /etc/apt/sources.list`

使用 vim 后会进入命令模式，敲键盘上的 「i」键键入编辑模式，然后复制下面这段代码（拷贝代码，然后在编辑器上鼠标右击就可以复制）：

<pre>  # deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
  deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
  deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
  deb http://mirrors.aliyun.com/ubuntu/ xenial universe
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
  deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
  deb http://archive.canonical.com/ubuntu xenial partner
  deb-src http://archive.canonical.com/ubuntu xenial partner
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
</pre>

完成之后再敲键盘上的「esc」退出编辑模式，然后再输入`:wq`点击保存并退出编辑器 vim。

![](https://cdn.sspai.com/2018/03/25/7f30887123a20b7913f0985a3cc424f5.png)编辑软件源

紧接着我们更新软件源让编辑的文件生效：

`apt-get update`

这里我们就将 Ubuntu 的软件源切换到阿里云的源了。

之后再输入：`apt-get upgrade` 对当前系统的软件和类库进行来更新。如果不出意外系统会自动对现有的软件包进行更新，经过这一系列的操作，目前 Ubuntu 的软件以及类库都是最新的，而系统版本也升级到 Ubuntu 16.04.4 LTS。

## 启用 SSH 并使用 SSH 客户端登录

虽说通过 App 或者应用的形式在 Windows 10 上体验 Linux 是一个不赖的选择，但对于很多软件开发的朋友而言，使用 Windows 内置的 CMD 或者 PowerShell 来操作 Linux 依旧有着很多不习惯。而最为关键的是当需要对文件进行操作时，使用交互命令远不如使用 SFTP 来的更为「简单粗暴」。因此只要通过配置 SSH 远程登录，就可以像管理远程服务器那样来操作这个 Linux 系统了。

首先，因为 Ubuntu 系统限制，所以我们需要可以为 root 用户设置新密码，这里输入：

`sudo passwd root`

配置好之后，未来使用 SSH 客户端或者 SFTP 客户端登录系统时，我们就可以直接使用 root 权限进行登录，就不用使用之前的 `sudo -i` 提权操作了。

其次按照常规，我们使用`cp` 命令将 SSH 相关配置文件进行备份：

`sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`

之后使用 vim 编辑器编辑 「sshd_config」文件：

`sudo vim /etc/ssh/sshd_config`

键盘上点击 「i」后进入编辑模式，编辑并调整以下设置项：

<pre>  
  Port 8022（因为 Windows 10 的 SSH 端口已经默认被占用，所以我换成了一个新的端口）
  （去掉前面的 #）ListenAddress 0.0.0.0
  UsePrivilegeSeparation no（原来是 yes 改成 no）
  PermitRootLogin yes(修改成 yes)
  (在前面加上 #)StrictModes yes
  PasswordAuthentication yes（原来是 no，改成 yes）
</pre>

之后点击 「Esc」退出编辑模式，直接输入 `:wq` 退出并保存。

![](https://cdn.sspai.com/2018/03/25/b7a954479ed5adb2dda354d7d5230fff.png)编辑配置文件并启动 SSH

然后输入命令：`service ssh start` 启动 SSH。

如何验证已经可以访问呢？我们首先打开 SSH 客户端，比如我目前使用 Xshell，选择「新建会话」。

之后在新建的会话设置框的「连接」中添加如下内容：

<pre>  名称：WSL（这个随便填）
  协议：SSH
  主机：127.0.0.1（本机环回接口）
  端口号：8022
</pre>

之后在「用户身份验证」中输入验证方法，方法选择 「Password」，然后在输入用户名：root，密码选择刚才新设置的 root 密码，最后点击确定。

![](https://cdn.sspai.com/2018/03/25/60863c9f3881482f3aa3f942b6b98dda.png)

然后在左侧的会话管理器找到刚才设置的新会话，双击后如果显示如下图所示的界面就算是成功了！

![](https://cdn.sspai.com/2018/03/25/0ca3ef403a0a54deb341d96534d7c7af.png)

除了使用 Xshell 这种 SSH 客户端进行服务器操作之外，还可以使用 Xftp 进行文件上传和管理，唯一的区别是在新建会话处，协议选择「SFTP」，端口号和之前 Xshell 使用的端口号一致即可，点击确认之后出现类似 FTP 管理的界面就算是成功了！这样你就可以使用更为直观的工具来访问 WSL 系统的文件目录。新建文件上传文件也变得更为简单。

![](https://cdn.sspai.com/2018/03/25/e3368f985de8f04598f50c895d744758.png)

## 开启图形化界面

比起 Windows 和 macOS，Linux 很多时候给普通用户都是冰冷的命令行形象，这让很多 Linux 初学者望而却步；但实际上 Linux 是可以使用我们所说的 GUI 图形化界面的，只不过图形化界面并没有默认安装，这里我尝试手动安装一个图形化桌面。

![](https://cdn.sspai.com/2018/03/25/2282d0b7f233ebcbf065219f7200b5e1.png)

由于属于 Linux 子系统的限制，因此安装一些比较「重」的图形化界面组件会大量消耗系统资源，因此我选择较为轻量级的图形化桌面组件：MATE，也是 Ubuntu MATE 的默认桌面组件，当然另一个轻量级桌面 xfce 体验也不错。

首先在终端中输入以下命令安装 Mate 桌面：

`sudo apt-get install mate-desktop-environment`

这一步命令就是安装完整的 MATE 桌面，这个过程相当长，因为 WSL 默认没有桌面环境，对应的相关组件也没有安装，所以安装桌面会将相关的组件以及依赖都一并安装。

![](https://cdn.sspai.com/2018/03/25/670cf7d73efda57f92743ce19a8580ac.png)安装图形化界面以及 VNC 服务端

紧接着我们需要安装可以访问图形化界面的软件，这里使用图形化远程访问工具：VNC；你可以理解成 Windows 电脑中的远程访问。当然 VNC 服务端 WSL 也是不会默认安装的，所以需要输入以下命令安装：

`sudo apt-get install vnc4server`

安装完毕之后需要修改 VNC 的默认启动桌面，这时候输入：

`sed -i 's$x-window-manager$mate-session$' ~/.vnc/xstartup`

将默认启动桌面改成 Mate 桌面启动，然后输入：`vncserver` 启动服务端（第一次启动需要设置连接密码）。这里 WSL 端就基本设置完毕了。

之后我们需要在 PC 上安装 VNC 的客户端，我这里选择的是 Realvnc，然后直接选择 [Chrome 应用版本](https://chrome.google.com/webstore/detail/vnc%C2%AE-viewer-for-google-ch/iabmpiboiopbgfabjmgeedhcmjenhbla?utm_source=chrome-app-launcher-info-dialog)，在 Chrome 商店中添加为 Chrome 独立应用。

![](https://cdn.sspai.com/2018/03/25/33cb68a1bcf5b9414a2bec638e853a2e.png)

打开 realvnc 并在地址栏中输入：127.0.0.1:1 ，点连接并输入连接密码，如果不出意外，你就可以看到安装有 mate 桌面的 Ubuntu 界面了！

![](https://cdn.sspai.com/2018/03/25/8119fb7c520f69de813425365f2c7b70.png)

可视化桌面的终端里面，你可以输入 `sudo apt-get install firefox` 来安装 Firefox 浏览器，不一会儿你可以在左上角菜单栏的「Applications」中的「Internet」中找到 Firefox 浏览器啦！好了，接下来还能做什么就自己去探索吧！

![](https://cdn.sspai.com/2018/03/25/99b6cf4e4195671d4f19edd187a55e91.png)图形化的 Linux 界面

## 一起动手做：搭建本地静态网站

经过以上的折腾，其实你应该对 WSL 有了比较清楚的认识了，其实对于很多开发者而言，WSL 最大的好处在于更接近项目生产环境，虽说 Windows 本身有 IIS 网页服务器可供选择。但目前大部分网站服务器系统都采用的是 Linux，而网页服务器也多是使用 Apache，所以在 WSL 在本机完成部署调试后可能会接近实际一些。所以这里我们做一个小实践：将开发好的一个静态网站部署到 WSL 里面并可以直接访问。

首先，我们要确保 WSL 中安装有 Apache 网页服务器，所以尝试安装（使用超级用户权限），在终端中输入：

`apt-get install apache2`

![](https://cdn.sspai.com/2018/03/25/8045557987f35eeb544e29005fd13fa3.png)

安装完毕之后在终端中输入以下命令开启 Apache 网页服务器：

`service apache2 start`

当终端里面显示诸如「 * Starting Apache httpd web server apache2」之后，打开本机的网页浏览器，访问：[http://127.0.0.1](http://127.0.0.1) ，当显示以下页面就表示 Apache 网页服务器已经生效！

![](https://cdn.sspai.com/2018/03/25/996e0eb85b23651d116f650d2d112485.png)

接下来我们尝试将自己开发的静态网页项目传到对应的目录中，这里我们打开 Xftp 这个远程文件工具，连接到 WSL 这个站点，然后访问 /var/www/html 这个目录，然后将项目文件夹传到该目录下方。

![](https://cdn.sspai.com/2018/03/25/1afe77305167f6f47080d81e438ce6f9.png)

例如我现在传过去的网页全景项目名为「xuyi」，那么传好后我打开浏览器，访问：[http://127.0.0.1/xuyi](http://127.0.0.1/xuyi) 就可以看到做好的网页的效果啦！如果你是使用 chrome 访问的话，Wappalyzer 扩展还可以显示出当前网站项目使用的框架等。

![](https://cdn.sspai.com/2018/03/25/762471744aa908a2c7b98190c4953565.png)

## 结语

至此，我们已经较为完整的体验了 Windows Subsystem for Linux 的一些基础玩法，其实在我看来，Windows 10 下的 Linux 子系统更多的是补充原本 Windows 10 在开发领域上的一些不足，让软件开发 / 网络开发人员可以以较低的成本来实现与生成环境的一致性，也不用再为了开发而安装双系统甚至虚拟机了。当然在本次体验中我并没有更深入的探索，比如说在 WSL 中安装 PHP 环境以及 Mysql 数据库，所以如果你对 Linux 感兴趣，想要在 Windows 10 上探索 Linux，系统原生支持的 WSL 不妨一试。

> 下载 [少数派 iOS 客户端](http://sspai.com/s/nqQk)、关注 [少数派公众号](http://sspai.com/s/KEPQ)，读有趣的内容 🎉


原文地址:https://sspai.com/post/43813