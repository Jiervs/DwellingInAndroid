#Review For Android
***
##Activity
***
###生命周期

**1**.虽然 **onStart()** 和 **onResume()** 都表示 **Activity** 已经可见，但是 **onStart()** 的时候 **Activity** 还在后台， **onResume()** 的时候 **Activity** 才会显示到前台.  

**2**.前一个 **Activity** 的 **onPause()** 必须前执行完，后一个 **Activity** 的 **onResume()** 才会被执行（基于5.0 源码），在 **onPause()** 和 **onStop()** 不要做太耗时的操作， **onStop()** 中可进行一些稍微重量级的回收操作，在 **onDestory()** 中可以做最终的资源释放.  

**3**.当新打开的 **Activity** 采用透明主题，那么当前 **Activity** 不会回调 **onStop()** .  

**4**.当系统配置发生改变后， **Activity** 会被销毁，由于是在异常情况下终止的，系统会调用 **onSaveInstanceState()** 来保存当前状态，在 **onStop()** 前调用，当 **Activity** 被重新创建后，系统会调用 **onRestoreInstanceState()** ,并且把 **Activity** 销毁时 **onSaveInstanceState()** 所保存的 **Bundle** 对象作为参数传递给 **onRestoreInstanceState()** 和 **onCreate()** , **onRestoreInstanceState()** 在 **onStart()** 之后.  

**5**. **View** 和 **Activity** 同样具有  **onSaveInstanceState()** 和  **onRestoreInstanceState()** ,关于保存和恢复 **View** 的层次结构 ： 委托思想.  

**6**. **Activity** 优先级一般分为3种:  
(1)前台 **Activity** ：正在和用户交互，优先级最高.  
(2)可见但非前台 **Activity** .  
(3)后台 **Activity** ：执行了 **onStop()** 优先级最低.  
	
当系统内存不足时，系统会按照上述优先级杀死目标 **Activity** 所在的 **进程**，脱离四大组件的进程很快被系统杀死，后台工作放入 **Service** 从而保证进程具有一定的优先级.
	
**7**.如果不想系统在系统配置发生改变后重新创建 **Activity** ，可以给 **Activity** 指定 **configChanges** 属性，例：  

<font size = 4 color = blue>`android:configChanges="orientation"`</font>  

之后系统不调用  **onSaveInstanceState()** 和  **onRestoreInstanceState()** ，而是调用 **onConfigurationChanged()** .  
***

###启动模式

**1**.4种 **LauchMode** ：(1) **standard**  (2) **singleTop**  (3) **singleTask**  (4) **singleInstance** .  

**2**.用 **ApplicationContext** 去启动 **standard** 模式的 **Activity** 时，会出现：  

<font size = 3 color = red>`android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?`</font>  

因为非 **Activity** 类型的 **Context** 并没有所谓的 任务栈 ， 解决方法 是为 待启动 的 **Activity** 指定  
<font size = 4 color = blue>`FLAG_ACTIVITY_NEW_TASK` </font> 标记位，这样启动 **Activity** 时会创建一个新的任务栈，本质以 **singleTask** 模式启动.

**3**.在 **singleTop** （栈顶复用模式中），**Activity** 不会被重新创建，会调用 **onNewIntent()** .  

**4**.在 **singleTask** （栈内复用模式中），该模式下的 **Activity** 请求启动后，系统首先会寻找是否存在该 Acitivity 想要的任务栈，如果不存在，就重新创建一个任务栈，然后把该 **Activity** 的实例放在栈中，如果存在需要的任务栈，这时需要看该 **Activity** 的实例在栈中是否存在，存在的话就会调用该 **Activity** 的 **onNewIntent()**.  

**5**. **singleTask** 默认具有 **clearTop** 的效果 ，例:栈 **S1** 中 **Activity** 实例有 **ADBC** ，**D** 需要栈 **S1**，以 **singleTask** 启动，最终栈 **S1** 中 **Activity** 实例为 **AD**.  

**6**. **singleInstance** （单实例模式），一种加强的 **singleTask**， 系统会为该模式下的 **Activity** 建一个新的任务栈 ，并独立存在栈中，除非这个独特的任务栈被系统销毁，不然不会创建新的 **Activity**.  

**7**.  **TaskAffinity**：任务相关性. 这个参数标识了一个 **Activity** 所需要任务栈的名字，默认情况下，所有 **Activity** 所需的任务栈的名字为应用包名. **TaskAffinity** 属性主要和 **singleTask** 启动模式 或者 **allowTaskReparenting** 属性配对使用，而任务栈分为前台任务栈和后台任务栈 ， 后台任务栈中的 **Activity** 位于暂停状态，用户可以通过切换将后台任务栈再次调到前台.

**8**.两种 **Activity** 的启动模式：  
(1)通过 **AndroidMenifest** 为 **Activity** 指定启动模式：  

<font size = 4 color = blue> `android:launchMode="singleTask"` </font>  

(2)通过在 **Intent** 中设置标志位为 **Activity** 指定启动模式：   

<font size = 4 color = blue> `Intent intent = new Intent();` </font>  
<font size = 4 color = blue> `intent.setClass(this,PieActivity.class);` </font>  
<font size = 4 color = blue> `intent.addFlags(intent.FLAG_ACTIVITY_NEW_TASK);` </font>   
<font size = 4 color = blue> `startActivity(intent);` </font>   
                 
优先级上，第二种方式高于第一种，限定范围不一样，第一种无法直接为 **Actvity** 设定 <font size = 4 color = blue>`FLAG_ACTIVITY_CLEAR_TOP` </font> 标识，第二种方式无法为 **Activity** 指定 **singleInstance** 模式.
  