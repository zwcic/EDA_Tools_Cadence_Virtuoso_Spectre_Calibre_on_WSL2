# 使用wsl centos7 搭建集成电路设计开发环境的一些经验




## 一些参考文档和社区资源：

[WSL官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/)  
[Rocky/Centos for WSL](https://github.com/mishamosher)  
[wsl.conf](https://docs.microsoft.com/zh-cn/windows/wsl/wsl-config#wslconf)  
[Ubuntu 20.04 安装 Virtuoso IC 6.1.8 和 Spectre 18.1](https://zhuanlan.zhihu.com/p/370708944)

感谢OpenCad大佬的工具包
在操作过程中遇到很多issue，各路网友的解答给了我很大帮助，在此也特别感谢






## Windows Terminal配置

安装完wsl的分发版后、wt里面会自动添加配置，也可以手动配置添加、自定义主题、窗口、目录.....
网上有很多美化教程、自行查阅

 windows terminal > 设置 >  添加新配置文件     **示例：**   

* cmd
   
    C:\WINDOWS\system32\wsl.exe -d CentOS7
   
* wsl启动目录
   
    \\wsl.localhost\CentOS7\<yourpath>
   
* wsl图标
   
    ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png





## wsl配置
### 更新

    wsl --update

### [其他指令](https://docs.microsoft.com/zh-cn/windows/wsl/)

    wsl --help

### 导入导出wsl
   
    wsl --export Ubuntu-20.04 D:\wsl-Ubuntu-20.04  
    wsl --import Ubuntu-20.04 D:\wsl\Ubuntu2004 D:\wsl-Ubuntu-20.04 --version 2  
  

### 释放wsl空间

1. 打开PowerShell, 输入命令：
    
         wsl --shutdown
         diskpart

2. window Diskpart
    
         select vdisk file="C:\WSL-Distros\…\ext4.vhdx"
         attach vdisk readonly
         compact vdisk
         detach vdisk
         exit



## linux系统配置
### etc/wsl.conf

可以配置windows盘挂载、网络、开机指令、用户等  [wsl.conf](https://docs.microsoft.com/zh-cn/windows/wsl/wsl-config#wslconf)


### 固定MAC地址
   
   wsl每次开机都会重新定义MAC地址，许多许可证管理需要检查MAC地址、可以将下面代码放到/etc/profile或者/etc/bashrc等配置文件中，来固定wsl的MAC地址
   
    wantmac=00:**:**:**:**:**
    mac=$(ip link show bond0 | awk '/ether/ {print $2}') 
    if [[ $mac !=  $wantmac ]]; then
   
        sudo ip link set dev bond0 address $wantmac
   
    fi 
   当然,我更建议使用无需MAC地址的许可证

### 更新、添加epel

    yum update 
    yum install epel-release  

### 文本编辑器Gedit

    yum install gedit

### Gedit 自动高亮

sh的自动高亮

> /usr/share/gtksourceview-3.0/language-specs/  

在下面的XXX位置插入想要的自动sh高亮的文件格式

> `<property name="globs">XXX;XXX`

### 文件浏览器Thunar
1. 安装
    
        yum install thunar

2. Thunar: `open in windows terminal here`
   
    Edit > Configure custom actions  更改或添加open terminal here设置`Command`为:
   
        wt.exe -w 0 -p "Rocky8" -d %f

### 安装xterm终端模拟器，iscape运行configuring需要用到

    sudo apt install xterm

### 安装csh，启动virtuoso需要用到

    sudo apt install csh

### 安装ksh，启动virtuoso需要用到  

    sudo apt install ksh

### 安装解压工具

1. 安装Xarchiver
   
        yum install xarchiver

2. 添加到thunar exact here
   
        yum install thunar-archive-plugin
        sudo update-desktop-database
        thunar -q
    
### 安装JAVA环境

3. 先查看本地是否自带java环境：
   
        yum list installed |grep java

4. 卸载自带的java（没有请勿略）
   
        yum remove java-1.8.0-openjdk* 
        yum remove tzdata-java*

5. 查看yum仓库中的java安装包
   
        yum -y list java

6. 安装java：  
   
        yum -y install java-1.8.0-openjdk*

7. 查找Java安装路径
   
        which java  
        ls -lrt /usr/bin/java   
        ls -lrt /etc/alternatives/java  
        cd /usr/lib/jvm  
        ls

8. 配置Java环境变量
   
        export JAVA_HOME=/usr/lib/jvm/java-1.8.0
        export JRE_HOME=$JAVA_HOME/jre
        export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
        export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/toolsjar:$JRE_HOME/lib

9. 检查Java安装和配置情况
   
        java -version




## Install Cadence Virtuoso IC 6.18 and SPECTREE
### 安装Installscape
安装包里面自带一个InstallScape所以不需要额外下载
1. 解压Hotfix或Base的tar.gz包（可在windows目录下）

        tar Tar -zxvf .\..\..\
2. 运行SETUP脚本
   
        cd ..../IC06.18.260_lnx86.Hotfix/CDROM1
        sudo ./SETUP.SH
3. config  
   >Specify path of install directory [OR type [RETURN] to exit]:
   
        /opt/cadence/IC618  
   
    >Do you have InstallScape for lnx86 platform installed[y/n]?
   
        n    
   
    >Do you want to install InstallScape for lnx86 [y/n]?
   
        y  
   
   >Type the path to InstallScape installation directory [ (q to quit)]:
   
        /opt/cadence/Iscape
        #自定义

### 安装好iscape后，可直接启动

    sudo /opt/cadence/Iscape/iscape/bin/iscape.sh
### 默认安装路径设置
可以在`preferences > InstallScape > Directories`中设置`Default Install Directory`和`Default download Directory`

    /opt/cadence
### 安装IC618
选择``Local directory/Media install``

    ..../IC06.18.260_lnx86.Hotfix/CDROM1

安装过程中提示
To complete installing this release, you must specify the following media:  IC 06.18.000 RHEL 6(lnx86) Base Release Disk 1

安装hotfix包的时候同时需要base包，指定一下base包路径即可

    ..../IC06.18.000_lnx86.Base/CDROM1

### 弹出Config窗口，可按提示配置

### Spectre操作类似

## 安装完成后设置环境变量和启动配置
>.bashrc  
>.cdsinit




## Error

### ERROR 参考的对象类型不支持尝试的操作。[已退出进程，代码为 4294967295]
   
* 临时解决方案
   
   使用管理员身份打开**Powershell**或者**CMD.exe**，输入
   
    netsh winsock reset
   
* 开发人员给出的解决方案（一劳永逸）
   
   参考回答 https://github.com/microsoft/WSL/issues/4177#issuecomment-597736482  

  1.  下载[NoLsp工具](https://www.proxifier.com/tmp/Test20200228/NoLsp.exe)   

  2.  以管理员身份运行，并且以wsl.exe的完整路径作为参数，输入
    
            NoLsp.exe c:\windows\system32\wsl.exe`；

  3.  以 Windows Terminal 的 Powershell 为例，输入
    
          .\NoLsp.exe c:\windows\system32\wsl.exe
    


### error while loading shared libraries: libcrypto.so.10

    # cd /lib64
    # ll libcry*
    # ln -s libcrypto.so.1.1 libcrypto.so.10

    yum install compat-openssl10

#### libcrypto.so

    ln -s libcrypto.so.10 libcrypto.so

### libGLU.so.1

    yum install mesa-libGLU

### libXss.so.1

    yum install libXScrnSaver

### libnsl.so.1

    ln -s libnsl.so.2 libnsl.so.1

### libssl.so.10

    ln -s libssl.so libssl.so.10

### libaprutil-1.so.0

    yum install apr-util

### libdb-4.7.so

 ln -s libdb-5.3.so libdb-4.7.so




   

### lsb_release -bash: lsb_release: command not found
    yum install redhat-lsb-core
or

    yum install redhat-lsb





### 下载后的文件如果没有可执行权限，要先增加可执行权限：
     chmod +x *.exe



### ERROR ThunarThumbnailer

Failed to retrieve supported types: GDBus.Error:org.freedesktop.DBus.Error.ServiceUnknown: The name org.freedesktop.thumbnails.Thumbnailer1 was not provided by any .service files  
解决办法:

    yum install gnome-keyring

### 安装Xvfb

    yum install Xvfb




