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


