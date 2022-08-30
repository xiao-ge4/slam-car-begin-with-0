# slam-car-begin-with-0

## Section 1

### ubuntu 安装

#### 下载镜像

现在 Ubuntu 官网只会放最新的版本，这里可以下载 18.04 的历史版本（其他版本第二个链接），树莓派4（其他版本，看链接内容）则选择以下来中其中一种（armhf 与 arm64 的区别也有列出）

> 1. 可下载的两个版本
>    ![1661648109147](C:\Users\22627\AppData\Roaming\Typora\typora-user-images\1661648109147.png)
>    ![1661648132065](C:\Users\22627\AppData\Roaming\Typora\typora-user-images\1661648132065.png)

> 2. 下载18.04 或其他版本的链接
>
> http://cdimage.ubuntu.com/releases/18.04/release/
>
> http://cdimage.ubuntu.com/releases/

> 3. armhf 与 arm64 的区别
>
> armhf =硬件浮点指令+ 32位指令集
>
> 一些树莓派尚不支持 64 位，4B 是都支持，我选的是armhf

#### 格式化 SD 卡

使用 Ventoy 格式化 SD 卡，为烧录系统做准备

> 1. SD 卡是什么
>    [SD卡？TF卡？傻傻分不清楚？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/353227488)
> 2. Ventoy 及其使用
>    [超级好用的装机神器——Ventoy - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/137477151)

#### 系统烧录 

用 Win32DiskImager 烧录

> 选择 Imager 
>
> 选择硬盘，
>
> 点击“写入”
>
> 等待出现“成功”-->关闭窗口，结束烧录

### 树莓派接口

https://zhuanlan.zhihu.com/p/407916696

### 开机启动

连接外设（屏幕、键盘、鼠标），通电

> 1. 注意：屏幕需要在通电之前接树莓派，关于如何连接，请看《树莓派接口》
> 2. 通电后，初始 账户+密码 都是 ubuntu，输入成功后需要再设置，这个地方过于简单不会通过

### 逻辑说明

这个地方我想理一理逻辑，此时的树莓派依赖于外接设备可以开始工作

**但是**

1. 当你没有携带外接设备时，你无法使用
2. 此时，update 这样的指令（但凡需要联网），你都无法使用

**解决**

1. ssh 是一种网络安全协议，你可以通过它来控制你的树莓派
2. VNC 是虚拟网络控制台，树莓派一端做vncserver，你的电脑做vncviewer 你可以 控制+查看界面（这里的桌面也需要你安装）
   显而易见 ssh 是一个可选项，但仍然建议先配置，如果vncserver没有成功开启，vnc服务就会失败，但 ssh 仍然可以使用
3. 首先我们得联网，但是 update 会默认使用国外的地址，非常容易访问超时，在此之前我们先更换网络源，同时我们需要网络（wifi）才能联系网络

**总结**

那么我们在配置ROS 之前，我们为了上网服务、控制树莓派、查看桌面，我们按照如下顺序进行

1. 连接 wifi
2. 更换源
3. 配置 ssh 
4. 配置 vnc

### 连接wifi

这里别人总结的很好

