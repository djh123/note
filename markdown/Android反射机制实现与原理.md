## Android反射机制实现与原理

####一、反射的概念及在Java中的类反射
　　 反射主要是指程序可以访问、检测和修改它本身状态或行为的一种能力。在计算机科学领域，反射是一类应用，它们能够自描述和自控制。这类应用通过某种机制来实现对自己行为的描述和检测，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。
　　 
 　　在Java中的反射机制，被称为Reflection（大家看到这个单词，第一个想法应该就是去开发文档中搜一下了）。它允许运行中的Java程序对自身进行检查，并能直接操作程序的内部属性或方法。Reflection机制允许程序在正在执行的过程中，利用Reflection APIs取得任何已知名称的类的内部信息，包括：package、 type parameters、 superclass、 implemented interfaces、 inner classes、 outer classes、 fields、 constructors、 methods、 modifiers等，并可以在执行的过程中，动态生成Instances、变更fields内容或唤起methods。
 　　
　　好，了解这些，那我们就知道了，我们可以利用反射机制在Java程序中，动态的去调用一些protected甚至是private的方法或类，这样可以很大程度上满足我们的一些比较特殊需求。你当然会问，反射机制在Android平台下有何用处呢？
　　
　　我们在进行Android程序的开发时，为了方便调试程序，并快速定位程序的错误点，会从网上下载到对应版本的Android SDK的源码（这里给大家提供一个2.3.3版本的下载链接）。你会发现很多类或方法中经常加上了“@hide”注释标记，它的作用是使这个方法或类在生成SDK时不可见，那么我们的程序可能无法编译通过，而且在最终发布的时候，就可能存在一些问题。
　　
　　那么，对于这个问题，第一种方法就是自己去掉Android源码中的"@hide"标记，然后重新编译生成一个SDK。另一种方法就是使用Java反射机制了，可以利用这种反射机制访问存在访问权限的方法或修改其域。
　　
　　废话半天，该入正题了，在进入正题之前，先给上一个反射测试类的代码，该代码中定义了我们需要进行反射的类，该类并没有实际的用途，仅供做为测试类。提示：本文提供的代码，并不是Android平台下的代码，而是一个普通的Java程序，仅仅是对Java反射机制的Demo程序，所以大家不要放在Android下编译啊，否则出现问题，别追究我的责任啦！

```java
ReflectionTest.java
package crazypebble.reflectiontest;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.Serializable;

public class ReflectionTest extends Object implements ActionListener,Serializable{
    // 成员变量
    private int bInt;
    public Integer bInteger = new Integer(4);
    public String strB = "crazypebble";
    private String strA;
    
    // 构造函数
    public ReflectionTest() {
        
    }
    
    protected ReflectionTest(int id, String name) { 
        
    }
    
    // 成员方法
    public int abc(int id, String name) {
        System.out.println("crazypebble ---> " + id + "-" + name);
        return 0;
    }
    
    protected static void edf() {
        
    }
    
    @Override
    public void actionPerformed(ActionEvent e) {
        // TODO Auto-generated method stub
        
    }
    
}
```


###二、反射机制中需要使用到的类
　　我把需要使用的类列在下表中，其中对我们特别有用的类，通过着重标记显示出来，并将在后面的使用中逐步解释：
　　
![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310321475.jpg)

###三、Class类
　　首先向大家说明一点，Class本身就是一个类，Class是该类的名称。看下面这个类的定义：
　　**public class MyButton extends Button {...}**
　　
 　　注意到上面的class的首字母是小写，它表示的是一种类类型，但是我们的Class是一个类，相当于上面定义的MyButton类。所以，千万不要把这里的Class做为一个类类型来理解。明白这一点，我们继续。
 　　
　　Class类是整个Java反射机制的源头，Class类本身表示Java对象的类型，我们可通过一个Object对象的getClass()方法取得一个对象的类型，此函数返回的就是一个Class类。获取Class对象的方法有很多种：
　　
![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310343653.jpg)


