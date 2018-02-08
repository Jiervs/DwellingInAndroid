# Review For Android
***
## View 的工作原理
***
### ViewRoot 和 DecorView  

**1.** **ViewRoot** 对应于 **ViewRootImpl** 类，他是链接 **WindowManager** 和 **DecorView** 的纽带，**View** 的三大流程均是通过 **ViewRoot** 来完成的 ，在 **ActivityThread** 中，当 **Activity** 对象被创建完毕后，会将 **DecorView** 添加到 **Window** 中，同时会创建 **ViewRootImpl** 对象和 **DecorView** 建立关联，这个过程可参考如下源码：  
<font size = 3 color = blue>`root = new ViewRootImp(view.getContext(),dispaly);` </font>  
<font size = 3 color = blue>`root.setView(view,wparams,panelParentView);` </font>  

**View** 的绘制流程从 **ViewRoot** 的 **performTraversals()** 开始，大致流程如下图：     
<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/performTraversals.png" width = 600 />
 
**measure()** 完成之后，可以通过 **getMeasuredWidth()** 和 **getMeasureHeight()** 方法获取到 **View** 测量后的宽高，**layout()** 过程决定了 **View** 四个顶点的坐标和实际的 **View** 宽高，**draw()** 决定了 **View** 的显示.  

**2.** 理解 **MeasureSpec** : 在很大程度上决定了一个 **View** 的尺寸规格，而父容器也会影响 **View** 的 **MeasureSpec**，**MeasureSpec** 代表一个 **32** 位 **int** 值，高 **2** 位代表 **SpecMode**，低 **30** 位代表 **SpecSize** ，**SpecMode** 是指测量模式， **SpecSize** 是指某种测量模式下的规格大小.  

**3.** **SpecMode** 有三类：  
(1). **UNSPECIFIED** : 父容器不对 **View** 有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量状态.   
(2). **EXACTLY** : 父容器已经检测出 **View** 所需要的精确大小，这个时候 **View** 的最终大小就是 **SpecSize** 所指定的值，对应  **LayoutParams** 中的 **MATCH\_PARENT** 和具体的数值这两种模式.  
(3). **AT\_MOST** :  父容器指定了一个可用大小即 **SpecSize** , **View** 的大小不能大于这个值 ，具体是什么值要看不同 **View** 的具体实现 ， 它对应于 **LayoutParams** 中的 **WRAP\_CONTENT** .  

<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/MeasureSpec.png" width = 600 />  

**4.** **View** 的 **measure** 过程 : 由 **measure()** , 该方法是个 **final** 方法,该方法会调用 **onMeasure()** 方法.   
**ViewGroup** 的 **measure** 过程 : **ViewGroup** 是一个抽象类，没有重写 **onMeasure()** 方法,但是提供了一个叫 **measureChlidren()** 的方法，除了完成自己的 **measure** 过程外，还会遍历调用所有子元素的 **measure()** 方法.  

**5.**一个比较好的习惯是在 **onLayout()** 方法中去获取 **View** 的测量宽/高或者最终宽/高，因为 **View** 的 **measure** 过程和 **Activity** 的生命周期方法不是同步执行的，实际上在基本生命周期方法中均无法正确得到某个 **View** 的宽/高信息，这里提供4种方法来解决 :  
(1). **Activity / View # onWindowFocusChanged**  
**onWindowFocusChanged()** 含义 : **View** 初始化完毕，宽/高已经准备好，该方法会被调用多次，当 **Activity** 的窗口得到焦点和失去焦点时均会被调用一次，如果频繁调用 **onResume()** 和 **onPause()**， **onWindowFocusChanged()** 也会被频繁调用. 典型代码如下：  

    @Override  
    public void onWindowFocusChanged(boolean hasFocus) {  
      super.onWindowFocusChanged(hasFocus);  
        if (hasFocus) {  
          int width = iv_test.getMeasuredWidth();  
          int height = iv_test.getMeasuredHeight();  
        }  
      }  

(2).**view.post(runnable)**  
通过 **post** 可以将一个 **runnable** 投递到消息队列的尾部，然后等待 **Looper** 调用此 **runnable** 的时候，**View** 也已经初始化好了.典型代码如下 :   

    @Override  
    protected void onStart() {  
      super.onStart();  
      iv_test.post(new Runnable() {  
        @Override  
        public void run() {  
          int width = iv_test.getMeasuredWidth();  
          int height = iv_test.getMeasuredHeight();  
        }  
      });  
    }

(3).**ViewTreeObserver**  
当 **View** 树的状态发生改变或者 **View** 树内部的 **View** 的可见性发生改变时，**onGlobaLayout()** 方法将被回调，因此可以获取 **View** 的宽高，伴随 **View** 树的状态改变等，**onGlobalLayout()** 会被多次调用，典型代码如下 :  

    @Override  
    protected void onStart() {  
      super.onStart();  
      ViewTreeObserver observer = iv_test.getViewTreeObserver();  
      observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {  
      @Override  
      public void onGlobalLayout() {  
        iv_test.getViewTreeObserver().removeOnGlobalLayoutListener(this);  
        int width = iv_test.getMeasuredWidth();  
        int height = iv_test.getMeasuredHeight();  
        }  
      });  
    } 

(4).**view.measure(int widthMeasureSpec , int heightMeasureSpec)**  
该方法较为复杂，需分情况讨论，详细分析 见 **《Android开发艺术探索》- 任玉刚 - p192**  

**6.** **View** 的 **layout** 过程 : **Layout** 作用是 **ViewGroup** 用来确定子元素的位置，**layout() ** 确定 **View** 本身的位置，而 **onLayout()** 方法则会确定左右子元素的位置.
源码导读 见 **《Android开发艺术探索》- 任玉刚 - p193 - p197**  

**7.** **View** 的 **draw** 过程 :  **View** 的绘制过程遵循如下几步 :  
(1).绘制背景 **background.draw(canvas)**.  
(2).绘制自己 (**onDraw**).  
(3).绘制children ( **dispatchDraw**).  
(4).绘制装饰 (**onDrawScrollBars**)  






