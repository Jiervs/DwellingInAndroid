# Review For Android
***
## View 的事件体系
***
### View 的基础知识  

**1**. **View** 是界面层控件的一种抽象，**ViewGroup** 也是继承于 **View** 的，**View** 的位置主要由它四个顶点来决定的：  
**top** : 左上角的纵坐标 .  
**left** : 左上角的横坐标 .  
**right** : 右下角的横坐标 .  
**bottom** : 右下角的纵坐标 .  
这些参数都是相对于 **View** 的父容器而言 ，从 **Android** 3.0 开始 ，**View** 增加了额外的几个参数 ： **x** 、**y** 、 **translationX** 、**translationY**、这几个参数也是相对于父容器的，其中  **x** ， **y**  是  **View** 的左上角的坐标 , **translationX** 、**translationY** 是 **View** 左上角相对于父容器的偏移量.   

**2**. **MontionEvent** 典型的事件类型有如下几种 :   
<font size = 4 color = blue>`ACTION_DOWN` </font> : 手指刚接触屏幕 .   
<font size = 4 color = blue>`ACTION_MOVE` </font> : 手指在屏幕上移动 .   
<font size = 4 color = blue>`ACTION_UP` </font> : 手指从屏幕上松开的一瞬间.   
通过  **MontionEvent** 对象我们可以得到点击事件发生的 **x** 和 **y** 的坐标，**getX()** 和 **getY()** 返回的是相当于当前 **View** 左上角的 **x** 和 **y** 的坐标，而 **getRawX()** 和 **getRawY()** 返回的是相对于手机屏幕左上角的  **x** 和 **y** 的坐标.   

**3**.**TouchSlop** 是系统所能识别出的被认为是**滑动**的**最小距离**，跟设备有关，可以通过 <font size = 4 color = blue>`ViewConfiguration.get(getContext()).getScaledTouchSlop()` </font> 获得.