# Review For Android
***
## Android动画
***
### View 动画
**view** 动画的种类：  

| 名称| 标签|子类|效果|
|:---|:---:|:-----|:-----|
|平移动画|\<translate\>|TranslateAnimation|移动View|
|缩放动画|\<scale\>|ScaleAnimation|放大或缩小View|
|旋转动画|\<rotate\>|RotateAnimation|旋转View|
|透明度动画|\<alpha\>|AlphaAnimation|改变View的透明度| 

该四中动画可以通过 **XMl** 来定义，也可以通过代码来动态创建，**XML** 格式的动画可读性更好，使用 **View** 动画先创建动画的 **XML** 文件，文件路径为：**res／anim／filename.xml**, **View**动画的描述文件是有固定的语法： 

<img src = "https://raw.githubusercontent.com/Jiervs/RepsitoryResource/master/Dwelling-in-the-past/ViewAnimation.png" width = 500 /> 

具体属性含义：见 **《Android开发艺术探索》- 任玉刚 - p267 - p270** 

使用动画：  
	
	Button button = (Button)findViewById(R.id.button);
	Animation animation = AnimationUtils.loadAnimation(this,R.anim.animation);  
	button.startAnimation(animation); 
	
除了在 **XML** 中定义动画外，还可以通过代码来应用动画。例如： 
	
	AlphaAnimation alphaAnimation = new AlphaAnimation(0,1);
	alphaAnimation.setDuration(300);
	button.startAnimation(alphaAnimation);
	
帧动画是顺序播放一组预先定义好的图片，使用比较简单，比较容易引起 **OOM** 。 

**View 动画的特殊使用场景**：  
**1.** **LayoutAnimation**作用于**ViewGroup** :  
为**ViewGroup**指定一个动画，这样当它的子元素出场时都会具有这种动画效果，例如这种效果常常被用在 **ListView**上，它的每个**Item**都是意一定的动画形式出现，它使用的就是**LayoutAnimation** ，本质也是一个**View**动画，为了给**ViewGroup** 的子元素加上出场效果，遵循如下几个步骤：  
（1）.定义**LayoutAnimation**文件：
 	
	//res/anim/anim_layout.xml 	
	<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
	android:delay="0.5"
	android:animationOrder="normal"
	android:animation="@anim/anim_item"/> 
	
（2）.为子元素指定具体的入场动画，如下所示：
	
	//res/anim/anim_item.xml
	<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:shareInterpolator="true">
    
    <alpha android:fromAlpha="0.0"
        android:toAlpha="1.0"/>
    
    <translate android:fromXDelta="500"
        android:toXDelta="0"/>
    </set>
  
（3）.为**ViewGroup**指定`android:layoutAnimation`属性：`android:layoutAnimation="@anim/anim_layout"`。对于**ListView**来说，这样**ListView**的**item**就具有出场动画了，这种方式适用于所有的**ViewGroup**，除了在**XML**中指定**LayoutAnimation**外，还可以**LayoutAnimationController**来实现：
 
	ListView lv = (ListView)findViewById(R.id.list);
	Animation animation = AnimationUtils.loadAnimation(this,R.anim.anim_item);
	LayoutAnimationController controller = new LayoutAnimationController（animation);
	controller.setDelay(0.5f);
	controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
	lv.setLayoutAnimation(controller);
	
	
**2.** **Activity**的切换效果:  
**Activity** 有默认的切换效果，但是可以自定义效果，主要用到`overridePendingTransition(int enterAnim,int exitAnim)`这个方法，这个方法必须在`startActivity(Intent)`或者`finish()`之后被调用才能生效，上述方法参数是动画资源id。

***
### 属性动画

**1.**
