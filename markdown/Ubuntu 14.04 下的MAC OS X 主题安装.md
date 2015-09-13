Ubuntu 14.04 下的MAC OS X 主题安装
---
>安装 MAC OS X 主题会帮助你的 Ubuntu 14.04 看起来更像MAC OS X。在这里我们介绍的Macbuntu安装包包含了GTK 主题，这些主题是专门为ubuntu unity定制的。图标主题可以为ubuntu 14.04使用，包括登陆界面引导启动的背景，登陆用户，甚至包括了lightdm 使用webkit的登陆界面。这个安装包是nobslab在bluedxca93的帮助下从gnome-look.org网站上开发的。

**第一步：下载壁纸**
>第一步要做的事仅仅是下载Mac OS X 的壁纸，下载连接在 [这里](http://drive.noobslab.com/data/Mac-13.10/MBuntu-Wallpapers.zip)这个压缩包的大小有39.2MB。 解压之后右键点击桌面->修改背景图片->选择下载的背景
>安装主题修改工具
为了修改GTK主题，图标，系统主题，光标，字体我们需要安装unity tweak。要安装unity tweak在ubuntu14.04上通过使用如下命令：
```
sudo apt-get install unity-tweak-tool
```

>当然你也可以通过安装ubuntu-tweak来实现主题更换

```
sudo add-apt-repository ppa:tualatrix/ppa
sudo apt-get update
sudo apt-get install ubuntu-tweak
```

>效果图：

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/172851tw17thwawpooawnk.jpeg)

**在ubuntu14.04上安装Mac OS X主题**
>为了修改上文所说的内容。我们需要打开终端运行如下命令：

```
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install mac-ithemes-v3
sudo apt-get install mac-icons-v3
```

>现在打开刚才安装的工具来选择主题，在GTK主题上选择MBuntu。再本地tab上选择Mbuntu-osx在光标tab上选择Mac-cursors.

>如图所示

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135743fpd202al2p0o1ppp.jpg)

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135744keh07q7rr0wfq70q.jpg)

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135746qsd207y0y0oxu2uu.jpg)

>现在unity桌面看起来就像Mac了。你已经有了mac的图标，mac的窗口样式，mac的鼠标指针样式。

**安装类似Mac OS X样式的DOCK在ubuntu14.04上**
>Docky是再UBUNTU平台上一个非常轻量级的类似Mac OS X 的dock。 它拥有mac一样的鼠标浮动效果。想要安装需要在终端上运行如下代码：
```
sudo add-apt-repository ppa:docky-core/ppa
sudo apt-get update
sudo apt-get install docky
```
**安装Mac doc主题**

>下载 [Mac 主题](http://drive.noobslab.com/data/Mac-14.04/Mac-OS-Lion%28Docky%29.tar)从unity启动器上运行docky。你就能看到docky运行再你的屏幕底端了，点击第一个docky配置按钮，选择3D mode点击下载主题按钮在上面选择Buyi-idock主题，现在你将会获得Mac OS X很像的dock了。

>配置图：

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135747u6lxnr7r8rgxpayi.jpg)

>效果图：

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135749feslgygxa9chgha6.jpg)

>隐藏unity 启动器
>再外观->行为中可以关闭启动器，
![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135751dxspqqhu7xu4z7pr.jpg)

**替换左上角的Ubuntu桌面为Mac OS X**
>想要修改成为Mac OS X执行下面命令
```
cd && wget -O Mac.po http://drive.noobslab.com/data/Mac-14.04/change-name-on-panel/mac.po
cd /usr/share/locale/en/LC_MESSAGES; sudo msgfmt -o unity.mo ~/Mac.po;rm ~/Mac.po;cd
```

>想要改回来执行下面命令
```
cd && wget -O Ubuntu.po http://drive.noobslab.com/data/Mac-14.04/change-name-on-panel/ubuntu.po
cd /usr/share/locale/en/LC_MESSAGES; sudo msgfmt -o unity.mo ~/Ubuntu.po;rm ~/Ubuntu.po;cd
```
`注意：执行完成命令之后需要重新登陆用户让修改生效。`
>效果图：![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135752no2b5lyll2c2oobt.jpg)

**替换延迟滚动条为正常滚动条**

>想要修改成为正常执行下面命令
```
gsettings set com.canonical.desktop.interface scrollbar-mode normal
```
>想要改回来执行下面命令
```
gsettings reset com.canonical.desktop.interface scrollbar-mode
```
**替换启动屏幕图片**

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135753lnd0b77z0nzi49m7.png)

>在这个小章节里面将会为ubuntu 14.04修改启动图片，包括载入动画跟苹果LOGO 命令如下
```
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install mbuntu-bscreen-v3
```
>想要修改回来：
```
sudo apt-get remove mbuntu-bscreen-v3
```
>修改Ubuntu14.04的登陆画面成为Mac OS X的样式
![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135755cmq50o4muo53tmqw.jpg)

>安装：
```
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install mbuntu-lightdm-v3
```
>修改回来：
```
sudo apt-get remove mbuntu-lightdm-v3
```
**去掉ubuntu 14.04锁屏的图标**

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135756w3zt35im33oj5m5g.jpg)

>去掉logo:
```
sudo xhost +SI:localuser:lightdm
sudo su lightdm -s /bin/bash
gsettings set com.canonical.unity-greeter draw-grid false;exit
sudo mv /usr/share/unity-greeter/logo.png /usr/share/unity-greeter/logo.png.backup
```
>如果想改回来：
```
sudo xhost +SI:localuser:lightdm
sudo su lightdm -s /bin/bash
gsettings set com.canonical.unity-greeter draw-grid true;exit
sudo mv /usr/share/unity-greeter/logo.png.backup /usr/share/unity-greeter/logo.png
```
**ubuntu14.04安装Mac字体**

>下载与安装字体：
```
wget -O mac-fonts.zip http://drive.noobslab.com/data/Mac-14.04/macfonts.zip
sudo unzip mac-fonts.zip -d /usr/share/fonts; rm mac-fonts.zip
sudo fc-cache -f -v
```

>配置： 启动tweak tool在字体选择上选择苹果系列的字体或者lucida Mac 字体，然后根据你的屏幕来调整字体。  

![enter image description here](https://dn-linuxcn.qbox.me/data/attachment/album/201407/29/135758cpy77tlglkwkvmpk.jpg)

>小伙伴们的测试结果

![enter image description here](https://github.com/lijianying10/FixLinux/raw/master/picture/MacTheme/1.jpg)

![enter image description here](https://github.com/lijianying10/FixLinux/raw/master/picture/MacTheme/2.jpg)

![enter image description here](https://github.com/lijianying10/FixLinux/raw/master/picture/MacTheme/3.jpg)