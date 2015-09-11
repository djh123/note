####AIDL的学习
1.今天学习了android中aidl开发，即通过aidl实现android中两个进程之间的通信，我们通过在服务端利用aidl文件暴露接口，然后再客户端通过ServiceConnection来绑定这些接口，进而实现调用。
首先新建服务端工程：
2.在com.example.aidlservice包下，新建一个ICat.aidl文件，内容如下：
```
[java] view plaincopy
package com.example.aidlservice;  
  
interface ICat  
{  
    String getColor();  
    double getWeight();  
} 
```
此时，ADT会在gen目录下，自动生成ICat.java文件
接下来定义一个Service实现类，该Service的onBind()方法所返回的IBinder对象就是ADT生成的ICat.Stub的子类的实例
收先定义一个类继承自stub，需要实现刚才在ICat.aidl中定义的方法：
```
[java] view plaincopy
public class IcatBinder extends Stub {  
  
        @Override  
        public String getColor() throws RemoteException {  
            // TODO Auto-generated method stub  
            return colors[(int)(Math.random()*3)];  
        }  
  
        @Override  
        public double getWeight() throws RemoteException {  
            // TODO Auto-generated method stub  
            return weights[(int)(Math.random()*3)];  
        }  
    }
``` 
需要在onBind方法中返回该IcatBinder的实例对象,IcatBinder对象可以在onCreate中实例化

另外还需要在AndroidManifest.xml中，配置该service
```
[html] view plaincopy
<service android:name="com.example.aidlservice.AidlService">  
            <intent-filter>  
                <action android:name="com.example.action.myaidl"/>  
            </intent-filter>  
 </service>  
```
这里需要注意的是，这里的action，是用来在客户端绑定该service用的


接下来就是新建客户端工程访问AIDLService
AIDL定义了连个进程之间通信的接口，因此不仅服务端需要AIDL接口，客户端同样需要之前定义的AIDL接口，将之前定义的接口拷贝到客户端的和服务端同名的包内：
即在客户端新建一个包名称和服务端存放ICat.aidl同样包名的包：com.example.aidlservice  ，然后需要将服务端的ICat.aidl文件拷贝过来
 
客户端绑定是通过ServiceConnection实现的：
首先声明绑定的代理：private ICat catService;
然后实现：ServiceConnection
```
[java] view plaincopy
private ServiceConnection conn = new ServiceConnection() {  
  
        @Override  
        public void onServiceDisconnected(ComponentName arg0) {  
            // TODO Auto-generated method stub  
            catService = null;  
        }  
  
        @Override  
        public void onServiceConnected(ComponentName arg0, IBinder service) {  
            // TODO Auto-generated method stub  
            catService = ICat.Stub.asInterface(service);  
        }  
}; 
```
接下来就可已调用服务端的方法了：catService.getColor()

===================================================================
下面看看在客户端和服务端传递自定义的数据类型：
需要注意的是，我们自定义的数据类型，如果需要在客户端和服务端之间传递，那么需要实现Parcelable接口:
我们新建一个Person类实现Parcelable接口：
重写writeToParcel方法：
```
[java] view plaincopy
@Override  
public void writeToParcel(Parcel dest, int arg1) {  
    // TODO Auto-generated method stub  
    dest.writeInt(personId);  
    dest.writeString(personName);  
    dest.writeString(personPass);  
    dest.writeInt(personAge);  
} 
```
并且需要新建一个CREATOR的实例变量，这个变量的类型必须是：
``public static final Parcelable.Creator<Person>`这样的
这里需要注意的是这个实例变量的名称必须是：**CREATOR**，这是因为ADT在gen目录下生成的IPerson.java中生成的这样名称
```
[java] view plaincopy
public static final Parcelable.Creator<Person> CREATOR = new Parcelable.Creator<Person>() {  
  
        @Override  
        public Person createFromParcel(Parcel arg0) {  
            // TODO Auto-generated method stub  
            Person person = new Person(arg0.readInt(),arg0.readString(),arg0.readString(),arg0.readInt());  
            return person;  
        }  
  
        @Override  
        public Person[] newArray(int arg0) {  
            // TODO Auto-generated method stub  
            return new Person[arg0];  
        }  
};  
```
当**person**类声明好了以后，首先需将该类进行定义：
在存放**aidl**文件的包下，新建一个Person.aidl文件，内容如下：
```
parcelable Person; 
```
然后新建需要暴漏给客户端调用的接口：
新建一个Iperson.aidl,内容如下：
```
[html] view plaincopy
package com.example.aidlservice;  
  
import java.util.List;  
import com.example.aidlservice.Person;  
  
interface IPerson  
{  
    List<Person> getAllPerson();  
    Person getPersonById(in int personId);  
}  
```
这里需要注意的是，必须得引入对应的包名和类，如果需要接受参数，需要需要在需要接受参数的数据类型前边加上in。

需要注意的是：
- 1.如果需要传递自定义的类型：需要实现Parcelable接口
- 2.服务端定义的service必须在Manifest.xml文件中声明
- 3.如果需要传递自定义类型，还需要将自定义类型声明为aidl文件
- 4.在服务端声明的aidl文件，在客户端必须要有一份相同的文件存在，并且，包名必须相同。这是因为aidl文件就相当于调用的接口，包名必须相同，通过ServiceConnection绑定该接口，然后，通过该aidl生成的接口调用对应的服务端的方法。