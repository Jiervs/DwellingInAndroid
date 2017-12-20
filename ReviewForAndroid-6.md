# Review For Android
***
## 自定义View
***
### 自定义 View 的分类

**1.**继承 **View** 重写 **onDraw()**  
这种方法主要用于实现一些不规则的效果，采用这种方式需要自己支持 **wrap_content**，并且 **padding** 也需要自己处理.  

**2.**继承 **ViewGroup** 派生特殊的 **Layout**  
这种方法主要用于实现自定义的布局，需要合适得处理 **ViewGroup** 的测量，布局这两个过程 ， 并同时处理子元素的测量和布局过程.  

**3.**继承特定的 **View** （比如 **TextView**） 
这种方法比较常见，一般适用于扩展某种已有的 **View** 的功能，不需要自己支持 **wrap_content** 、**padding** 等.   

**4.** 继承特定的 **ViewGroup** （比如 **LinearLayout**） 
采用这种方法不需要自己处理 **ViewGroup** 的测量和布局这两个过程，和方法2的区别在于 方法2 更接近 **View** 的底层  


***
### 自定义 View 须知  

**1.**让 **View** 支持 **wrap_content**  
直接继承 View 或 ViewGroup 的控件 ，如果不在 onMeasure 中对 **wrap_content** 做特殊处理，那么当外界在布局中使用 **wrap_content** 无法达到预期效果.  

**2.** 如果有必要，让你的 **View** 支持 **padding**  
直接继承 **View** 的控件，需要在 **draw()** 中处理 **padding** ，直接继承 **ViewGroup** 的控件需要在 **onMeasure()** 和 **onLayout()** 中考虑 **padding** 和子元素的 **margin** 对其造成的影响.  

**3.** 尽量不要在 **View** 中使用 **Handler**  

**4.** **View** 中如果有线程或者动画，需要即时停止，参考 **View # onDetachedFromWindow**  
如果有线程或者动画需要停止时，那么 **onDetachedFromWindow()** 是一个很好的时机.  

**5.** **View** 带有滑动嵌套情形时 ， 需要处理好滑动冲突 

***
### 自定义 View 示例   
见 **《Android开发艺术探索》- 任玉刚 - p202 - p217**