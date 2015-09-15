###Linux下Android NDK环境配置以及配合eclipse使用NDK

一.下载NDK
http://developer.android.com/sdk/ndk/index.html
二.配置环境
可以配置到  ~/.bashrc 文件 也可以  /etc/profile   这边以后者为例
```
sudo gedit /etc/profile 
```

在配置文件中加入如下部分：
```
export ANDROID_NDK=/home/feicong/tools/android-ndk-r8  
export PATH=/home/feicong/tools/android-ndk-r8:$PATH
```

保存文件后退出，在终端提示符中输入**"source /etc/profile"**使环境变量生效。 

三.代码的编写
**1.首先是写java代码**
建立一个Android应用工程HelloJni，创建HelloJni.java文件：
```
HelloJni.java:
代码:
package com.xxx.hello;
import android.app.Activity;
importandroid.os.Bundle;
importandroid.widget.TextView;
public class HelloJniextendsActivity {
/**Called when the activity is first created. */
@Override
publicvoid onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
TextViewtv =newTextView(this);
tv.setText(stringFromJNI());
setContentView(tv);
}
public native String stringFromJNI();
static{
System.loadLibrary("hello-jni");
}
}
```
这段代码很简单，注释也很清晰，这里只提两点：
- a:
```
static{
System.loadLibrary("hello-jni");
}
```
>表明程序开始运行的时候会加载hello-jni,static区声明的代码会先于onCreate方法执行。如果你的程序中有多个类，而且如果HelloJni这个类不是你应用程序的入口，那么hello-jni（完整的名字是libhello-jni.so）这个库会在第一次使用HelloJni这个类的时候加载。

- b:
```
publicnative String stringFromJNI();
```
>可以看到这两个方法的声明中有native关键字，这个关键字表示这两个方法是本地方法，也就是说这两个方法是通过本地代码（C/C++）实现的，在java代码中仅仅是声明。
用eclipse编译该工程，生成相应的.class文件，这步必须在下一步之前完成，因为生成.h文件需要用到相应的.class文件。

**2.编写相应的C/C++代码**
刚开始学的时候，有个问题会让人很困惑，相应的C/C++代码如何编写，函数名如何定义？这里讲一个方法，利用javah这个工具生成相应的.h文件，然后根据这个.h文件编写相应的C/C++代码。

>- 2.1生成相应.h文件：
就拿我这的环境来说，首先在终端下进入刚刚建立的HelloJni工程的目录：

>>```
代码:
xxx@xion-driver:~$cd android_env/eclipse/Workspace/HelloJni/
ls查看工程文件
代码:
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$ls
AndroidManifest.xml assets bin default.properties gen res src
可以看到目前仅仅有几个标准的android应用程序的文件（夹）。
首先我们在工程目录下建立一个jni文件夹：
代码:
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$mkdir jni
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$ls
AndroidManifest.xml assets bin default.properties gen jni res src
下面就可以生成相应的.h文件了：
代码:
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$javah -classpath bin/classes -d jni com.xxx.hello.HelloJni
-classpathbin：表示类的路劲
-djni: 表示生成的头文件存放的目录
com.xxx.hello.HelloJni则是完整类名
```
**现在可以看到jni目录下多了个.h文件：**
```
```
代码:
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$cd jni/
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni/jni$ls
com_xxx_hello_HelloJni.h
```
```
我们来看看com_xxx_hello_HelloJni.h的内容：
com_xxx_hello_HelloJni.h:
```
```
代码:
/* DONOT EDIT THIS FILE - it is machine generated */
#include<jni.h>
/*Header for class com_xxx_hello_HelloJni */
#ifndef_Included_com_xxx_hello_HelloJni
#define_Included_com_xxx_hello_HelloJni
#ifdef__cplusplus
extern"C" {
#endif
/*
*Class: com_xxx_hello_HelloJni
*Method: stringFromJNI
*Signature: ()Ljava/lang/String;
*/
JNIEXPORTjstring JNICALL Java_com_xxx_hello_HelloJni_stringFromJNI
(JNIEnv*, jobject);
#ifdef__cplusplus
}
#endif
#endif
```


>>上面代码中的JNIEXPORT和JNICALL是jni的宏，在android的jni中不需要，当然写上去也不会有错。
从上面的源码中可以看出这个函数名那是相当的长啊。。。。不过还是很有规律的， 完全按照：java_pacakege_class_mathod形式来命名。
也就是说：
Hello.java中stringFromJNI()方法对应于C/C++中的Java_com_xxx_hello_HelloJni_stringFromJNI方法
注意下其中的注释：
代码:
Signature:()Ljava/lang/String;
()Ljava/lang/String;
()表示函数的参数为空（这里为空是指除了JNIEnv*, jobject这两个参数之外没有其他参数，JNIEnv*,jobject是所有jni函数必有的两个参数，分别表示jni环境和对应的java类（或对象）本身），
Ljava/lang/String;表示函数的返回值是java的String对象。

**2.2编写相应的.c文件：**
```
hello-jni.c:
代码:
#include<string.h>
#include<jni.h>
JNIEXPORTjstring JNICALL Java_com_xxx_hello_HelloJni_stringFromJNI(JNIEnv*env, jobject obj)
{
return(*env)->NewStringUTF(env,"Hello from JNI !");
}
```
**Java_com_xxx_hello_HelloJni_stringFromJNI**函数只是简单的返回了一个内容为"Hellofrom JNI !"的jstring对象（对应于java中的String对象）。
hello-jni.c文件已经编写好了，现在可以把com_xxx_hello_HelloJni.h文件给删了，当然留着也行，只是我还是习惯把不需要的文件给清理干净了。

**3.编译hello-jni.c生成相应的库**
**3.1编写Android.mk文件**


在jni目录下（即hello-jni.c同级目录下）新建一个Android.mk文件，Android.mk文件是Android的makefile文件，内容如下：

```
代码:
LOCAL_PATH:= $(call my-dir)
include$(CLEAR_VARS)
LOCAL_MODULE := hello-jni
LOCAL_SRC_FILES:= hello-jni.c
include$(BUILD_SHARED_LIBRARY)