> 1. 参考文章 --> 非常好用
>    [树莓派4B安装Ubuntu Server20.04（18.04）连接wifi（对于ubuntu server 99%适用）_小白天才的博客-CSDN博客_树莓派ubuntu连接wifi](https://blog.csdn.net/qq_30613365/article/details/120739069)
> 2. 我用的是第二个

### 换源

直接上指令

> 1. 编辑文件 #你要是熟悉其他编辑器 vi 等都可以
>
> ```shell
> sudo vim /etc/apt/sources.list
> ```
>
> 2. 点击键盘 i 进入 Insert 模式
> 3. 应该有很多个 deb 开头的一行，其他的被 # 注释掉了，删掉未被注释的 deb 下面的 #deb-src 前面的#
> 4. 将现在没被注释的每行  https://到/Ubuntu-ports之间的网址替换成mirrors.aliyun.com  换完之后长这样
>
> ```shell
> #注意：每个 sources.list 文件内容不一定完全一样，根据上面步骤更改，如下只是 样式 示例
> 
> deb http://mirrors.aliyun.com/ubuntu-ports/  bionic main restricted universe multiverse
> deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic main restricted universe multiverse
> 
> 当然，实际上有很多行
> ```
>
> 5. 然后按键 `Esc` ，输入 `:wq`退出当前编辑
> 6. 输入如下命令，更新软件，同时验证配置成功
>
> ```shell
> sudo apt-get update
> sudo apt-get upgrade
> ```

### 防火墙问题

写在前面的话

1. 如果暂时没有遇到该问题，可先跳过，在下面的几步，非常可能遇到超时问题，返回来寻求解决
2. 另：防火墙只是超时问题的一种可能原因，不是的话，Google / Bing 一下，也欢迎反馈问题，我们一起完善文档

问题介绍

链接超时可能是树莓派没有允许你的电脑访问它，也就是被防火墙挡住了，有以下几个解决方式

> 1. ufw 设置
>
> ```shell
> sudo apt-get install ufw #安装
> sudo ufw enable #启用
> sudo ufw status #查看防火墙状态
> sudo ufw default deny #系统启动时自动开启
> sudo ufw enable|disable #开启/关闭防火墙 
> 
> sudo ufw allow|deny [service] #开启/禁用特定端口
> 例如：
> sudo ufw allow 53 #允许外部访问53端口(tcp/udp)
> sudo ufw allow 22/tcp #允许外部访问22端口(tcp)
> sudo ufw allow from 192.168.1.100 允许此IP访问所有的本机端口(这里可以设置称自己的电脑)
> ```
>
> 2. 另：vnc 连接的话，需要输入的地址是
>
> ```shell
> 树莓派地址:端口号
> 例如 192.168.1.1:1
> ```
>
> 3. 另，很多人可能知道 Ubuntu 的防火墙，以下是二者的关系

### ssh 远程连接

上命令

> 1. 安装 ssh
>
> ```shell
> sudo apt-get install openssh-server
> sudo apt-get remove openssh-server #卸载命令
> ```
>
> 2. 启动SSH服务
>
> ```shell
> sudo /etc/init.d/ssh start
> ```
>
> 3. 查看进程，检查是否启动成功
>
> ```shell
> ps -e | grep sshd
> 
> #另：sshd 是否正常运行
> sudo systemctl status ssh
> ```
>
> 4. 配置允许 root 用户登录
>
> ```shell
> sudo vim /etc/ssh/sshd_config #编辑文本
> 
> 取消“PermitRootLogin yes”前面的 # 号
> 
> 保存退出
> ```
>
> 5. 设置开机启动（如果需要的话）
>
> ```shell
> sudo systemctl enable ssh #开启开机自启动
> sudo systemctl disable ssh #关闭开机自启动
> sudo systemctl start ssh #单次开启 ssh
> sudo systemctl stop ssh #单次关闭 ssh
> 
> #注意：每次设置自启动后 重启一次
> reboot
> ```
>
> 6. 补充：重启命令与关闭命令
>
> ```shell
> sudo /etc/init.d/ssh restart   #重启SSH服务
> sudo /etc/init.d/ssh stop      #关闭SSH服务
> ```
>
> 7. 远程连接 工具putty
>
> ```shell
> ssh name@ip_address
> ```

### 桌面安装

上命令

> 1. 安装 mate 桌面
>
> ```shell
> sudo apt-get install ubuntu-mate-desktop
> ```
>
> 2. 重启
>
> ```shell
> sudo reboot
> ```
>
> 3. 若开机等待时间长
>    原因：安装时连接了网络，系统被自动设置成连接路由器自动通过DHCP来获取ip地址完成上网。但是由于网络断开，他会有个重连过程，等待时间为5分钟
>
> ```shell
> sudo vim /etc/systemd/system/network-online.target.wants/networking.service
> 
> 找到TimeoutStartSec=5min修改为TimeoutStartSec=2sec
> ```

### VNC 设置

#### server 

> 1. 安装 vncserver
>
> ```shell
> sudo apt-get -y install vnc4server
> ```
>
> 2. 此时，首次启动`vncserver`，需要设置密码，根据提示设置，目的是先打开一次服务，生成一个`~/.vnc/xstartup`文件
> 3. 查看桌面环境
>
> ```shell
> echo $DESKTOP_SESSION  //如果显示mate就是mate桌面环境
> ```
>
> 4. 打开`~/.vnc/xstartup`文件后在末尾添加：
>
> ```shell
> unset SESSION_MANAGER
> unset DBUS_SESSION_BUS_ADDRESS
> mate-session & 
> mate-panel &
> ```
>
> 5. 重启 vnc 服务
>
> ```shell
> vncserver -kill :1  #关闭“1”desktop
> vncserver :1 #开启“1”desktop
> 
> #千万注意 “-kill”后有空格，“:”后无空格
> ```
>
> 6. 设置 vnc 服务在树莓派开机时自启动
>    在`/etc/init.d/`文件夹下新建一个服务脚本文件`vnc4server`，作用就是将`vnc4server程序`作为一个服务在开机时启动。创建完后把下面的代码粘贴上去保存（其中，注明了可以自己更改的参数）
>
> ```shell
> #!/bin/sh
> ### BEGIN INIT INFO
> # Provides: vncserver
> # Required-Start: $local_fs
> # Required-Stop: $local_fs
> # Default-Start: 2 3 4 5
> # Default-Stop: 0 1 6
> # Short-Description: Start/stop vncserver
> ### END INIT INFO
> 
> # More details see:
> # http://www.penguintutor.com/linux/vnc
> 
> ### Customize this entry
> # Set the USER variable to the name of the user to start vncserver under
> export USER='pi'
> ### End customization required
> 
> eval cd ~$USER
> 
> case "$1" in
>  start)
> 	# 启动命令行。此处自定义分辨率、控制台号码或其它参数。
> 	su $USER -c '/usr/bin/vncserver -depth 16 -geometry 1024x768 :1'
> 	echo "Starting VNC server for $USER "
> 	;;
>  stop)
> 	# 终止命令行。此处控制台号码与启动一致。
> 	su $USER -c '/usr/bin/vncserver -kill :1'
> 	echo "vncserver stopped"
> 	;;
>  *)
> 	echo "Usage: /etc/init.d/vncserver {start|stop}"
> 	exit 1
> 	;;
> esac
> exit 0
> 
> ```
>
> 执行
>
> ```shell
> sudo chmod 755 /etc/init.d/vnc4server # 修改脚本权限为可执行权限
> sudo update-rc.d vnc4server defaults # 将该脚本作为一项服务，设置为开机自动加载:
> ```
>
> 重启
>
> ```shell
> sudo reboot #已经能自启动 vnc 服务了
> 
> #想要停止的话就执行：sudo service vnc4server stop
> ```
>
> 一些其他vncserver相关指令
>
> ```shell
> 启动 
> vncserver :1
> 关闭
> vncserver -kill :1
> 查看
> vncserver -list
> 默认端口对应
> vncserver :1	5901
> vncserver :2	5902
> 依次类推
> ```
>
> 7. 参考来源
>    [树莓派ubuntu18.04 mate 安装VNC_夏小正的鲜小海的博客-CSDN博客_树莓派安装vnc](https://blog.csdn.net/chenzz444/article/details/119016938)

#### viewer

Windows 下载 Vnc Viwer

### 安装ROS

> 1. 设置下载源
>
> ```shell
> #注意：以下三个源下一个即可
> 
> #设置中科大源
> sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
> 
> #设置清华源
> sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
> 
> #设置上海交大源
> sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.sjtug.sjtu.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
> 
> #设置公钥
> sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
> ```
>
> 2. 安装 ROS
>
> ```shell
> #先更新一下
> sudo apt update
> sudo apt upgrade #没有更新可能会影响后面的安装
> 
> sudo apt install ros-melodic-desktop-full
> 包括ROS, rqt, rviz, 机器人通用库, 2D/3D simulators and 2D/3D perception
> 
> #另有如下其他版本
> sudo apt install ros-melodic-desktop
> 包括ROS, rqt, rviz, 和机器人通用库
> 
> sudo apt install ros-melodic-ros-base
> 包括ROS 包, build和通信库。没有GUI工具
> ```
>
> 3. 设置环境变量
>
> ```shell
> echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc #编辑文本
> 
> source ~/.bashrc #执行
> 
> #这个步骤是让你的 bash 终端 以后可以识别 roscore，rosrun等命令
> 
> #如果你用的是 zsh 等其他终端更改一下命令，例如
> echo "source /opt/ros/melodic/setup.zsh" >> ~/.zshrc
> source ~/.zshrc
> ```
>
> 4. 下载必要功能组件
>
> ```shell
> sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
> ```
>
> 5. rosdep init
>
> ```shell
> sudo rosdep init  #来了来了，几乎都会报错的QAQ
> ```
>
> 原因：访问外网 raw.githubusercontent.com ，很难的啦
> 解决：a. 梯子 b.解析服务其地址（访问外网时绕过去）
> 操作：
>
> ```shell
> 1.sudo chmod 777 /etc
> # 此指令为获取etc中的权限
>    2. 在etc中手动创建文件夹
> /etc/ros/rosdep/sources.list.d
>    3. 解析 raw.githubusercontent.com 的服务器地址，工具https://site.ip138.com，打开该网站，根据提示输入要解析的网址，下面就会有地址（选一个）
> 我选的是 185.199.111.133
>    4. sudo vim /etc/hosts
> 将选择的服务器地址、网址加进去，格式如下
> 185.199.111.133 raw.githubusercontent.com
> 保存、退出
>    5. sudo rosdep init 
> 
> 若有报错ERROR: default sources list file already exists: /etc/ros/rosdep/sources.list.d/20-default.list
> Please delete if you wish to re-initialize
> 原因：/etc/ros/rosdep/sources.list.d目录下已经有20-default.list
> 解决：删除它
> 碎碎念：但是根据我刚刚的步骤，应该不会有这个问题
> ```
>
> 6. rosdep update
>
> ```shell
> sudo rosdep update	#可能又又又会报错
> ```
>
> 原因：仍然是 raw.githubusercontent.com 被墙
>
> 解决：
>
> a. 梯子 
>
> b. 多次执行/手机开热点 
>
> c.需要更新的文件下载到本地
>
> 其中：a/b方法 非常的易操作，b 方法不可小觑，网络好的话，就不需要 c 方法那么复杂了
>
> 详细讲讲 c 方法：将更新的文件下载到本地
>
> 1. 先去 github (https://github.com/ros/rosdistro)将需要的包 down 下来，这里有一个小 Trick,因为 github 的服务器都在国外，访问速度较慢，当遇到这种情况时，我会把它导入 Gitee ，这样 访问速度 会快很多（授人以渔，下面有一个章节专门用来介绍如何 Github 转 Gitee）
>    附：我的仓库地址(https://gitee.com/xiaoge4/rosdistro.git)
>
> ```
> 记录存放地址
> ```
>
> 2. 修改 /etc/ros/rosdep/source.list.d/下的20-default.list，将文件中指向raw.githubusercontent.com的url地址全部修改为指向本地文件的地址
>
> ```shell
> sudo vim /etc/ros/rosdep/source.list.d/20-default.list
> 
> 更改后：
> # os-specific listings first
> yaml file://....../rosdep/osx-homebrew.yaml osx
> 
> # generic
> yaml file://....../rosdep/base.yaml
> yaml file://....../rosdep/python.yaml
> yaml file://....../rosdep/ruby.yaml
> gbpdistro file://....../releases/fuerte.yaml fuerte
> 
> # newer distributions (Groovy, Hydro, ...) must not be listed anymore, they are being fetched from the rosdistro index.yaml instead
> ```
>
> ​	其中，......就是你的存放地址，例如我放在/home/leisp下，那么我	更改后就应该是
>
> ```shell
> # os-specific listings first
> yaml file:///home/leisp/rosdistro/rosdep/osx-homebrew.yaml osx
> 
> # generic
> yaml file:///home/leisp/rosdistro/rosdep/base.yaml
> yaml file:///home/leisp/rosdistro/rosdep/python.yaml
> yaml file:///home/leisp/rosdistro/rosdep/ruby.yaml
> gbpdistro file:///home/leisp/rosdistro/releases/fuerte.yaml fuerte
> 
> # newer distributions (Groovy, Hydro, ...) must not be listed anymore, they are being fetched from the rosdistro index.yaml instead
> ```
>
> 3. 修改` /usr/lib/python2.7/dist-packages/rosdep2/sources_list.py`中的默认的url的地址
>
>    ```shell
>    修改后：
>    
>    # default file to download with 'init' command in order to bootstrap
>    # rosdep
>    DEFAULT_SOURCES_LIST_URL = 'file:///home/leisp/rosdistro/rosdep/sources.list.d/20-default.list'
>    
>    # seconds to wait before aborting download of rosdep data
>    ```
>
> 4. 修改`/usr/lib/python2.7/dist-packages/rosdep2/rep3.py`
>
>    ```shell
>    修改后：
>    
>    # location of targets file for processing gbpdistro files
>    REP3_TARGETS_URL = 'file:///home/xxx/rosdistro/releases/targets.yaml'
>    
>    # seconds to wait before aborting download of gbpdistro data
>    ```
>
> 5. 修改`/usr/lib/python2.7/dist-packages/rosdistro/__init__.py`
>
>    ```shell
>    修改后：
>    
>    # index information
>    
>    DEFAULT_INDEX_URL = 'file:///home/xxx/rosdistro/index-v4.yaml'
>    
>    def get_index_url():
>    ```
>
> 6. 最后
>
>    ```shell
>    sudo rosdep init 
>    ```
>
> 7. check --> 小海龟
>
> ```shell
> roscore 
> rosrun turtlesim turtlesim_node #启动节点
> rosrun turtlesim  turtle_teleop_key
> 
> #注意：分别在三个终端输入以上命令
> ```
>
> 8. 给自己鼓个掌吧，接下来我们要开始和硬件打交道了
>    阶段性 胜利！
>    ![1661823097704](C:\Users\22627\AppData\Roaming\Typora\typora-user-images\1661823097704.png)

### Github 转 Gitee

因为 github 的服务器都在国外，访问速度较慢，当遇到这种情况时，我会把它导入 Gitee ，这样 访问速度 会快很多

> 1. 创建仓库 
> 2. 点击“导入已有仓库”
> 3. 添加要导入项目的网址
> 4. 点击创建，仓库生成
> 5. 注意：如果想要仓库开源（所有人可见）
>    在仓库建立之后，点击管理，设置中更改信息