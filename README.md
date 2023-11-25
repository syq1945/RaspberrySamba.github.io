# 树莓派搭建Samba 文件共享服务器


一、树莓派挂载U盘

1. 创建挂载目录
// 创建目录
sudo mkdir -p /home/pi/udisk
​
// 为目录设置读写权限
sudo chmod 777 /home/pi/udisk


2. 查看U盘信息
插入U盘到树莓派上后，输入 sudo fdisk -l 查看硬盘位置和硬盘格式

Device     Boot Start       End   Sectors  Size Id Type
/dev/sda1          32 120176639 120176608 57.3G  7 HPFS/NTFS/exFAT
如图所见，硬盘位置为 /dev/sda1 ,硬盘格式是NTFS格式

U盘最好先做好格式化，在windows上面格式化为NTFS格式，否则可能会出现挂载中文文件名乱码等奇怪问题

3. 挂载U盘到树莓派
将硬盘位置/dev/sda1 挂载到我们创建的udisk挂载目录

sudo mount /dev/sda1 /home/pi/udisk


挂载报错：Mount is denied because the NTFS volume is already exclusively opened.
The volume may be already mounted, or another software may use it which
could be identified for example by the help of the ‘fuser’ command.

出现错误的原因：已经被挂载或者有应用程序正在使用它。

执行命令$fuser  /dev/sda1 查看正在使用它的应用程序
root@pi:/home/pi// fuser /dev/sda1
/dev/sda1:            1918

root@pi:/home/pi// kill 1918
root@pi:/home/pi// sudo mount /dev/sda1 /home/pi/udisk

执行命令：$kill 1918 杀死它。
再次挂载：sudo mount /dev/sda1 /home/pi/udisk

成功！

4. 测试挂载效果
创建文件查看是否挂载成功，我们可以使用vim创建中文名文件看中文格式是否正常

sudo vim 测试.txt

vim是Linux下最常用的文本编译器，在终端输入vim时可能会出现

Connand 'vim' not found, but can be installed with:

这是因为默认的文本编译器是vi, 而没有安装vim的缘故

输入 sudo apt-get install vim 安装vim


sudo nano /etc/apt/sources.list


二、设置开机自动挂载
要实现开机自动挂载U盘，我们需要将U盘的设备信息写入到/etc/fstab文件中

1. 查看硬盘UUID信息
blkid

我们记录下信息准备后面使用：
 /dev/sda1:  UUID="0CDC5408DC53EA8A"  PARTUUID="69016b36-01"

2. 挂载信息写入配置文件
查看下cat /etc/fstab文件的内容

将以下信息，添加到/etc/fstab文件末尾
使用nano编辑/etc/fstab文件，追加下面的内容
sudo nano  /etc/fstab

UUID=0CDC5408DC53EA8A  /home/pi/udisk  ntfs defaults 0 0


3. 使配置生效
sudo mount -a


4. 查看挂载情况
df -h

5.重启服务查看自动挂载效果

//重启
sudo reboot


三、树莓派部署Samba服务
1. 安装samba服务
sudo apt-get install samba


2. 创建账户与设置密码
这里把pi为用samba的登录用户，并设置密码

// 创建samba配置的密码文件
sudo touch /etc/samba/smbpasswd
​
// 添加smb账户
sudo smbpasswd -a pi
执行smbpasswd命令后会提示输入samba的账户密码，这个密码后面访问smb服务会用到，
我这里使用pi这个默认root用户所以不用新建，用户需要在系统中存在，没有则先用useradd创建。


3. 设置samba的配置文件
需要在smaba配置中指定相关的smb共享文件夹

sudo nano /etc/samba/smb.conf
将如下配置添加到smb.conf最后面

//树莓派本磁盘配置，挂载的是test目录
[local]
    comment = local
    path = /home/pi/test
    browseable=yes
    writable=yes
    //public=yes
    //available= yes

    //这一行是特别重要的，设置可以被共享访问！！
    guest ok=yes

    create mask = 0775
    directory mask = 0775

    //将新文件的所有者和组属性设置为属于root用户
    forceuser = pi
    forcegroup = pi
​
// u盘挂载的配置，目录是挂载目录
[udisk]
    comment = udisk
    path = /home/pi/udisk
    browseable=yes
    writable=yes
    //public=yes
    //available= yes

    //这一行是特别重要的，设置可以被共享访问！！
    guest ok=yes

    create mask = 0775
    directory mask = 0775

    //将新文件的所有者和组属性设置为属于root用户
    forceuser = pi
    forcegroup = pi

上面我添加了两个共享目录配置，
一个是local共享的是树莓派本地的文件夹/home/ubuntu/test，
一个是udisk共享的是U盘挂载的/home/ubuntu/udisk文件夹，
大家可以根据自己情况设置一个或是多个

各个参数具体含义如下：

[udisk]：分享名称
comment：备注描述
path：共享文件夹目录
writable：是否可写入，不能写入就不能创建文件夹
browseable：是否可以访问浏览


4. 重启服务使配置生效

使用下面的命令效果相同

// 重启服务
sudo service smbd restart
​
// 查看服务状态
sudo service smbd status


5. 设置开机自动启动
// 开机自启动
sudo systemctl enable smbd


sudo chmod 777 /home/pi/udisk
sudo chmod 777 /home/pi/test

sudo service smb restart 