LOCAL_PATH:= $(call my-dir)

```

一个Android.mk文件首先必须定义好LOCAL_PATH变量。它用于在开发树中查找源文件。在这个例子中，宏函数’my-dir’,由编译系统提供，用于返回当前路径（即包含Android.mkfile文件的目录）。
```
include$( CLEAR_VARS)

CLEAR_VARS由编译系统提供，指定让GNUMAKEFILE为你清除许多LOCAL_XXX变量（例如LOCAL_MODULE, LOCAL_SRC_FILES,LOCAL_STATIC_LIBRARIES, 等等...),
除LOCAL_PATH。这是必要的，因为所有的编译控制文件都在同一个GNUMAKE执行环境中，所有的变量都是全局的。
LOCAL_MODULE:= hello-jni
编译的目标对象，LOCAL_MODULE变量必须定义，以标识你在Android.mk文件中描述的每个模块。名称必须是唯一的，而且不包含任何空格。
注意：编译系统会自动产生合适的前缀和后缀，换句话说，一个被命名为'hello-jni'的共享库模块，将会生成'libhello-jni.so'文件。
重要注意事项：
如果你把库命名为‘libhello-jni’，编译系统将不会添加任何的lib前缀，也会生成'libhello-jni.so'，这是为了支持来源于Android平台的源代码的Android.mk文件，如果你确实需要这么做的话。
LOCAL_SRC_FILES:= hello-jni.c
LOCAL_SRC_FILES变量必须包含将要编译打包进模块中的C或C++源代码文件。注意，你不用在这里列出头文件和包含文件，因为编译系统将会自动为你找出依赖型的文件；仅仅列出直接传递给编译器的源代码文件就好。
注意，默认的C++源码文件的扩展名是’.cpp’.指定一个不同的扩展名也是可能的，只要定义LOCAL_DEFAULT_CPP_EXTENSION变量，不要忘记开始的小圆点（也就是’.cxx’,而不是’cxx’）
include$(BUILD_SHARED_LIBRARY)
BUILD_SHARED_LIBRARY表示编译生成共享库，是编译系统提供的变量，指向一个GNUMakefile脚本，负责收集自从上次调用'include$(CLEAR_VARS)'以来，定义在LOCAL_XXX变量中的所有信息，并且决定编译什么，如何正确地去做。还有BUILD_STATIC_LIBRARY变量表示生成静态库：lib$(LOCAL_MODULE).a，BUILD_EXECUTABLE表示生成可执行文件。

```
**3.2生成.so共享库文件**
代码:
```
xxx@xion-driver:~/android_env/eclipse/Workspace/HelloJni$ndk-build
Install : libhello-jni.so => libs/armeabi/libhello-jni.so
```

可以看到已经正确的生成了libhello-jni.so共享库了。
4.在eclipse重新编译HelloJni工程，生成apk
eclipse中刷新下HelloJni工程，重新编译生成apk，libhello-jni.so共享库会一起打包在apk文件内。

####遇到的问题
Android NDK: No rule to make target 什么什么.o


###stackoverflow 
给出了如下解答
![Alt text](./QQ截图20150806124447.png)


他说你的(call my -dir) 后面有横线 或者空格什么的  ,,,   妹子的  还真有 把后面空删除就可以了