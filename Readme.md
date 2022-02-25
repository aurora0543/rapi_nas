# 拯救吃灰的树莓派

——树莓派搭建NAS以及智能家居系统（基于Nextcloud+Homeassistant)保姆级教程

​	首先声明一下，我的大学专业不是计算机，web层面的很多东西都是我自己查文档慢慢摸索的，可能比较业余，讲得不妥当的地方还请各位大佬帮忙纠正，谢谢！

​	我的树莓派4B已经购买了两年多的时间了，但是由于树莓派的性能比较拉跨等种种原因，使得树莓派除了用来学Linux开发再无别的用处，所以大多数人买的树莓派难逃吃灰的命运。但是考虑的现在树莓派溢价如此严重，我觉得不让它发挥点价值心里也有点过意不去，所以考虑了好久如何让压榨树莓派的剩余价值。

​	我最终考虑到的解决方案是用树莓派搭建一套NAS服务器，同时可以作为家庭的影音系统使用。在这同时，还可以给树莓派安装Homeassistant，结合米家等平台实现智能家具管理系统。我整理我完整的搭建过程，在这里供大家参考。

1. 从零开始，第一步，准备工作

   > * 下载系统
   >
   > ​	树莓派有很多系统可供大家选择，虽然官方主推使用官方的Raspbain系统，不过在实际使用过程中，Raspbain的对各种web组件的支持并不友好，而且网络上的参考方案少的可怜，除此之外Raspbain对arm64的支持相当差劲，所以权衡之下我选择使用Ubuntu系统。
   >
   > ​	首先从ubuntu官网对应的[Ubuntu Server arm64](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.4&architecture=server-arm64+raspi) 20.4版本镜像文件（我们整个配置过程用不到图形界面，而且图形界面树莓派带起来很吃力，所以使用server版本就足够了）。
   >
   > * 准备需要的软件
   >   1. [SD card formatter](https://www.sdcard.org/downloads/formatter/) : 用于格式化SD卡
   >   2. [Raspberry Pi Imager](https://www.raspberrypi.com/software/) : 用于向SD卡烧录系统镜像
   >   3. [DiskGenius](https://www.diskgenius.cn/download.php) : 用于SD卡烧录完成后扩容SD分区大小
   >   4. [Final shell](http://www.hostbuf.com/t/988.html) : SSH工具，是我用过对新手最友好的ssh软件了，如果你喜欢使用其他的可以自己按需选择
   > * 准备需要的硬件
   >   1. SD卡以及读卡器 ：按需选择，如果不差这点钱的话建议SD卡直接一步到位买个64G的，如果要做web服务器16G的SD卡真心不够用。
   >   2. 树莓派4B以及电源、千兆网线 ：树莓派4B虽然官方建议使用5V 3A供电，但是树莓派真的没有那么矫情，找一个手机快抽头就可以了。至于为什么用网线，因为我们既然要做nas，肯定要把速度拉满，Wi-Fi的速度肯定是不够的。
   >   3. 一块硬盘以及硬盘盒 ：用于作为数据的存储盘，机械和固态自行选择。如果选择机械硬盘，硬盘盒最好选择带电源的那种，因为树莓派有可能供电不足。（如果直接用闲置的移动硬盘那这些钱全省了！)

2. 正式开始，安装及配置系统

   > * 制作系统SD卡
   >
   >   1. 插入SD卡使用SD card formatter把SD卡格式化。
   >
   >      ​	打开SD card formatter后，选择对应SD卡，然后点击Format等待格式化完成即可
   >
   >      <img src="images\2-1.png" alt="图2-1" style="zoom:50%;" />
   >
   >   2. 烧录系统镜像文件
   >
   >      ​	把下载好的系统文件解压，得到一个img文件。打开Raspberry Pi Imager，点击CHOOSE OS，在打开的选项卡选择最下面的Use custom选项，然后选择刚才解压好的img文件。点击CHOOSE STORAGE，选择刚格式化的SD卡。然后点击ERITE，等待完成烧写工作。完成后，拔掉SD卡，插入树莓派中，准备下一步操作。
   >
   >      <img src="images\2-2.png" alt="图2-2" style="zoom:40%;" />
   >
   >      3. 扩容系统分区
   >
   >         ​	再重新把SD卡插回到电脑上，打开DiskGenius，左侧栏选择SD卡对应的设备，完成对EXT4分区的扩容。至此系统SD卡制作完成。
   >
   >         <img src="images\2-3.png" alt="图2-3" style="zoom:80%;" />
   >
   > * 系统配置
   >
   >   4. 完成安装
   >
   >      ​	插入硬盘盒usb口到树莓派的usb3.0接口，插入网口到路由器，树莓派插入SD卡后上电开机。等待两分钟让树莓派完成启动，然后登录路由器的管理页面查看树莓派对应的局域网ip地址。然后在电脑上打开Final shell，完成ssh连接的创建并连接。
   >
   >      默认用户名：ubuntu
   >      默认密码：ubuntu
   > <img src="images\2-4.png" alt="图2-4" style="zoom:50%;" />
   >
   >   5. 换国内软件源
   >
   >      ​	ubuntu arm架构要使用ubuntu-ports源，国内源中只有清华源是支持最好的，这里我们选择清华源，如果会自己配置的建议浏览[官方帮助文档](https://mirror.tuna.tsinghua.edu.cn/help/ubuntu-ports/)。
   >
   >      ``` bash
   >      # 打开软件源配置文件
   >      sudo nano /etc/apt/sources.list
   >      # 把原有内容注释掉，嫌麻烦的可以直接删掉（不会出事的。。。）
   >      # 添加以下内容：
   >      ```
   >
   >      ``` bash
   >      # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
   >      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
   >      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
   >      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
   >      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
   >      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
   >      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
   >      deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
   >      # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
   >      ```
   >
   >      ```bash
   >      # ctrl+o保存	 ctrl+x退出
   >      # 更新软件
   >      sudo apt update && sudo apt upgrade -y
   >      # 这一步可能会报连接错误，这是因为树莓派无法访问https网址导致的，把替换的国内源中https换成http再执行一遍更新操作就好了
   >      ```
   >
   >   6. 设置系统语言为中文
   >
   >      ​	还是中文界面看着舒服呀，这里教大家怎么把树莓派的系统默认语言设置成中文。
   >
   >      ``` bash
   >      # 安装中文支持包
   >      sudo apt install language-pack-zh-hans -y
   >      # 设置区域
   >      sudo dpkg-reconfigure locales
   >      ```
   >
   >      ​	在打开的界面，使用方向键翻页和选择，找到zh_CN.UTF-8 UTF-8选项，使用空格键选择，然后使用Tab按键移动光标到OK上点击回车。在下一个界面，继续使用Tab选择到zh_CN.UTF-8 UTF-8回车确定（如果有两个随便选一个就可以）
   >
   >      <img src="images\2-7.png" style="zoom:38%;" />
   >
   >      <img src="images\2-7-1.png" alt="2-7-1" style="zoom:38%;" />
   >
   >   7. 调整swap大小
   >
   >      ​	swap，即交换分区，在内存不够用的情况下开辟出一块虚拟内存用于缓存数据，由于我们之后需要编译安装很多组件，所以必须要使用大量swap空间，而ubuntu默认开辟的100M的swap空间显然不够用，我们需要增加swap空间。
   >
   >      ```bash
   >      # 新建一个存放swap文件的目录，如果提示已经存在不用理会
   >      sudo mkdir /swap 
   >      cd /swap
   >      # 创建swap文件，此过程耗时较长，耐心等待
   >      # count=2000000表示swap大小为2G，你可以按实际需求更改
   >      sudo dd if=/dev/zero of=swapfile bs=1024 count=2000000
   >      # 设置正确权限，可以省略，但是以后会报警告
   >      sudo chmod 600 /swapfile
   >      # 启动swap文件
   >      sudo swapon /swapfile
   >      ```
   >
   >      ​	这样swap分区创建完毕了，但是这是一次性的，重启一次swap分区就消失了，所以我们要设置swap分区的开机自动挂载。
   >
   >      ``` bash
   >      # 文本编辑器可以自行选择，新手推荐使用nano，使用方法一看就会
   >      sudo nano /etc/fstab
   >      # 键入一下文本，有强迫症的可以自行用Tab键对齐。。。。
   >      /swapfile swap swap defaults 0 0
   >      # ctrl+o保存	 ctrl+x退出
   >      free -m
   >      #查看交换分区是否启动了
   >      ```
   >
   >   8. 下载系统必备软件
   >
   >      ​	这一步根据自己喜好下载。

3. 服务器安装

   > * 安装宝塔面板
   >
   >   1. 运行安装脚本，直接运行ubuntu系统宝塔面板安装脚本即可。
   >
   >   ```bash
   >   wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
   >   # 出现确认界面输入'y'即可
   >   ```
   >
   >   2. 安装完成后按照提示登录宝塔面板。
   >
   > * 面板配置
   >
   >   3. web服务安装
   >
   >      ​	进入应用商店，选择运行环境分类。分别安装：Nginx、MySQL、PHP以及phpMyAdmin。（版本可以根据实际情况选择，不过我使用过在树莓派上最不容易出bug的版本组合是Nginx 1.20、MySQL MariaDB 10.5、PHP 8.0、phpMyAdmin选最高版本即可）
   >
   >      <img src="images\3-3.png" alt="图3-3" style="zoom:38%;" />
   >
   >      ​	因为是编译安装，树莓派本身性能就拉跨，这个过程可能需要好几个小时的时间（三四个小时起步），而且还有极端情况下可能会死机（swap配置过小导致）。所以建议大家在晚上睡前进行安装，这样第二天起床就可以用了！！！
   >
   >      ​	服务器基本服务的安装到这里就结束了！

4. Nextcloud服务器搭建

   > * 文件准备
   >
   >   1. 下载服务器源码
   >
   >      ​	去官网下载服务[器版本的源码](https://nextcloud.com/install/#instructions-server)。打开网址后选择Archive File选项卡，点击Download下载。
   >
   > * 部署nextcloud
   >
   >   2. 宝塔面板新建网站
   >
   >      ​	在宝塔面板网站选项卡下新建站点，并配置好数据库和php。（数据库的编码类型一定要选择utf8mb4，不然可能正常无法显示emoji表情。。。当然如果你没有这个需要的话可以直接用utf8）
   >
   >      <img src="images\4-2.png" alt="图4-2" style="zoom: 50%;" />
   >
   >   3. 上传源码到服务器
   >
   >      ​	进入文件选项卡，进入到网站的根目录（默认在/www/wwwroot/nextcloud），上传源码压缩包，解压，把解压生成的nextcloud文件夹里的文件拷贝到网站根目录下，删除压缩包和解压生成的文件夹。
   >
   >   4. 设置
   >
   >      ​	浏览器输入树莓派局域网ip，网页可以正常访问证明服务器配置正确，下面进行nextcloud的设置。
   >
   >      ​	

5. 速度优化

   > * 安装PHP必要的扩展服务
   > * 配置缓存
   > * 设置伪静态
   > * 更改数据库安全等级
   > * 设置PHP FMP的PATH路径
   > * 解除文件区块大小限制
   > * 调整nginx参数
   > * 调整php参数
   > * 配置定时任务

6. 配置家庭影音系统

   > * 配置SMB服务
   > * 配置DLNA服务
   > * 配置苹果Air Play音箱

7. 配置离线下载器

   > * 安装aria2
   > * 配置aria2 web前端

8. 安装Docker平台

   > * 使用脚本安装Docker
   > * 配置Portainer容器

9. 安装Homeassistant

   > * 使用脚本安装Homeassistant+Supervisor
   > * 解决国内无法联网问题
   > * 配置Homeassistant与米家等平台的互联

10. 系统的一些个性化设置

    > * 终端美化
    >
    >   1. 安装zsh
    >   2. 安装oh-my-zsh
    >   3. 安装插件
    >   4. 安装主题
    >
    > * 编辑SSH连接提示语
    >
    > * 关于固定树莓派的ip地址
    >
    >   1.方法一，路由器配置
    >
    >   2.方法而，树莓派配置
    >
    > * 尽请期待......
