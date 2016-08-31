# RecycleView 基础
## How to use RecyclerView in Android Projects
### Add dependices
In order to use the RecycleView, first wo should to add the dependencies in **build.gradle**, like the following code.

``` xml
dependencies {    compile fileTree(dir: 'libs', include: ['*.jar'])    testCompile 'junit:junit:4.12'    compile 'com.android.support:appcompat-v7:23.4.0'    compile 'com.android.support:recyclerview-v7:23.4.0'}
```
Then, you can use RecyclerView in class files.

### RecyclerView工作必须的组件
控制显示方式通过布局管理器，LayoutManager
控制Item的间隔，通过ItemDecoration
控制Item的增删动画，通过ItemAnimator
控制点击、长按事件？

必要的几样东西
RecyclerView.Adapter：托管数据集合，为每个Item创建视图；
RecyclerView.ViewHolder：承载Item视图的子视图；
RecyclerView.LayoutManager：负责Item视图的布局；
RecyclerView.ItemDecoration：为每个Item视图添加子视图，在Demo中被用来绘制Divider；
RecyclerView.ItemAnimator：负责添加、删除数据时的动画效果；

LayoutManager
RecyclerView提供了三种内置的LayoutManager:LinearLayoutManager:线性布局,横向或者纵向滑动列表GridLayoutManager:表格布局StaggeredGridLayoutManager:流式布局,例如瀑布流效果.
当然除了上面的三种内部布局之外，我们还可以继承RecyclerView.LayoutManager来实现一个自定义的LayoutManager。


Adapter需要继承自RecyclerView.Adapter，实现三个接口：

public ViewHolder onCreateViewHolder(ViewGroup view, int position)：
在该方法中我们创建一个ViewHolder并返回，ViewHolder必须有一个带有View的构造函数，这个View就是Item的根布局

onBindViewHolder()
处理在Item的控件上显示的数据

getItemCount()
返回Item的个数

另外还需要实现RecyclerView.ViewHolder，


3.分割线  非必须，但是有分割线会好看很多
继承自RecyclerView.Decoration,需要实现三个接口，分别是：
onDraw方法：其绘制将会在每个Item被绘制之前进行；
onDrawOver：在绘制完Item后进行绘制；
getItemOffsets 可以通过outRect.set()为每个Item设置一定的偏移量；

getItemOffsets中为outRect设置的4个方向的值，将被计算进所有decoration的尺寸中，而这个尺寸被计入了RecyclerView每个item view的padding中
在onDraw为divider设置绘制范围，并绘制到canvas上，而这个绘制范围可以超出在 getItemOffsets 中设置的范围，但由于 decoration 是绘制在 child view 的底下，所以并不可见，但是会存在 overdraw
decoration 的 onDraw，child view 的 onDraw，decoration 的 onDrawOver，这三者是依次发生的
onDrawOver 是绘制在最上层的，所以它的绘制位置并不受限制



## ItemDecoration理解

与绘制有关的函数是：
public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state)
在RecyclerView.ItemDecoration废弃了一个接口，是：
public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent)
在源码中，两者之间的关系是这样的

``` java
public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {            getItemOffsets(outRect, ((LayoutParams) view.getLayoutParams()).getViewLayoutPosition(),                    parent);        }
```
所以，只是一层封装而已，但是也提示了一点，就是可以通过
((LayoutParams) view.getLayoutParams()).getViewLayoutPosition()来获取当前View在parent中的位置。
getItemOffsets接口在 Rect getItemDecorInsetsForChild(View child)函数中调用，源码如下：

``` java
Rect getItemDecorInsetsForChild(View child) {        final LayoutParams lp = (LayoutParams) child.getLayoutParams();        if (!lp.mInsetsDirty) {            return lp.mDecorInsets;        }        final Rect insets = lp.mDecorInsets;        insets.set(0, 0, 0, 0);        final int decorCount = mItemDecorations.size();        for (int i = 0; i < decorCount; i++) {            mTempRect.set(0, 0, 0, 0);            mItemDecorations.get(i).getItemOffsets(mTempRect, child, this, mState);            insets.left += mTempRect.left;            insets.top += mTempRect.top;            insets.right += mTempRect.right;            insets.bottom += mTempRect.bottom;        }        lp.mInsetsDirty = false;        return insets;    }
```
分析：

1.lp.mInsetsDirty 这个boolean的标志位，应该是用来标志当前的view的decoration的Rect是否已经计算过.

2.mItemDecorations是一个ArrayList，定义有：

``` java
private final ArrayList<ItemDecoration> mItemDecorations = new ArrayList<>();
```
3.这里会把所有的ItemDecoration的Rect都累加起来，在for循环中可以看得出来.

这个函数会在public void measureChild(View child, int widthUsed, int heightUsed)中被调用。此函数的源码在这里：

``` java
		public void measureChild(View child, int widthUsed, int heightUsed) {            final LayoutParams lp = (LayoutParams) child.getLayoutParams();            final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);            widthUsed += insets.left + insets.right;            heightUsed += insets.top + insets.bottom;            final int widthSpec = getChildMeasureSpec(getWidth(), getWidthMode(),                    getPaddingLeft() + getPaddingRight() + widthUsed, lp.width,                    canScrollHorizontally());            final int heightSpec = getChildMeasureSpec(getHeight(), getHeightMode(),                    getPaddingTop() + getPaddingBottom() + heightUsed, lp.height,                    canScrollVertically());            if (shouldMeasureChild(child, widthSpec, heightSpec, lp)) {                child.measure(widthSpec, heightSpec);            }        }
```
上面这段代码中调用 getChildMeasureSpec 函数的第三个参数就是 child view 的 padding，而这个参数就把 insets 的值算进去了。那么现在就可以确认了，getItemOffsets 中为 outRect 设置的4个方向的值，将被计算进所有 decoration 的尺寸中，而这个尺寸，被计入了 RecyclerView 每个 item view 的 padding 中。

![image](http://blog.piasy.com/img/201603/getItemOffsets_test1.png)
可以看到，当 left, top, right, bottom 全部设置为50时，RecyclerView 的每个 item view 各个方向的 padding 都增加了，对比各种情况，确实 getItemOffsets 中为 outRect 设置的值都将被计入 RecyclerView 每个 item view 的 padding 中。

## ItemAnimator



