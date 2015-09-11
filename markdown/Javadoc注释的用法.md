###Javadoc注释的用法

####一.Java 文档
// 注释一行
/* ...... */ 注释若干行
/** ...... */ 注释若干行，并写入 javadoc 文档

通常这种注释的多行写法如下：

/**
*.........
*.........
*/

javadoc -d 文档存放目录 -author -version 源文件名.java
这条命令编译一个名为"源文件名.java"的 java 源文件，并将生成的文档存放在"文档存放目录"指定的目录下，生成的文档中 index.html 就是文档的首页。-author 和 -version 两个选项可以省略。

####二. 文档注释的格式

1. 文档和文档注释的格式化

生成的文档是 HTML 格式，而这些 HTML 格式的标识符并不是 javadoc 加的，而是我们在写注释的时候写上去的。
比如，需要换行时，不是敲入一个回车符，而是写入 <br>，如果要分段，就应该在段前写入 <p>。
文档注释的正文并不是直接复制到输出文件 (文档的 HTML 文件)，而是读取每一行后，删掉前导的 * 号及 * 号以前的空格，再输入到文档的。如

/**
**This is first line. <br>
****This is second line. <br>
This is third line.
*/ 


2. 文档注释的三部分
先举例如下

/**
*show 方法的简述.
*show 方法的详细说明第一行
*show 方法的详细说明第二行
*@param b true 表示显示，false 表示隐藏
*@return 没有返回值
*/
public void show(boolean b) {
frame.show(b);
}

第一部分是简述。文档中，对于属性和方法都是先有一个列表，然后才在后面一个一个的详细的说明 
简述部分写在一段文档注释的最前面，第一个点号 (.) 之前 (包括点号)。换句话说，就是用第一个点号分隔文档注释，之前是简述，之后是第二部分和第三部分。

第二部分是详细说明部分。该部分对属性或者方法进行详细的说明，在格式上没有什么特殊的要求，可以包含若干个点号。 
*show 方法的简述.
*show 方法的详细说明第一行<br>
*show 方法的详细说明第二行

简述也在其中。这一点要记住了

第三部分是特殊说明部分。这部分包括版本说明、参数说明、返回值说明等。
*@param b true 表示显示，false 表示隐藏
*@return 没有返回值


三. 使用 javadoc 标记
javadoc 标记由"@"及其后所跟的标记类型和专用注释引用组成
javadoc 标记有如下一些：
@author 标明开发该类模块的作者 
@version 标明该类模块的版本 
@see 参考转向，也就是相关主题 
@param 对方法中某参数的说明 
@return 对方法返回值的说明 
@exception 对方法可能抛出的异常进行说明 

@author 作者名
@version 版本号
其中，@author 可以多次使用，以指明多个作者，生成的文档中每个作者之间使用逗号 (,) 隔开。@version 也可以使用多次，只有第一次有效

使用 @param、@return 和 @exception 说明方法
这三个标记都是只用于方法的。@param 描述方法的参数，@return 描述方法的返回值，@exception 描述方法可能抛出的异常。它们的句法如下：
@param 参数名参数说明
@return 返回值说明
@exception 异常类名说明


四. javadoc 命令
用法：
　　javadoc [options] [packagenames] [sourcefiles]

选项：

-public 仅显示 public 类和成员 
-protected 显示 protected/public 类和成员 (缺省) 
-package 显示 package/protected/public 类和成员 
-private 显示所有类和成员 
-d <directory> 输出文件的目标目录 
-version 包含 @version 段 
-author 包含 @author 段 
-splitindex 将索引分为每个字母对应一个文件 
-windowtitle <text> 文档的浏览器窗口标题 

javadoc 编译文档时可以给定包列表，也可以给出源程序文件列表。例如在 CLASSPATH 下有两个包若干类如下：

　　fancy.Editor
　　fancy.Test
　　fancy.editor.ECommand
　　fancy.editor.EDocument
　　fancy.editor.EView

可以直接编译类：
javadoc fancy\Test.java fancy\Editor.java fancy\editor\ECommand.java fancy\editor\EDocument.java fancy\editor\EView.java

也可以是给出包名作为编译参数，如：javadoc fancy fancy.editor
可以自己看看这两种方法的区别

到此为止javadoc就简单介绍完了，想要用好她还是要多用，多参考标准java代码(可参考JDK安装目录下的src源文件包)