　在平时的使用，要注意对这几种方法的灵活运用，尤其是对Class.forName()方法的使用。因为在很多开发中，会直接通过类的名称取得Class类的对象。

###四、获取类的相关信息
####1、获取构造方法
　　Class类提供了四个public方法，用于获取某个类的构造方法。
　　Constructor getConstructor(Class[] params)     根据构造函数的参数，返回一个具体的具有public属性的构造函数
       Constructor getConstructors()     返回所有具有public属性的构造函数数组
　   Constructor getDeclaredConstructor(Class[] params)     根据构造函数的参数，返回一个具体的构造函数（不分public和非public属性）
       Constructor getDeclaredConstructors()    返回该类中所有的构造函数数组（不分public和非public属性）
　　由于Java语言是一种面向对象的语言，具有多态的性质，那么我们可以通过构造方法的参数列表的不同，来调用不同的构造方法去创建类的实例。同样，获取不同的构造方法的信息，也需要提供与之对应的参数类型信息；因此，就产生了以上四种不同的获取构造方法的方式。
　　
Android反射机制实现与原理 - Nelson - Nelsonget_Reflection_Constructors()

    /**
     * 获取反射类中的构造方法
     * 输出打印格式："Modifier修饰域   构造方法名(参数类型列表)"
     */
    public static void get_Reflection_Constructors(ReflectionTest r) {
        
        Class temp = r.getClass();
        String className = temp.getName();        // 获取指定类的类名
        
        try {
            Constructor[] theConstructors = temp.getDeclaredConstructors();           // 获取指定类的公有构造方法
            
            for (int i = 0; i < theConstructors.length; i++) {
                int mod = theConstructors[i].getModifiers();    // 输出修饰域和方法名称
                System.out.print(Modifier.toString(mod) + " " + className + "(");

                Class[] parameterTypes = theConstructors[i].getParameterTypes();       // 获取指定构造方法的参数的集合
                for (int j = 0; j < parameterTypes.length; j++) {    // 输出打印参数列表
                    System.out.print(parameterTypes[j].getName());
                    if (parameterTypes.length > j+1) {
                        System.out.print(", ");
                    }
                }
                System.out.println(")");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

####2、获取类的成员方法
　　与获取构造方法的方式相同，存在四种获取成员方法的方式。
　　　　Method getMethod(String name, Class[] params)    根据方法名和参数，返回一个具体的具有public属性的方法
　　　　Method[] getMethods()    返回所有具有public属性的方法数组
　　　　Method getDeclaredMethod(String name, Class[] params)    根据方法名和参数，返回一个具体的方法（不分public和非public属性）
　　　　Method[] getDeclaredMethods()    返回该类中的所有的方法数组（不分public和非public属性）
　　　　
Android反射机制实现与原理 - Nelson - Nelsonget_Reflection_Method()

    /**
     * 获取反射类的方法
     * 打印输出格式："RetType FuncName(paramTypeList)"
     */
    public static void get_Reflection_Method(ReflectionTest r) {
        
        Class temp = r.getClass();
        String className = temp.getName();
        
        /*
         * Note: 方法getDeclaredMethods()只能获取到由当前类定义的所有方法，不能获取从父类继承的方法
         * 方法getMethods() 不仅能获取到当前类定义的public方法，也能得到从父类继承和已经实现接口的public方法
         * 请查阅开发文档对这两个方法的详细描述。
         */
        //Method[] methods = temp.getDeclaredMethods();
        Method[] methods = temp.getMethods();
        
        for (int i = 0; i < methods.length; i++) {
            
            // 打印输出方法的修饰域
            int mod = methods[i].getModifiers();
            System.out.print(Modifier.toString(mod) + " ");
            
            // 输出方法的返回类型
            System.out.print(methods[i].getReturnType().getName());    
            
            // 获取输出的方法名
            System.out.print(" " + methods[i].getName() + "(");
            
            // 打印输出方法的参数列表
            Class[] parameterTypes = methods[i].getParameterTypes();
            for (int j = 0; j < parameterTypes.length; j++) {
                System.out.print(parameterTypes[j].getName());
                if (parameterTypes.length > j+1) {
                    System.out.print(", ");
                }
            }
            System.out.println(")");
        }
    }
    
　　在获取类的成员方法时，有一个地方值得大家注意，就是getMethods()方法和getDeclaredMethods()方法。
　　　　getMethods()：用于获取类的所有的public修饰域的成员方法，包括从父类继承的public方法和实现接口的public方法；
　　　　getDeclaredMethods()：用于获取在当前类中定义的所有的成员方法和实现的接口方法，不包括从父类继承的方法。
　　　　
大家可以查考一下开发文档的解释：
 getMethods() - Returns an array containing Method objects for all public methods for the class C represented by this Class.  Methods may be declared in C, the interfaces it implements or in the superclasses of C. The elements in the returned array are in no particular order. 
 getDeclaredMethods() - Returns a Method object which represents the method matching the specified name and parameter types that is declared by the class represented by this Class. 
　　因此在示例代码的方法get_Reflection_Method(...)中，ReflectionTest类继承了Object类，实现了actionPerformed方法，并定义如下成员方法：
 　　　　Android反射机制实现与原理 - Nelson - Nelson
　　通过这两个语句执行后的结果不同：
　　a、Method[] methods = temp.getDeclaredMethods()执行后结果如下：
　　　　Android反射机制实现与原理 - Nelson - Nelson
　　b、Method[] methods = temp.getMethods()执行后，结果如下：
　　　　　Android反射机制实现与原理 - Nelson - Nelson
####3、获取类的成员变量（成员属性）
　　存在四种获取成员属性的方法
　　　　Field getField(String name)    根据变量名，返回一个具体的具有public属性的成员变量
　　　　Field[] getFields()    返回具有public属性的成员变量的数组
　　　　Field getDeclaredField(String name)    根据变量名，返回一个成员变量（不分public和非public属性）
　　　　Field[] getDelcaredField()    返回所有成员变量组成的数组（不分public和非public属性）
　　　　
Android反射机制实现与原理 - Nelson - Nelsonget_Reflection_Field_Value()

    /**
     * 获取反射类中的属性和属性值
     * 输出打印格式："Modifier Type : Name = Value"
     * Note: 对于未初始化的指针类型的属性，将不输出结果
     */
    public static void get_Reflection_Field_Value(ReflectionTest r) {
        
        Class temp = r.getClass();    // 获取Class类的对象的方法之一
        
        try {
            System.out.println("public 属性");
            Field[] fb = temp.getFields();
            for (int i = 0; i < fb.length; i++) {
                
                Class cl = fb[i].getType();    // 属性的类型
                
                int md = fb[i].getModifiers();    // 属性的修饰域
                
                Field f = temp.getField(fb[i].getName());    // 属性的值
                f.setAccessible(true);
                Object value = (Object)f.get(r);
                
                // 判断属性是否被初始化
                if (value == null) {
                    System.out.println(Modifier.toString(md) + " " + cl + " : "      + fb[i].getName());
                }
                else {
                    System.out.println(Modifier.toString(md) + " " + cl + " : "      + fb[i].getName() + " = " + value.toString());
                }
            }
            
            System.out.println("public & 非public 属性");
            Field[] fa = temp.getDeclaredFields();
            for (int i = 0; i < fa.length; i++) {
                
                Class cl = fa[i].getType();    // 属性的类型
                
                int md = fa[i].getModifiers();    // 属性的修饰域
                
                Field f = temp.getDeclaredField(fa[i].getName());    // 属性的值
                f.setAccessible(true);    // Very Important
                Object value = (Object) f.get(r);
                
                if (value == null) {
                    System.out.println(Modifier.toString(md) + " " + cl + " : "      + fa[i].getName());
                }
                else {
                    System.out.println(Modifier.toString(md) + " " + cl + " : "  + fa[i].getName() + " = " + value.toString());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
####4、获取类、属性、方法的修饰域
　　类Class、Method、Constructor、Field都有一个public方法int getModifiers()。该方法返回一个int类型的数，表示被修饰对象（ Class、 Method、 Constructor、 Field ）的修饰类型的组合值。
　　在开发文档中，可以查阅到，Modifier类中定义了若干特定的修饰域，每个修饰域都是一个固定的int数值，列表如下：
　　![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310504727.jpg)
　　该类不仅提供了若干用于判断是否拥有某中修饰域的方法boolean isXXXXX(int modifiers)，还提供一个String toString(int modifier)方法，用于将一个表示修饰域组合值的int数转换成描述修饰域的字符串。
　　![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310515856.jpg)
　　
###五、如何调用类中的private方法
　　在介绍之前，先放一个代码吧，这段代码是参考其他文章的代码拷贝过来的，代码不算长，但是动态调用类的成员方法的过程讲解的通俗易懂。
　　
Android反射机制实现与原理 - Nelson - NelsonLoadMethod.java
```java
package crazypebble.reflectiontest;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;


public class LoadMethod {

    /**
     * 在运行时加载指定的类，并调用指定的方法
     * @param cName            Java的类名
     * @param MethodName    方法名
     * @param types            方法的参数类型
     * @param params        方法的参数值
     * @return
     */
    public Object Load(String cName, String MethodName, String[] types, String[] params) {
        
        Object retObject = null;
        
        try {
            // 加载指定的类
            Class cls = Class.forName(cName);    // 获取Class类的对象的方法之二
            
            // 利用newInstance()方法，获取构造方法的实例
            // Class的newInstance方法只提供默认无参构造实例
            // Constructor的newInstance方法提供带参的构造实例
            Constructor ct = cls.getConstructor(null);
            Object obj = ct.newInstance(null);    
            //Object obj = cls.newInstance();
            
            // 构建 方法的参数类型
            Class paramTypes[] = this.getMethodTypesClass(types);
            
            // 在指定类中获取指定的方法
            Method meth = cls.getMethod(MethodName, paramTypes);
            
            // 构建 方法的参数值
            Object argList[] = this.getMethodParamObject(types, params);
            
            // 调用指定的方法并获取返回值为Object类型
            retObject = meth.invoke(obj, argList);
            
        } catch (Exception e) {
            System.err.println(e);
        }
        
        return retObject;
    }
    
    /**
     * 获取参数类型，返回值保存在Class[]中
     */
    public Class[] getMethodTypesClass(String[] types) {
        Class[] cs = new Class[types.length];
        
        for (int i = 0; i < cs.length; i++) {
            if (types[i] != null || !types[i].trim().equals("")) {
                if (types[i].equals("int") || types[i].equals("Integer")) {
                    cs[i] = Integer.TYPE;
                } 
                else if (types[i].equals("float") || types[i].equals("Float")) {
                    cs[i] = Float.TYPE;
                }
                else if (types[i].equals("double") || types[i].equals("Double")) {
                    cs[i] = Double.TYPE;
                }
                else if (types[i].equals("boolean") || types[i].equals("Boolean")) {
                    cs[i] = Boolean.TYPE;
                }
                else {
                    cs[i] = String.class;
                }
            }
        }
        return cs;
    }
    
    /**
     * 获取参数Object[]
     */
    public Object[] getMethodParamObject(String[] types, String[] params) {
        
        Object[] retObjects = new Object[params.length];
    
        for (int i = 0; i < retObjects.length; i++) {
            if(!params[i].trim().equals("")||params[i]!=null){  
                if(types[i].equals("int")||types[i].equals("Integer")){  
                    retObjects[i]= new Integer(params[i]);  
                }
                else if(types[i].equals("float")||types[i].equals("Float")){  
                    retObjects[i]= new Float(params[i]);  
                }
                else if(types[i].equals("double")||types[i].equals("Double")){  
                    retObjects[i]= new Double(params[i]);  
                }
                else if(types[i].equals("boolean")||types[i].equals("Boolean")){  
                    retObjects[i]=new Boolean(params[i]);  
                }
                else{  
                    retObjects[i] = params[i];  
                }  
            } 
        }
        
        return retObjects;
    }
}
```

　　要调用一个类的方法，首先需要一个该类的实例（当然，如果该类是static，就不需要实例了，至于原因，你懂得！）。
####1、创建一个类的实例
　　在得到一个类的Class对象之后，我们可以利用类Constructor去实例化该对象。Constructor支持泛型，也就是它本身应该是Constructor<T>。这个类有一个public成员函数：T newInstance(Object... args),其中args为对应的参数，我们通过Constructor的这个方法来创建类的对象实例。
　　在代码LoadMethod.java和LoadMethodEx.java中，分别给出了两种实例化Class类的方法：一种是利用Constructor类调用newInstance()方法；另一种就是利用Class类本身的newInstance()方法创建一个实例。两种方法实现的效果是一样的。
// 利用newInstance()方法，获取构造方法的实例

// Class的newInstance方法，仅提供默认无参的实例化方法，类似于无参的构造方法

// Constructor的newInstance方法，提供了带参数的实例化方法，类似于含参的构造方法 

Constructor ct = cls.getConstructor(null);

Object obj = ct.newInstance(null);

Object obj = cls.newInstance();
####2、行为
　　Method类中包含着类的成员方法的信息。在Method类中有一个public成员函数：Object invoke(Object receiver, Object... args)，参数receiver指明了调用对象，参数args指明了该方法所需要接收的参数。由于我们是在运行时动态的调用类的方法，无法提前知道该类的参数类型和返回值类型，所以传入的参数的类型是Object，返回的类型也是Object。（因为Object类是所有其他类的父类）
　　如果某一个方法是Java类的静态方法，那么Object receiver参数可以传入null，因为静态方法从不属于对象。
#### 3、属性
　　对类的成员变量进行读写，在Field类中有两个public方法：
　　　　Object get(Object object)，该方法可用于获取某成员变量的值
　　　　Void set(Object object, Object value)，该方法设置某成员变量的值
　其中，Object参数是需要传入的对象；如果成员变量是静态属性，在object可传null。
 
###六、对LoadMethod.java的优化处理
　　在上一节中给出的LoadMethod.java中，类LoadMethod对固定参数类型的方法进行了调用，并且参数类型是通过一个String[]数组传入，然后经过方法 getMethodTypesClass() 解析之后，才得到了参数的具体的类型。同时在getMethodTypesClass()和getMethodParamObject()方法中，通过对传入的字符串参数进行过滤后，再处理那些可以匹配中的参数类型，其他不能匹配的参数都做为String对象来处理。如果我们调用的方法所需要的参数不是简单类型的变量，而是自定义的类对象，或者List列表，再如果我们只知道类名和方法名，不知道方法的参数类型，那我们该如何处理这些情况呢？
　　因此，我对LoadMethod类进行了一定的优化处理。先附上代码：
　　
Android反射机制实现与原理 - Nelson - NelsonLoadMethodEx.java
```java
package crazypebble.reflectiontest;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;


public class LoadMethodEx {

    /**
     * 在运行时加载指定的类，并调用指定的方法
     * @param cName            Java的类名
     * @param MethodName    方法名
     * @param params        方法的参数值
     * @return
     */
    public Object Load(String cName, String MethodName, Object[] params) {
        
        Object retObject = null;
        
        try {
            // 加载指定的类
            Class cls = Class.forName(cName);    // 获取Class类的对象的方法之二
            
            // 利用newInstance()方法，获取构造方法的实例
            // Class的newInstance方法只提供默认无参构造实例
            // Constructor的newInstance方法提供带参的构造实例
            Constructor ct = cls.getConstructor(null);
            Object obj = ct.newInstance(null);    
            //Object obj = cls.newInstance();

            // 根据方法名获取指定方法的参数类型列表
            Class paramTypes[] = this.getParamTypes(cls, MethodName);
            
            // 获取指定方法
            Method meth = cls.getMethod(MethodName, paramTypes);
            meth.setAccessible(true);
            
            // 调用指定的方法并获取返回值为Object类型
            retObject = meth.invoke(obj, params);
            
        } catch (Exception e) {
            System.err.println(e);
        }
        
        return retObject;
    }
    
    /**
     * 获取参数类型，返回值保存在Class[]中
     */
    public Class[] getParamTypes(Class cls, String mName) {
        Class[] cs = null;
        
        /*
         * Note: 由于我们一般通过反射机制调用的方法，是非public方法
         * 所以在此处使用了getDeclaredMethods()方法
         */
        Method[] mtd = cls.getDeclaredMethods();    
        for (int i = 0; i < mtd.length; i++) {
            if (!mtd[i].getName().equals(mName)) {    // 不是我们需要的参数，则进入下一次循环
                continue;
            }
            
            cs = mtd[i].getParameterTypes();
        }
        return cs;
    }
}
```
 
 我们通过前面几节的一系列分析，只要我们知道了一个类的类名（包括其包的路径），那我们就可以通过Class类的一系列方法，得到该类的成员变量、构造方法、成员方法、以及成员方法的参数类型和返回类型、还有修饰域等信息。
 
　　如果我们已经知道某个类名和需要动态调用的方法名，怎样才能不用传入方法的参数类型就可以调用该方法呢？
　　
     在已知类名的情况下，我们可以打印输出该类的所有信息，当然包括类的成员方法；然后通过给定的方法名，对打印输出的方法名进行筛选，找到我们需要的方法；再通过该方法的Method对象，得到该方法的参数类型、参数数量、和返回类型。那么我们在外部动态调用该方法时，就不需要关心该类需要传入的参数类型了，只需要传入类名、方法名、参数值的信息即可。笔者实现了一个类LoadMethodEx，先从两个类的同一个方法需要的参数方面做一个对比：
     ![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310590386.jpg)
       
![enter image description here](http://pic002.cnblogs.com/images/2011/277165/2011041310592526.jpg)


>1、LoadMethodEx类，少了一个参数（方法参数类型列表），本文直接从类LoadMethod内部获取该参数类型列表，不需要用户传入该信息，好处其实也不言而喻了。

>2、方法的参数值：类LoadMethod是将所有的方法参数都做为一个String来传入，在传入再进行解析；而本文则直接使用Object类型做为参数类型，因为invoke(Object obj, Object...args)方法本身所需要的参数类型就是Object，避免了不必要的参数类型变换。
　　在调用LoadMethod的Load()方法时，用户只需要知道类名、方法名，并且将已经初始化的参数先向上转型为Object，然后传递给Load()方法即可。方法的返回值为Object，这个肯定是由用户根据自己的需要，再转换成自己所需的类型。



 
执行结果
属性：
public 属性
public class java.lang.Integer : bInteger = 4
public class java.lang.String : strB = crazypebble
public & 非public 属性
private int : bInt = 0
public class java.lang.Integer : bInteger = 4
public class java.lang.String : strB = crazypebble
private class java.lang.String : strA

构造方法：
public crazypebble.reflectiontest.ReflectionTest()
protected crazypebble.reflectiontest.ReflectionTest(int, java.lang.String)

父类/接口：
父类: java.lang.Object
接口0: java.awt.event.ActionListener
接口1: java.io.Serializable

成员方法：
public int abc(int, java.lang.String)
public void actionPerformed(java.awt.event.ActionEvent)
public final native void wait(long)
public final void wait()
public final void wait(long, int)
public boolean equals(java.lang.Object)
public java.lang.String toString()
public native int hashCode()
public final native java.lang.Class getClass()
public final native void notify()
public final native void notifyAll()

反射机制调用方法:LoadMethod
crazypebble ---> 1-hello, android-1!

反射机制调用方法:LoadMethodEx
crazypebble ---> 2-hello, android-2?
返回结果：0



###七、总结
　　关于反射机制，其实还有一个比较敏感的话题，就是反射机制带来我们的安全性问题。由于我在这方面研究的不是很深入，所以讲不好。大家有空可以跟踪一下在本文最后提供的两个链接，里面有一些介绍。
　　我们介绍了Java的反射机制，但是在Android平台下，反射机制具体有没有什么用途呢？答案是肯定的。推荐大家看一篇文章《利用Java反射技术阻止通过按钮关闭对话框》，这篇文章为CSDN推荐为精品文章，所以还是很值得一看的。我特地从CSDN转载过来供大家一起学习。
　　原链接：http://blog.csdn.net/nokiaguy/archive/2010/07/27/5770263.aspx
　　转载链接：http://www.cnblogs.com/crazypebble/archive/2011/04/13/2014297.html
程序实现的源码：

Android反射机制实现与原理 - Nelson - NelsonAndroidReflection

```java
package crazypebble.androidreflection;

import java.lang.reflect.Field;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity {
    /** Called when the activity is first created. */
    private static Button btnHandler = null;
    private static Button btnShowing = null;
    AlertDialog alertDialog = null;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    
        btnHandler = (Button)findViewById(R.id.btn_mHandler);
        btnHandler.setOnClickListener(new ButtonListener());
        
        btnShowing = (Button)findViewById(R.id.btn_mShowing);
        btnShowing.setOnClickListener(new ButtonListener());
        
        alertDialog = new AlertDialog.Builder(this)
                .setTitle("abc")
                .setMessage("Content")
                .setIcon(R.drawable.icon)
                .setPositiveButton("确定", new PositiveClickListener())
                .setNegativeButton("取消", new NegativeClickListener())
                .create();
    }
    
    private class ButtonListener implements OnClickListener {

        @Override
        public void onClick(View v) {
            switch (v.getId()) {
            case R.id.btn_mHandler:
                modify_mHandler();
                alertDialog.show();
                break;
            case R.id.btn_mShowing:
                alertDialog.show();
                break;
            default:
                break;
            }
        }
    }
    
    private class PositiveClickListener implements android.content.DialogInterface.OnClickListener {

        @Override
        public void onClick(DialogInterface dialog, int which) {
            // 方法二时启用
            modify_dismissDialog(false);
        }
    }
    
    private class NegativeClickListener implements android.content.DialogInterface.OnClickListener {

        @Override
        public void onClick(DialogInterface dialog, int which) {
            // 方法一时启用
            //dialog.dismiss();

            // 方法二时启用
            modify_dismissDialog(true);
        }
    }
    
    /*
     * 第一种方法：修改AlertController类的private成员变量mHandler的值
     */
    public void modify_mHandler() {
        try {
            Field field = alertDialog.getClass().getDeclaredField("mAlert");
            field.setAccessible(true);
            // 获取mAlert变量的值
            Object obj = field.get(alertDialog);
            field = obj.getClass().getDeclaredField("mHandler");
            field.setAccessible(true);
            // 修改mHandler变量的值，使用新的ButtonHandler类
            field.set(obj, new MyButtonHandler(alertDialog));
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /*
     * 第二种方法：修改dismissDialog()方法
     */
    public void modify_dismissDialog(boolean flag) {
        try {
            Field field = alertDialog.getClass().getSuperclass().getDeclaredField("mShowing");
            field.setAccessible(true);
            // 将mShowing变量设为false，表示对话框已经关闭
            field.set(alertDialog, flag);
            alertDialog.dismiss();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

Android反射机制实现与原理 - Nelson - NelsonMyButtonHandler.java
package crazypebble.androidreflection;

import java.lang.ref.WeakReference;

import android.content.DialogInterface;
import android.os.Handler;
import android.os.Message;

public class MyButtonHandler extends Handler{

    // Button clicks have Message.what as the BUTTON{1,2,3} constant
    private static final int MSG_DISMISS_DIALOG = 1;
    
    private WeakReference<DialogInterface> mDialog;

    public MyButtonHandler(DialogInterface dialog) {
        mDialog = new WeakReference<DialogInterface>(dialog);
    }

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            
            case DialogInterface.BUTTON_POSITIVE:
            case DialogInterface.BUTTON_NEGATIVE:
            case DialogInterface.BUTTON_NEUTRAL:
                ((DialogInterface.OnClickListener) msg.obj).onClick(mDialog.get(), msg.what);
                break;
        }
    }
}

```