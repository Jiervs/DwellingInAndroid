# Review For Android

***
## Window 和 WindowManager
 
**Window** 是一个抽象类,具体实现是 **PhoneWindow** ,**Window** 通过 **WindowManager** 可以创建 ,**WindowManager** 也是外界访问 **Window** 的入口 , **Window** 的具体实现位于 **WindowManagerService** 中 , **WindowManagerService** 和 **WindowManager** 的交互是一个 **IPC** 过程 ,**Android** 中所有视图都是通过 **Window** 实现的, **Window** 是 **View** 的直接管理者。 

***
### Window 简述

通过 **WindowManager** 添加 **Window** 过程（将一个Button添加到屏幕坐标位置为（100，300）的位置上）: 

```java
WindowManager manager = getWindowManager();
        Button button = new Button(this);
        button.setText("button");
        WindowManager.LayoutParams params =
                new WindowManager.LayoutParams(WindowManager.LayoutParams.WRAP_CONTENT,
                WindowManager.LayoutParams.WRAP_CONTENT,0,0,PixelFormat.TRANSPARENT);
        params.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL;
        params.gravity = Gravity.START | Gravity.TOP;
        params.x = 100;
        params.y = 300;
        manager.addView(button,params);

```	        
**WindowManager.LayoutParams** 中的 **flags** 和 **type** 参数比较重要 , **Flags** 参数表述 **Window** 的属性 , 通过这些控制 **Window** 的现实特性， **Type** 参数表示 **Window** 的类型，三种类型：1.应用 **Window** (Activity) 。 2.子 **Window** (dialog) 。3.系统 **Window** (toast,系统状态栏)。（见 《Android开发艺术探索》- 任玉刚 - p295），系统级别的 **window** 需要申明权限 `<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`  

**WindowManager** 继承 **ViewManager** :

```java
public interface ViewManager{
   	public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```
即 **添加View** ， **更新View** ， **删除View** ，如果需要删除 **Window** 只需删除里面的 **View** 。

***
## Window 的内部机制 

**Window** 是一个抽象概念，通过 **ViewRootImp** 和 **View** 建立联系，所以 **Window** 是以 **View** 的形式存在 ，为了分析 Window 的内部机制，这里从 Window 的添加、更新、删除说起。 

### Window 的添加过程 
**WindowManager** 的实现类 是 **WindowManagerImpl**，对于其三大操作的实现本质上是交给 **WindowManagerGlobal** 来处理。**WindowManagerGlobal** 的 **addView** 方法主要分为如下几步：

1.检查参数是否合法，如果是子 **Window** 那么还需要调整一些布局参数 

```java
if (view == null) {
    throw new IllegalArgumentException("view must not be null");
}
if (display == null) {
    throw new IllegalArgumentException("display must not be null");
}
if (!(params instanceof WindowManager.LayoutParams)) {
    throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
}
	
final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
if (parentWindow != null) {
    parentWindow.adjustLayoutParamsForSubWindow(wparams);
}
```	        

2.创建 **ViewRootImpl** 并将 **View** 添加到列表中  
在 **WindowManagerGlobal** 内部有如下几个列表比较重要: 

```java
private final ArrayList<View> mViews = new ArrayList<View>();
private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
private final ArrayList<WindowManager.LayoutParams> mParams =
        new ArrayList<WindowManager.LayoutParams>();
private final ArraySet<View> mDyingViews = new ArraySet<View>();  
```
**mViews** 存储的是所有 **Windows** 所对应的 **View**  
**mRoots** 存储的是是所有 **Windows** 所对应的 **ViewRootImpl**  
**mParams** 存储的是所有 **Windows** 所对应的布局参数  
**mDyingViews** 是存储了那些正在被删除的 **View** 对象，或者说是那些已经调用 removeView()但是删除操作还未完成的 **Window** 对象。  

在 **addView** 中通过如下方式将 Window 的一系列对象添加到列表中:  

```java
root = new ViewRootImpl(view.getContext(), display);
view.setLayoutParams(wparams);
	
mViews.add(view);
mRoots.add(root);
mParams.add(wparams);
```	  

3.通过 **ViewRootImpl** 来更新界面并完成 **Window** 的添加过程  
这个步骤由 **ViewRootImpl** 的 **setViews()** 来完成 ， **View** 的绘制过程是由 **ViewRootImpl** 来完成的，在 **setView()** 内部通过 **requestLayout()** 来完成异步刷新请求。接着会通过 **WindowSession** 最终来完成 **Window** 的添加过程。因为涉及到 **mWindowSession** （是一个 **Binder** 对象），所以 Window 的添加过程是一次 **IPC** 调用。

**Window** 的删除，更新同添加类似分析（见 《Android开发艺术探索》- 任玉刚 - p301 - p304）

### Window 的创建过程 
