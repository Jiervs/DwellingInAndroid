#Review For Android
***
##IPC机制 (Inter-Process Communication)
***
###Android IPC 简介   
按照操作系统中的描述，**线程**是 **CPU** 调度的最小单元，**线程** 是一种有限的系统资源 .  而 **进程** 一般指一个执行单元，在 **PC** 和移动设备上指一个程序或者一个应用.一个**进程**可以包含多个 **线程** . 在 **Android** 里面主线程也叫 **Ui** 线程，在 **Ui** 线程才能操作界面元素，当在主线程中执行耗时操作则会造成界面无法响应，即 **ANR** (**Application Not Responding**) ,应用无响应.
***
###Android 中的多进程模式

**1**.**Android** 中多进程是指一个应用中存在多个进程的情况（这里暂不讨论两个应用之间的多进程情况）. 在 **Android** 中使用多进程只有一种方法：就是给四大组件在 **AndroidMenifest** 中指定 <font size = 4 color = blue>`android:process` </font> 属性.（还有一种非常规方法，详情见**《Android开发艺术探索》- 任玉刚 - p37**），没有指定的情况下，默认进程的进程名是包名.  

**2**.进程名以 ”：“开头的进程属于当前应用的私有进程，其他应用组件不可以和它跑在同一个进程中，而进程名不以”：“开头的进程属于全局进程，其他应用通过 **ShareUID** 方式可以和它跑在同一个进程中.

**3**.一般来说，使用多进程会造成如下几方面的问题：  
(1).静态成员和单例模式完全失效.   
(2).线程同步机制完全失效.   
(3). **SharePreference** 的可靠性下降.   
(4). **Appliction** 会多次创建.  

详情见**《Android开发艺术探索》- 任玉刚 - p40**    
结论：在上述多进程模式中，不同进程的组件的确会拥有独立的虚拟机，**Application**  以及内存空间.   
***
### **IPC** 的基础概念
**1**. **Serializeble** 是 **Java** 所提供的一个序列化接口，使用 **Serializeble** 方式进行序列化只需要在类的声明中指定一个类似下面的标识即可自动实现默认的序列化过程：  
<font size = 3 color = blue>`private static final long serialVersionUID = 876512345634128908L` </font>  
对对象的序列化和反序列化只需要采用 **ObjectOutputStream** 和 **ObjectInputStream** 即可轻松实现，如下：  
**序列化过程** ：  
    User user = new User("Jiervs",1);  
    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("cache.txt"));   
    out.writeObject(user);  
    out.close();   
    
**反序列化过程** ：  
    ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.txt"));  
    User userNew = (User) in.readObject();   
    in.close(); 
    
静态成员变量属于类不属于对象，所以不会参与序列化过程，其次可以用 **transient** 关键字标记成员变量不参与序列化过程.

**2**.在 **Android** 中提供了新的序列化方式， **Parcelable** 接口.实现该接口以后就可以实现序列化并可以通过 **Intent** 和 **Binder** 传递，详情见**《Android开发艺术探索》- 任玉刚 - p45**  
**Intent** ， **Bundle** ， **Bitmap** 是可以直接序列化的，系统已经实现了 **Parcelable** 接口.  **Parcelable** 的序列化效率 比 **Serializeble** 高，开销更小，但是 **Parcelable** 主要用在内存的序列化上，如果是将对象序列化到存储设备中或者将对象序列化后进行网络传输 ，使用 **Parcelable** 过程会稍微复杂，此两种情况下建议使用 **Serializeble**.  

***
### **Binder**  
( **Binder** 机制较为复杂 ,详情见**《Android开发艺术探索》- 任玉刚 - p47 - p61**   )  
**Binder** 实现了 **IBinder** 接口，从 **Android** 应用层来说，**Binder** 是客户端和服务端进行通信的媒介，当 **bindService** 的时候，服务端会返回一个包含了服务端业务调用的 **Binder** 对象，通过这个 **Binder** 对象，客户端就可以获取服务端提供的服务或者数据，这里服务包括普通服务和基于 **AIDL** 的服务.  **Binder** 的工作机制：  

<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/binder.png" width = 500 />  


