---
title: Android开发中基础动画技巧的应用
date: 2016-08-17
categories: Android小记
tags: []
---
## Android开发中基础动画技巧的应用

### 一、引言

        我是先入门iOS的移动开发者，提到动画开发，iOS开发者很容易联想到3种方式，UIImageView的帧动画，UIView层的属性动画和CoreAnimation动画。Android中也有3种方式创建基础动画效果，分别为View Animation，Property Animation和Drawable Animation。由于Android开发的固有特点，其在进行动画编程时也支持使用代码和xml配置文件两种方式。本篇博客，将主要向大家介绍这3种创建Android动画方式的使用方法与可以做到的效果。

### 二、View Animation动画的应用

        View Animation又被称为Tweened Animation，其应用于View视图变化的动画过渡效果。View Animation主要分为如下4类：

①.AlphaAnimation：透明度动画

②.RotateAnimation：旋转动画

③.ScaleAnimation:缩放动画

④.TranslateAnimation：位移动画

#### 1.AlphaAnimation的应用

    AlphaAnimation用于当视图透明度发生变化时展示过渡动画，可以渐隐也可以渐现。使用AlphaAnimation创建动画的核心代码如下：

```java
//创建AlphaAnimation动画对象 构造方法中需要传入两个float值 分别是视图动画起始的alpha值与最终的alpha值
AlphaAnimation alphaAnimation = new AlphaAnimation(1,0);
//设置动画执行时间 
alphaAnimation.setDuration(3000);
//调用视图的startAnimation方法来开启动画
animationImageView.startAnimation(alphaAnimation);
```

