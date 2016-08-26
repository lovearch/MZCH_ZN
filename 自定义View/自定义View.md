# 自定义View
## 自定义控件的优点
Android本身提供了很多的控件。
The Android framework has a large set of View classes for interacting with 
the user and displaying various types of data. But sometimes your app has 
unique needs that aren’t covered by the built-in views. This class shows 
you how to create your own views that are robust and reusable.

## 如何自定义控件
自定义View分为以下四个步骤：
1. Creating a View Class
2. Custom Drawing
3. Marking the View Interactive
4. Optimizing the View

## Creating a View Class
创建具有自定义属性，并且支持Android Studio布局的View，如同内建的View一样。

### 继承自View或者其子类
所有的Android框架中的View有关的类都是继承自View父类的，所以自定义的View首先要继承自View或者继承自View的子类。
并且提供构造函数，包含有两个参数Context和AttributeSet，如下：
``` java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```
### 自定义属性
自定义控件的属性，在XML文件中声明，一般自定义的控件都提供在XML 编辑器中设置属性的值，为了能
在XML 编辑器中直接设置，需要一下几步：

#### 为自定义的View定义 `<declare -styleable>` 
在项目中的resource中添加attrs.xml文件，res/values/attrs.xml，
比如像这样定义：
``` xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```
这里有一个规矩，declare-styleable的名字最好和自定义的View的名字一样，就是和类名一样，
很多编辑器是回默认这个规矩的。
这里面定义了两个属性 showText和labelPosition.
format标签是指定属性的类型，有这些类型：
string,color,demension,integer,enum,reference,float,boolean,fraction,flag;:
在XML 编辑器中需要这样使用：
```xml 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```
xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
是一定要加上的，后面的包路径是项目的package。
如果定义的View类是内部类的时候，custom的声明有变化，比如：PieChart含内部类PieView；
这时使用PieView的属性需要这样定义：com.example.customviews.charting.PieChart$PieView。

### 获取自定义的属性值
自定义的View需要从xml中获取属性值，这些值存储在AttributeSet中，从中解析出需要的属性值。
先调用obtainStyledAttributes()接口，传入AttributeSet对象，获得一个TypeArray类型的数组，
然后遍历TypeArray数组，通过匹配 R.styleable.自定义属性 来取得属性值。最后使用了的TypeArray
需要回收，调用recycle()接口。
示例代码如下：
``` java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

### 属性值加载
当获取到自定义的属性值后，需要将属性值应用到view的显示中，通过调用invidate()
和requestLayout()接口，来重绘view。另外还可能需要定义事件监听，来监听view的变化

## Custom Drawing
onSizeChanged()函数当view大小发生变化时候会调用
onMeasure()函数
onDraw()函数   
### onMeasure函数  
onMeasure传入的widthMeasureSpec和heightMeasureSpec不是一般的尺寸数值，
而是将模式和尺寸组合在一起的数值。
我们需要通过int mode = MeasureSpec.getMode(widthMeasureSpec)得到模式，
用int size = MeasureSpec.getSize(widthMeasureSpec)得到尺寸。

mode共有三种情况，取值分别为
MeasureSpec.UNSPECIFIED, MeasureSpec.EXACTLY, MeasureSpec.AT_MOST。

MeasureSpec.EXACTLY是精确尺寸，
当我们将控件的layout_width或layout_height指定为具体数值时
如andorid:layout_width="50dip"，或者为FILL_PARENT是，都是控件大小已经确定的情况，
都是精确尺寸。

MeasureSpec.AT_MOST是最大尺寸，
当控件的layout_width或layout_height指定为WRAP_CONTENT时，
控件大小一般随着控件的子空间或内容进行变化，此时控件尺寸只要不超过父控件允许的最大尺寸即可。
因此，此时的mode是AT_MOST，size给出了父控件允许的最大尺寸。

MeasureSpec.UNSPECIFIED是未指定尺寸，这种情况不多，一般都是父控件是AdapterView，
通过measure方法传入的模式。

## VIew的交互性

## 忠告
自定义View的谷歌官方文档：https://developer.android.com/training/custom-views/index.html






