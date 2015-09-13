###虚拟机VirtualBox USB连接问题解决

生命在于折腾...
 
神奇的Linux...
 
Ubuntu 14.04 LTS
 
sudo apt-get install virtualbox -y

然后建好虚拟机之后（Windows也好，Linux也好），启动并点击菜单栏中的"设备"--“安装增强功能”
 
然后就会自动联网下载增强功能，当然也可以在官网下载双击安装，下载成功后，在虚拟机DVD驱动盘中你会发现增强功能安装包已经加载上去
 
双击安装...重启...
 
接着，解决虚拟机连接U盘的问题
 
sudo vim /etc/group

然后搜索vboxusers然后在末尾添加当前用户名
 
vboxusers:x:126:tracyone

然后重启电脑，重启之后，打开VirtualBox，然后点击你的虚拟机，点击设置，然后点击USB设备，点+..

![enter image description here](http://www.linuxidc.com/upload/2014_04/14042821427268.png)