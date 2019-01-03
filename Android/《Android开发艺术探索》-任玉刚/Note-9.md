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

**1.**属性动画中有 **ValueAnimator** 、 **ObjectAnimator** 、**AnimatorSet** 等类 , **ValueAnimator** 继承  **ObjectAnimator** ，**AnimatorSet** 是动画合集。

**2.** 使用案例:  
（1）改变一个对象（myObject）的 translationY 属性，让其沿着 Y 轴向上平移一段距离:  

	ObjectAnimator.ofFloat(myObject,"translationY",myObject.getHeight()).start();
	  
（2）改变一个对象的背景颜色属性，典型的情形是改变View的背景色，下面的动画可以让背景色在3秒内实现从 0xFFFF8080 到 0xFF8080FF 的渐变，动画会无限循环而且会有反转的效果:   

	ValueAnimator colorAnim = 
		ObjectAnimator.ofInt(this,"backgroundColor", 0xFFFF8080, 0xFF8080FF);
	colorAnim.setDuration(3000);
	colorAnim.setEvaluator(new ArgbEvalutor());
	colorAnim.setRepeatCount(ValueAnimator.INFNITE);
	colorAnim.start();

属性动画也可以通过 XML来定义，属性动画需要定义在res/animator/目录下。（具体属性含义：见 《Android开发艺术探索》- 任玉刚 - p278 ），使用XML属性动画资源：  

	AnimatorSet set = 
		AnimatorInflater.loadAnimator(myContext,R.animator.property_animator);
	set.setTarget(mButton);
	set.start();
	
**3.** 理解插值器 和 估值器	
下图表示一个匀速动画，采用了线性插值器和整型估值算法，在40ms内，View的x属性实现从0到40的变换：   

<img src = "https://raw.githubusercontent.com/Jiervs/RepositoryResource/master/Dwelling-in-the-past/principle_of_interpolator.png" width = 500 /> 

由于动画默认刷新率为10ms/帧，该动画5帧进行，如第三帧（x=20,t=20ms）,当时间 t=20ms ，时间流逝的百分比是 0.5 （20/40 =0.5），时间过了一半，x的改变是多少呢？这个就是有插值器和估值算法来确定的。如线性插值器，x变化也是一半。线性插值器的源码：  

	/**
	 * An interpolator where the rate of change is constant
	 */
	@HasNativeInterpolator
	public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {
	
	    public LinearInterpolator() {
	    }
	
	    public LinearInterpolator(Context context, AttributeSet attrs) {
	    }
	
	    public float getInterpolation(float input) {
	        return input;
	    }
	
	    /** @hide */
	    @Override
	    public long createNativeInterpolator() {
	        return NativeInterpolatorFactoryHelper.createLinearInterpolator();
	    }
	}
	 
在 `getInterpolation(float input)` 方法中，输入和返回时一样的因此意味着x的改变时0.5，这个时候插值器的工作就完成了，具体x变成什么值，这个就需要估值算法来确定，估值算法的源码:  

	/**
	 * This evaluator can be used to perform type interpolation between <code>int</code> values.
	 */
	public class IntEvaluator implements TypeEvaluator<Integer> {
	
	    public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
	        int startInt = startValue;
	        return (int)(startInt + fraction * (endValue - startInt));
	    }
	} 
	
`evaluate(float fraction, Integer startValue, Integer endValue)` 的三个参数代表估值小数，开始值，结束值，所以估值算法返回给 x=20，t=20ms。

属性动画要求对象属性有set()方法，get()方法可选插值器和估值算法我们可以自定义，具体一点插值器需要实现 Interpolator 或者 TimeInterpolator,自定义估值算法需要实现 TypeEvaluator，对其他类型（非int,float,Color）做动画，就需要自定义类型估值算法。  

属性动画的监听器 ：AnimatorListener ，AnimatorUpdateListener。

**4.** 对任意属性做动画 及 工作原理（见 《Android开发艺术探索》- 任玉刚 - p282 - p293）
	

	
		 
	
  

   