![](http://static.oschina.net/uploads/space/2016/0816/133103_PrKC_2340880.gif)

#### 2.RotateAnimation的应用

    RotateAnimation用于创建视图的旋转动画。其相比AlphaAnimation要复杂一些，在使用时，除了需要设置其动画的起始角度和最终角度外，还可以设置视图旋转时的参照位置，示例代码如下：

```java
//创建旋转动画对象
RotateAnimation rotateAnimation = new RotateAnimation(0,360, Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
//设置动画时间
rotateAnimation.setDuration(3000);
//开始动画
animationImageView.startAnimation(rotateAnimation);
```

这里使用了RotateAnimation类中最复杂的一个构造方法，其中需要传入6个参数，前两个参数分别为旋转动画的起始角度与终止角度，第3个参数为旋转参照点的x轴相对位置类型，第4个参数为参照点x轴位置，第5个和第6个参数分别为旋转参照点的y轴相对位置类型与y轴相对位置。

    关于参照点的相对位置类型，Animation类中定义了几个常量供开发者选择使用，意义如下：

```java
//绝对定位 以当前窗口做参照
public static final int ABSOLUTE = 0;
//以其父视图做为位置参照
public static final int RELATIVE_TO_PARENT = 2;
//以本身作为位置参照
public static final int RELATIVE_TO_SELF = 1;
```

还有一点需要注意，如果选择的参照类型是RELATIVE\_TO\_SELF，则参照点的位置参数取值范围为0-1之间，代表的是相对于自身的位置比例，如果参照类型是RELATIVE\_TO\_PARENT，则参照点的位置参数取值范围为0-1之间，代表的是相对于父视图的位置比例，如果参照类型是ABSOLUTE，则参照点的位置参数取值为绝对坐标值，例如100，150，其代表了相对窗口视图的坐标位置。例如上面示例代码中，以视图本身为参照物，x、y轴位置都设置为0.5，则旋转动画以视图本身中心为旋转点，如果需要以视图右下角为旋转点，修改代码如下：

```java
RotateAnimation rotateAnimation = new RotateAnimation(0,360, Animation.RELATIVE_TO_SELF,1f,Animation.RELATIVE_TO_SELF,1f);
```

![](http://static.oschina.net/uploads/space/2016/0816/144612_kHnQ_2340880.gif)

#### 3.ScaleAnimation的应用

        ScaleAnimation用于创建放大或者缩小的形变动画，示例代码如下：

```java
//创建缩放动画对象
ScaleAnimation scaleAnimation = new ScaleAnimation(1,2,1,2,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
//设置动画时间
scaleAnimation.setDuration(3000);
//执行动画
animationImageView.startAnimation(scaleAnimation);
```

上面示例代码中前4个参数分别设置缩放动画x轴方向的起始值、最终值和y轴方向的起始值、终止值。需要注意，这里的单位都是比例，1表示原视图尺寸，2表示原视图尺寸的2倍。这个方法后4个参数的意义是确定缩放参照点的位置，和RotateAniamtion构造方法中的参数意义一致。

![](http://static.oschina.net/uploads/space/2016/0816/145733_M8TL_2340880.gif)

#### 4.TranslateAnimation的应用

        TranslateAnimation用于创建位移动画，示例代码如下：

```java
//创建位移动画对象
TranslateAnimation translateAnimation = new TranslateAnimation(Animation.ABSOLUTE,0,Animation.ABSOLUTE,100,Animation.RELATIVE_TO_SELF,0,Animation.RELATIVE_TO_SELF,1);
//设置动画时间
translateAnimation.setDuration(3000);
//执行动画
animationImageView.startAnimation(translateAnimation);
```

上面示例代码中使用的TranslateAnimation构造方法中的8个参数分别代表，起始位置的x轴参照点类型与起始位置的x轴值、终止位置的x轴参照点类型与终止位置的x轴值、起始位置的y轴参照点类型与起始位置的y轴值、终止位置的y轴参照点类型与终止位置的y轴值。

#### 5.Animation类中的通用方法

        上面介绍的4种动画实际上都是Animation类的子类，Animation类中封装了许多动画通用的方法，例如前面使用的设置动画执行时间的方法setDuration就是Animation类的方法，其中更多常用方法列举如下：

```java
//取消正在执行的动画
public void cancel();
//设置动画的执行函数
public void setInterpolator(Interpolator i);
//这只方法设置开始动画的延时值 单位为毫秒
public void setStartOffset(long startOffset);
//设置动画的执行时间
public void setDuration(long durationMillis);
//设置动画特效的最长运行时间
public void restrictDuration(long durationMillis);
//设置动画的执行强度比例 例如放大两倍的动画 这个值如果设置为2 将被放大4倍
public void scaleCurrentDuration(float scale);
//为动画设置一个开始的时间
public void setStartTime(long startTimeMillis);
//设置动画循环模式
/*
只有当动画的循环次数大于1时这个值才有效果，其可以设置为如下常量：
RESTART 每次循环都从头执行
REVERSE 正逆交替执行
*/
public void setRepeatMode(int repeatMode);
//设置循环次数 设置为INFINITE则为无限循环
public void setRepeatCount(int repeatCount);
//设置是否允许填充动画 这个方法设置为true后setFillBefore()与setFillAfter()方法才会生效
public void setFillEnabled(boolean fillEnabled);
//动画结束后 是否以起始位置填充视图
public void setFillBefore(boolean fillBefore);
//动画结束后 是否以结束位置填充视图
public void setFillAfter(boolean fillAfter)
//设置动画执行时在Z轴上的位置
/*
可以设置为如下3中常量
    public static final int ZORDER_BOTTOM = -1; //将动画放在Z轴最下边
    public static final int ZORDER_NORMAL = 0;  //将动画放在Z轴原位置
    public static final int ZORDER_TOP = 1;     //将动画放在Z轴最上边
*/
public void setZAdjustment(int zAdjustment);
//设置动画的执行是否影响到壁纸
public void setDetachWallpaper(boolean detachWallpaper);
//获取动画是否开始执行了
public boolean hasStarted();
//获取动画是否结束执行了
public boolean hasEnded();
```

上面列举的方法中，setInterpolator()方法很有意思，其可以设置动画执行的时间函数，例如是先快后慢还是先慢后快等等，这个方法需要传入一个Interpolator类型的参数，实际上使用时是通过Interpolator的子类来实现的，示例如下：

```java
ranslateAnimation.setInterpolator(new AccelerateDecelerateInterpolator());
```

对于Interpolator参数开发者可以设置的子类及意义列举如下：

AccelerateDecelerateInterpolator：先加速后减速执行

AccelerateInterpolator：加速执行

AnticipateInterpolator：先后退执行一步后正向加速执行(类似弹簧效果)

![](http://static.oschina.net/uploads/space/2016/0816/190457_bwZh_2340880.gif)

 AnticipateOvershootInterpolator：先后退执行一步后加速执行到达极限后再前进一步后再回到极限(弹簧)

![](http://static.oschina.net/uploads/space/2016/0816/182915_U1XI_2340880.gif)

BounceInterpolator：动画执行到结尾后进行阻尼效果

![](http://static.oschina.net/uploads/space/2016/0816/191145_5dtQ_2340880.gif)

CycleInterpolator：以正弦规则循环执行数次动画，这个类来构造时需要传入循环次数，如下：

```java
new CycleInterpolator(3)
```

DecelerateInterpolator：减速执行动画

 FastOutLinearInInterpolator：基于贝塞尔曲线的速率变化

 FastOutSlowInInterpolator：基于贝塞尔曲线的速率变化

 LinearInterpolator：线性匀速执行

LinearOutSlowInInterpolator：基于贝塞尔曲线的速率变化

OvershootInterpolator：执行超出极限后在回退

![](http://static.oschina.net/uploads/space/2016/0816/192211_5GhI_2340880.gif)

PathInterpolator：自定义运动路径

#### 6.实现对Animation动画状态的监听

        Animation类中也定义了一个监听器协议，其中提供了对动画状态进行监听的方法，如下：

```java
public interface AnimationListener {
    //当动画开始执行时触发的方法
    void onAnimationStart(Animation var1);
    //动画执行结束后触发的方法
    void onAnimationEnd(Animation var1);
    //动画重复执行的时候会触发
    void onAnimationRepeat(Animation var1);
}
```

#### 7.使用xml文件配置View Animation

        上面介绍的全部是通过代码来创建View Animation动画，Android也支持使用xml文件来配置View Animation动画。

首先在Android Studio的res目录中创建一个动画文件目录，将其类型选择为anim，如下图所示：

![](http://static.oschina.net/uploads/space/2016/0816/223215_tS6Z_2340880.png)

在创建的目录中创建一个新的xml文件，在其中编写动画代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha android:fromAlpha="1" android:toAlpha="0" android:duration = "3000"/>
</set>
```

在代码中，使用如下代码来加载xml配置的动画：

```java
//加载动画文件
Animation animation = AnimationUtils.loadAnimation(this,R.anim.my_anmi);
//执行动画
animationImageView.startAnimation(animation);
```

#### 8.复合的View Animation

        View Animation也支持进行复合动画的操作，如果使用xml配置复合动画，十分简单，只需要将要要复合的动画都配置进xml文件的set标签中即可，如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha android:fromAlpha="1" android:toAlpha="0" android:duration = "3000"/>
    <scale android:fromXScale="1" android:toXScale="2" android:fromYScale="1" android:toYScale="2" android:duration="3000"/>
</set>
```

![](http://static.oschina.net/uploads/space/2016/0816/224522_H4v6_2340880.gif)

如果要用代码创建复合动画，需要使用到AnimationSet类进行复合，示例如下：

```java
//创建动画集合容器 参数决定容器中所有动画是否共用Interpolator时序函数
AnimationSet set = new AnimationSet(true);
//创建动画
RotateAnimation rotateAnimation = new RotateAnimation(0,360, Animation.RELATIVE_TO_SELF,1f,Animation.RELATIVE_TO_SELF,1f);
ScaleAnimation scaleAnimation = new ScaleAnimation(1,2,1,2,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
scaleAnimation.setDuration(3000);
rotateAnimation.setDuration(3000);
//将动画添加进集合中
set.addAnimation(rotateAnimation);
set.addAnimation(scaleAnimation);
//执行动画
animationImageView.startAnimation(set);
```

![](http://static.oschina.net/uploads/space/2016/0816/230119_e9ow_2340880.gif)

### 三、Property Animation动画的应用

        在前面介绍的View Animation动画体系中，虽然使用起来十分方便，但也有十分多的局限性，例如只能支持透明度，位置，缩放和旋转动画，并且在动画执行时，视图实际上并没有移动，如果需要做动画的是可以用户交互的按钮控件则会带来很多的不便。在Android3.0之后，系统推出了Property Animation动画，这种机制可以将对象任意属性的修改实现过渡动画效果。

#### 1.ObjectAnimator动画的应用

        ObjectAnimator是Property Animation动画体系中最简单易用的一种方式，开发者只需要设置要改变的属性值和一些动画参数即可使用，例如若要实现视图以y方向为轴进行旋转操作，使用 如下代码实现：

```java
//创建属性动画对象
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(animationImageView,"rotationY",0,360,0);
//设置动画时间
objectAnimator.setDuration(3000);
//开始动画
objectAnimator.start();
```

ObjectAnimator类的静态方法ofFloat()用于创建属性动画实例本身，与其类似的方法还有ofInt()与ofObject()。需要注意，这些方法第1个参数为要执行动画的视图，第2个参数为要发生动画改变的属性名，从第3个参数开始后面可以添加任意多个值，这些值代表了属性值改变的路径，例如上面示例代码表示将视图以y方向为轴从0°开始旋转到360°后再旋转回0°。

![](http://static.oschina.net/uploads/space/2016/0817/111354_Wur6_2340880.gif)

    ObjectAnimator类继承自ValueAnimator，ValueAnimator类则更加灵活自由，其可以为自定义类的自定义属性做动画处理，后面会介绍，ValueAnimator类中提供了许多动画配置的方法，常用如下：

```java
//设置动画执行时间
public ValueAnimator setDuration(long duration);
//设置延时执行
public void setStartDelay(long startDelay);
//设置动画循环次数
public void setRepeatCount(int value);
//设置动画循环模式
public void setRepeatMode(int value);
//设置动画执行时序模式
public void setInterpolator(TimeInterpolator value);
//开始执行动画
public void start();
//结束动画
public void end();
//取消动画
public void cancel();
//恢复动画
public void resume();
//暂停动画
public void pause();
```

需要注意，使用ObjectAnimator创建动画的属性必须实现set和get方法。

#### 2.ValueAnimator实现更加灵活的自定义动画

        ObjectAnimator是ValueAnimator的子类，可以理解，ValueAnimator要比ObjectAnimator更加灵活自由，其可以实现任意自定义属性的动画行为。示例代码如下：

```java
//创建ValueAnimator实例
ValueAnimator valueAnimator = new ValueAnimator();
//设置动画的路径值
valueAnimator.setFloatValues(0,200,100,300,0);
//设置动画的执行时间
valueAnimator.setDuration(6000);
//添加动画执行监听
valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
   //这个方法会在每次动画值改变时调用
   @Override
   public void onAnimationUpdate(ValueAnimator valueAnimator) {
       //设置视图的横坐标 
       animationImageView.setX((Float) valueAnimator.getAnimatedValue());
   }
});
//执行动画
valueAnimator.start();
```

如果运行上面代码，可以看到视图在6s内从x坐标点为0的地方平移到200后再次回到100后再次移动到300最终回到原点0。

       上面的示例代码只是演示了ValueAnimator的工作原理，开发者可以在onAnimationUpdate()方法中进行任意属性的修改。仅从上面演示代码并不能体现出ValueAnimator的强大之处，可以通过实现类似抛物线的动画来理解ValueAnimator的灵活之处，示例代码如下：

```java
//创建ValueAnimator实例
final ValueAnimator animator = new ValueAnimator();
//示例进行抛物线动画 让控件从(0,0)点位置移动到x轴为400的位置，y轴方向做自由落体
animator.setObjectValues(new Point(0,0),new Point(400,0));
//设置动画时间
animator.setDuration(4000);
//设置时序为线性函数
animator.setInterpolator(new LinearInterpolator());
//由于抛物线运动在x轴和y轴上的速度变化并不相同 需要自定义枚举器
animator.setEvaluator(new TypeEvaluator<Point>() {
   //这个枚举方法中传入的v值为动画执行的比例 0为初始状态 1为动画执行完成 开发者根据这个值模拟抛物线坐标
   @Override
   public Point evaluate(float v, Point o,Point t1) {
       //创建Point对象 模拟抛物运动
       Point point = new Point();
       point.x = (int)((v*8)*100);
       point.y = (int)((v*60)*(v*60)/4); 
       return point;
   }
});
//监听动画执行
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator valueAnimator) {
        //设置视图位置
        animationImageView.setX(((Point)valueAnimator.getAnimatedValue()).x);
        animationImageView.setY(((Point)valueAnimator.getAnimatedValue()).y);
   }
});
//执行动画
animator.start();
```

![](http://static.oschina.net/uploads/space/2016/0817/131414_Pmyy_2340880.gif)

需要注意，Property Animation与View Animation最大的不同在于View Animation只是展示视图的界面动画，它并没有真正改变视图的属性，而Property Animation是实实在在的改变了发生动画控件的属性。

#### 3.Property Animation动画的监听

        ValueAnimator对象可以使用addListener()方法来添加监听者，接口方法如下：

```java
//动画监听接口
public interface AnimatorListener {
    //动画开始
    void onAnimationStart(Animator var1);
    //动画结束
    void onAnimationEnd(Animator var1);
    //动画取消
    void onAnimationCancel(Animator var1);
    //动画重复
    void onAnimationRepeat(Animator var1);
}
```

#### 4.使用PropertyValuesHolder进行动画复合

        对于Property Animation，开发者可以通过ValueAnimator实现自定义的复合动画，也可以使用PropertyValuesHolder进行属性动画的复合操作，示例如下：

```java
//创建子属性动画  翻转
PropertyValuesHolder holder = PropertyValuesHolder.ofFloat("rotationY",0,360,90);
//创建子属性动画  透明
PropertyValuesHolder holder2 = PropertyValuesHolder.ofFloat("alpha",1,0,1);
//进行动画复合
ObjectAnimator objectAnimation = ObjectAnimator.ofPropertyValuesHolder(animationImageView,holder,holder2);
//执行动画
objectAnimation.setDuration(3000);
objectAnimation.start();
```

![](http://static.oschina.net/uploads/space/2016/0817/133234_li88_2340880.gif)

### 三、Drawable Animation动画的应用

        相比前两种动画模式，Drawable Animation动画要容易的多，其使用一组图像快速切换的原理来实现动画效果。

在Android Studio的drawable文件夹中添加一个animation文件，xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/bird1" android:duration="200" />
    <item android:drawable="@drawable/bird2" android:duration="200" />
    <item android:drawable="@drawable/bird3" android:duration="200" />
    <item android:drawable="@drawable/bird4" android:duration="200" />
    <item android:drawable="@drawable/bird5" android:duration="200" />
    <item android:drawable="@drawable/bird6" android:duration="200" />
    <item android:drawable="@drawable/bird7" android:duration="200" />
    <item android:drawable="@drawable/bird8" android:duration="200" />
</animation-list>
```

将需要展示动画的视图背景设置为这个drawable文件，示例如下：

```xml
 <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@drawable/anmi_draw_list"
        android:id="@+id/animatedImageView"/>
```

在需要开始动画时，调用如下代码即可：

```java
//获取到drawable背景 调用start()方法开始动画
((AnimationDrawable)animationImageView.getBackground()).start();
```

![](http://static.oschina.net/uploads/space/2016/0817/135417_BSQr_2340880.gif)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639
