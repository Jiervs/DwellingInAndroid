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
**ViewGroup** 的 **measure** 过程 : **ViewGroup** 是一个抽象类，没有重写 **onMeasure()** 方法,但是提供了一个叫 **measureChlidren()** 的方法，除了完成自己的 **measure** 过程外，还会遍历调用所有子元素的 measure() 方法




