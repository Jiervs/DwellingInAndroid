# Review For Android
***
## RemoteView
***
### RemoteView 的应用   

**1.**通知栏上的应用：使用系统默认的样式弹出一个通知  (**Notification** 类的使用)  

**2.** **RemoteViews** 在桌面小部件的应用，**AppWidgetProvider** 是 **Android** 中提供的用于实现桌面小部件的类，是 **BroadcastProvider** 的继承类，小部件的开发步骤一般分为如下几步：  
(1). 定义小部件界面 : 在 **res/layout/** 下新建一个 **XML** 文件，命名为 **widget.xml**，名称和内容可以自定义.   
(2). 定义小部件配置信息 : 在 **res/xml/** 下新建 **appwidget\_provider\_info.xml**,名称随意选择  
(3). 定义小部件的实现类 : 这个类需要继承 **AppWidgetProvider**.   
(4). 在 **AndroidManifest.xml** 中声明小部件.  
具体案例 : 见 **《Android开发艺术探索》- 任玉刚 - p221 - p227**  

**3.** **AppWidgetProvider** 的一些方法：  
**onEnable()** : 当该窗口小部件第一次添加到桌面时调用该方法，可添加多次但是只在第一次调用.  
**onDeleted()** : 每删除一次桌面小部件就调用一次.  
**onDisabled()** :  当最后一个该类型的桌面小部件被删除时调用该方法，注意是最后一个.  
**onReceive()** : 这是广播的内置方法，用于分发具体的事件给其他方法.

***
### PendingIntent 概述 