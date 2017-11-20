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

<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/View.png" width = 300 />

这些参数都是相对于 **View** 的父容器而言 ，从 **Android** 3.0 开始 ，**View** 增加了额外的几个参数 ： **x** 、**y** 、 **translationX** 、**translationY**、这几个参数也是相对于父容器的，其中  **x** ， **y**  是  **View** 的左上角的坐标 , **translationX** 、**translationY** 是 **View** 左上角相对于父容器的偏移量.  

**2**. **MontionEvent** 典型的事件类型有如下几种 :   
<font size = 4 color = blue>`ACTION_DOWN` </font> : 手指刚接触屏幕 .   
<font size = 4 color = blue>`ACTION_MOVE` </font> : 手指在屏幕上移动 .   
<font size = 4 color = blue>`ACTION_UP` </font> : 手指从屏幕上松开的一瞬间.   
通过  **MontionEvent** 对象我们可以得到点击事件发生的 **x** 和 **y** 的坐标，**getX()** 和 **getY()** 返回的是相当于当前 **View** 左上角的 **x** 和 **y** 的坐标，而 **getRawX()** 和 **getRawY()** 返回的是相对于手机屏幕左上角的  **x** 和 **y** 的坐标.   

**3**.**TouchSlop** 是系统所能识别出的被认为是**滑动**的**最小距离**，跟设备有关，可以通过 <font size = 4 color = blue>`ViewConfiguration.get(getContext()).getScaledTouchSlop();` </font> 获得. 

**4**.**VelocityTracker** : 用于追踪手指在滑动过程中的速度 , 使用过程 :   
首先在 **View** 的 **onTouchEvent** 方法中追踪当前单击事件的速度 :   
<font size = 4 color = blue>`VelocityTracker tracker = VelocityTracker.obtain();`</font>   
<font size = 4 color = blue>`tracker.addMovement(event);`</font>   

接着当我们想知道滑动速度时候，可以通过以下方式获得 :    
<font size = 4 color = blue>`tracker.computeCurrentVelocity(1000)`</font>   
<font size = 4 color = blue>`int xVelocity = (int) tracker.getXVelocity();`</font>    
<font size = 4 color = blue>`int yVelocity = (int) tracker.getYVelocity();`</font>   
必须先调用  **VelocityTracker.obtain()** 方法，参数是时间间隔，这里是1000ms , 而这里的速度是指 一段时间内手指所滑过的像素值 .   

最后当不需要使用它的时候，需要调用 clear() 方法来重置并回收内存 :  
<font size = 4 color = blue>`tracker.clear();`</font>  
<font size = 4 color = blue>`tracker.recycle();`</font> 

**5**. **GestureDetecor** : 手势检测，用于辅助检测用户的 **单击** ，**滑动**，**长按**，**双击** 等行为，使用过程如下 :  
首先 , 创建一个 **GestureDetecor** 对象并实现 **OnGestureListener** 接口 , 根据需要我们还可以实现 **OnDoubleTapListener** 从而能够监听双击行为 :  
<font size = 4 color = blue>`GestureDetector detector = new GestureDetector(this);`</font>  
<font size = 4 color = blue>`detector.setIsLongpressEnabled(false);`</font>  //解决长按屏幕后无法拖动的现象  

接着 接管 目标 **View** 的 **onTouchEvent()** 方法，在监听 **View** 的 **onTouchEvent()** 方法中添加如下实现 :   
<font size = 4 color = blue>`boolean  consume = detector.onTouchEvent(event);`</font>  
<font size = 4 color = blue>`return consume;`</font>  

做完上面两步，就可以有选择地实现 **OnGestureListener** 和 **OnDoubleTapListener** 中的方法了，方法含义如下图所示： 

<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/OnGestureListener%EF%BC%8COnDoubleTapListener(1).png" width = 700 />
<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/OnGestureListener%EF%BC%8COnDoubleTapListener(2).png" width = 700 />   

**6**. **Scroller** : 弹性滑动对象 ， 使用 **View** 的 **scrollTo** / **scrollBy** 方法来进行滑动时，其过程是瞬间完成的，用户体验不好，这个时候可以用 **Scroller** 来实现有过度效果的滑动，**Scroller** 本身无法让 **View** 弹性滑动，它需要和 **View** 的 **computeScroll()** 配合使用才能共同完成这个功能 .   

***
### View 的滑动   

**1**. **View** 的滑动一般使用三种方法 :   
1.通过 **View** 本身提供的 **scrollTo()** 和 **scrollBy()** 方法实现滑动 .   
2.第二种是通过动画给 **View** 施加平移效果来实现滑动 .   
3.第三种是通过改变 **View** 的 **LayoutParam** 使得 **View** 重新布局从而实现滑动 .  

**2**. 使用 **scrollTo()** 和 **scrollBy()** : **scrollBy()** 本质上调用了 **scrollTo()**，**scrollTo()**是实现了基于所传递参数的绝对滑动 ， **scrollBy()** 是相对滑动，**scrollTo()** 和 **scrollBy()** 只能改变 **View 内容**的位置而不能改变 **View** 在布局中的位置，关于 **View** 内部两个属性 **mScrollX** 和 **mScrollY** 的改变规则请看下图 ：    
<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/mScrollX_mScrollY.png" width = 450 />  
 
**3**. **使用动画** : 主要是操作 View 的 **translationX** 和 **translationY**  属性，既可以采用传统的 **View** 动画，也可以采用属性动画，**View** 动画改变的只是一种影像， **View** 的位置参数，包括宽高都不会改变，而属性动画是真实改变 **View** 的位置，但是在3.0以上才能使用（有兼容库）.   

**4**.  改变布局参数，比如使一个 **Button** 向左平移100px :    
<font size = 4 color = blue>`ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams)button.getLayoutParams();`</font>     
<font size = 4 color = blue>`params.leftMargin += 100`</font>   
<font size = 4 color = blue>`button.requestLayout();`</font>   
<font size = 4 color = blue>`// 或者 button.setLayoutParams(params);`</font>   

**5**. 各种滑动方式的对比 : 1. **scrollTo(),scrollBy()** 操作简单，适合对 **View** 内容滑动 ;  2.动画 : 操作简单 , 主要适用于没有交互的 **View** 和实现复杂的动画效果 ;  3. 改变布局参数 : 操作稍微复杂，适用于有交互的 **View**.   

**6**.弹性滑动 : 