# Review For Android

***
## Window 和 WindowManager
 
**Window** 是一个抽象类,具体实现是 **PhoneWindow** ,**Window** 通过 **WindowManager** 可以创建 ,**WindowManager** 也是外界访问 **Window** 的入口 , **Window** 的具体实现位于 **WindowManagerService** 中 , **WindowManagerService** 和 **WindowManager** 的交互是一个 **IPC** 过程 ,**Android** 中所有视图都是通过 **Window** 实现的, **Window** 是 **View** 的直接管理者。 

***
### Window 简述

通过 **WindowManager** 添加 **Window** 过程（将一个Button添加到屏幕坐标位置为（100，300）的位置上）: 
 
			
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
		        
	
**Flags** 参数表述 **Window** 的属性 , 通过这些控制 **Window** 的现实特性， **Type** 参数表示 **Window** 的类型，三种类型：1.应用 **Window** (Activity) 。 2.子 **Window** (dialog) 。3.系统 **Window** (toast,系统状态栏)。（见 《Android开发艺术探索》- 任玉刚 - p295），系统级别的 **window** 需要申明权限 `<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`  

**WindowManager** 继承 **ViewManager** :
	
	public interface ViewManager{
	   	public void addView(View view, ViewGroup.LayoutParams params);
	    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
	    public void removeView(View view);
	}
即 **添加View** ， **更新View** ， **删除View** ，如果需要删除 **Window** 只需删除里面的 **View** 。

***
## Window 的内部机制 

**Window** 是一个抽象概念，通过 **ViewRootImp** 和 **View** 建立联系，所以 **Window** 是以 **View** 的形式存在 ，为了分析 Window 的内部机制，这里从 Window 的添加、更新、删除说起。 

### Window 的添加过程 
**WindowManager** 的实现类 是 **WindowManagerImpl**，对于其三大操作的实现本质上是交给 **WindowManagerGlobal** 来处理。**WindowManagerGlobal** 的 **addView** 方法主要分为如下几步：

1.检查参数是否合法，如果是子 **Window** 那么还需要调整一些布局参数
