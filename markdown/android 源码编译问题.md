###android 源码编译问题
####问题一：
```
Your version is: java version "1.8.0_20". 
The correct version is: 1.6.
```

貌似很早之前Ubuntu里面的JDK就被换成了open-jdk,试了下官网给的方法：
```
$ sudo add-apt-repository "deb http://archive.canonical.com/ lucid partner" 
$ sudo apt-get update $ sudo apt-get install sun-java6-jdk
```
 结果不怎么给力，好像也安装不了，记得以前可以的...直接去oracle网站下载JDK，现在一进入下载就只有JDK1.7的，找了一下JDK1.6的下载地址，如下：
 http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html#jdk-6u45-oth-JPR，
 我记得我当时下载是单独注册了一个账号的，没有账号不让下载，太TM坑爹了。
 有了安装bin安装文件，我是直接放在我的用户目录下面的，然后执行：
```java
    chmod +x jdk-6u45-linux-x64.bin
   ./jdk-6u45-linux-x64.bin 
```

执行后会在用户目录里面生成jdk目录：/home/chadm/jdk1.6.0_45。然后配置Java环境，执行命令：
```
   ~$ sudo gedit /etc/profile

    在文件尾加上：

         export JAVA_HOME=/home/chadm/jdk1.6.0_45
         export JRE_HOME=/home/chadm/jdk1.6.0_45/jre  
         export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH  
```
   保存文件，然后执行：source /etc/profile。

   运行java -version和javac -version
####问题二：

external/clearsilver/cgi/cgi.c:22:18: 致命错误: zlib.h: 没有那个文件或目录编译终端。
```
sudo apt-get install zlib1g-dev
```

sudo gedit /etc/profile

####问题三：javac: 目标发行版 1.5 与默认的源发行版 1.8 冲突
关于更新Ubuntu 源的问题
不要把环境变量配置在/etc/environment和/etc/profile文件中，这样配置的在有的ubuntu版本上会出现退出当前终端后不起作用的问题，在ubuntu12.04上我就遇到了此问题。
把环境变量配置在用户目录.bashrc文件中是最好的选择。
```
export JAVA_HOME=/home/abc/jdk1.6.0_45
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
```
然后执行source .bashrc使配置生效即可。
由于ubuntu中可能会有默认的jdk，如openjdk。假如有openjdk的话，所以，为了使默认使用的是我们安装的jdk，还要进行如下工作。在终端充输入：
```
sudo update-alternatives --install /usr/bin/java java ~/abc/jdk1.6.0_45/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac ~/abc/jdk1.6.0_45/bin/javac 300
sudo update-alternatives --install /usr/bin/javac javap ~/abc/jdk1.6.0_45/bin/javap 300
```
通过这一步将我们安装的jdk加入java选单。
然后执行以下命令设置默认的java jdk
sudo update-alternatives --config java
例如：
~/projects$ sudo update-alternatives --config java
[sudo] password for abc: 
有 4 个候选项可用于替换 java (提供 /usr/bin/java)。

  选择       路径                                          优先级  状态
```
  0            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      自动模式
 *1            /home/abc/jdk1.6.0_45/bin/java             300       手动模式
  2            /usr/bin/gij-4.6                                 1046      手动模式
  3            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      手动模式
  4            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1051      手动模式
```
要维持当前值[*]请按回车键，或者键入选择的编号：

frameworks/base/core/java/android/content/pm/PackageParser.java:475: 找不到符号