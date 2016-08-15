# OpenGL矩阵变换
在OpenGl中矩阵分为三种：
1. 视图矩阵 View Matrix
2. 模型矩阵 Modle Matrix
3. 投射矩阵 Projection Matrix
这三个矩阵在实现3D变换中起着很重要的作用，下面针对原文进行讲解。

## OpenGL 中的3D矩阵变化
### OverView概览
Overview
Geometric data such as vertex positions and normal vectors 
are transformed via Vertex Operation and Primitive Assembly operation 
in OpenGL pipeline before raterization process.
类似于常规向量和几何顶点等数据在Opengl的管道中通过定点变换和原始组装变换，在光栅化之前
（光栅化 是什么意思， 有待进一步查究）
具体过程如下：
![几何数据处理过程](http://www.songho.ca/opengl/files/gl_transform02.png)
* Object Coordinates
物体对象的原始坐标系(也可以成为本地坐标系)，它初始化了物体对象的坐标和方向等信息，是
必须在其他的变换之前做的。这个坐标系可以通过glRotatef(), glTranslatef(), glScalef()
这些函数进行变换。

* Eye Coordinates
将 物体的坐标系和ModleViewMatrix相乘，得到EyeCoordinates，也就是观察坐标系。
物体通过GL_MODELVIEW矩阵转化到观察坐标系（eye coordinates）,
GL_MODELVIEW matrix is a combination of Model and View matrices 
(Mview * Mmodel). OpenGl中 GL_MODELVIEW是Modle Matrix 和View Matrix的乘积。
Model transform is to convert from object space to world space. 
And, View transform is to convert from world space to eye space.
Modle Matrix是将物体从物体的坐标系转化为世界坐标系。View Matrix是将物体坐标从
世界坐标转化为观察坐标。
![GL_MODELVIEW](http://www.songho.ca/opengl/files/gl_transform07.png)
Note that there is no separate camera (view) matrix in OpenGL. 
Therefore, in order to simulate transforming the camera or view, 
the scene (3D objects and lights) must be transformed with 
the inverse of the view transformation. 
In other words, OpenGL defines that the camera is always 
located at (0, 0, 0) and facing to -Z axis in the eye space coordinates, 
and cannot be transformed. See more details of GL_MODELVIEW matrix in 
[ModelView Matrix](http://www.songho.ca/opengl/gl_transform.html#modelview).
OpenGl世界中没有单独的相机或者视图矩阵，因此为了模拟相机或者视图的变换，3D场景
必须通过和视图转换相反的变换。所以在OpenGL中总是将相机位置定位在(0,0,0)，并且朝向-Z轴
的方向。普通的向量从Object Coordinates转换到Eye Coordinates与定点坐标转换不一样，
具体是<br/>
![normal vectors](http://www.songho.ca/opengl/files/gl_normaltransform01.png)
乘以MV矩阵的逆矩阵的转置

* Clip Coordinates
视图坐标系乘以 Projection Matrix转换为 clip coordinates 成为裁剪坐标系。
Projection Matrix称为投影矩阵，它决定着最终的数据是如何投射到屏幕上的。
See more details of GL_PROJECTION matrix in [Projection Matrix](http://www.songho.ca/opengl/gl_transform.html#projection).
![裁剪坐标系](http://www.songho.ca/opengl/files/gl_transform08.png)

* Normalized Device Coordinates (NDC)
它是将 Clip Coordinates坐标系初一 w 得到的。
It is called perspective division. 
It is more like window (screen) coordinates, 
but has not been translated and scaled to screen pixels yet. 
The range of values is now normalized from -1 to 1 in all 3 axes.
这部分做的是将坐标标准化，所得的坐标值在[-1, 1]区间内，是在映射到屏幕之前所做的工作。
w分量是一个权值。标准化类似坐标的归一化。
![裁剪坐标系](http://www.songho.ca/opengl/files/gl_transform12.png)
将剪切面坐标系除以w所得[关于w的讨论可见此处](http://www.songho.ca/math/homogeneous/homogeneous.html)

* Window Coordinates (Screen Coordinates)
It is yielded by applying normalized device coordinates (NDC) 
to viewport transformation. 
The NDC are scaled and translated in order to fit into 
the rendering screen. The window coordinates finally are passed to 
the raterization process of OpenGL pipeline to become a fragment. 
glViewport() command is used to define the rectangle of the rendering 
area where the final image is mapped. 
And, glDepthRange() is used to determine the z value of the window 
coordinates. The window coordinates are computed with the given 
parameters of the above 2 functions; 
将标准化设备坐标系(NDC)应用于视口转换。
NDC将缩小和平移以便适应屏幕的透视。
窗口坐标系最终传递给OpenGL的管道处理变成了fragment。
glViewPort()函数用来定义最终图片映射的投影区域。
同样，glDepthRange()用来决定窗口坐标系的z坐标。
窗口坐标系由下面两个方法给出的参数计算出来. <br/>
![](http://www.songho.ca/opengl/files/gl_transform13.png)
glViewport(x, y, w, h); 
这个函数是用来设置viewport，也就是视口的，就是投影到的那个屏幕。默认这个viewport的法向
是Z轴，所以只有x和y坐标，wh是width和wheight，是视口的长宽。
glDepthRange(n, f); 设置远近的，就是我们看到的东西都只能在远近这个区间[near,far],
glViewPort是设置x轴和y轴的区间（就是最后的映射区间），glDepthRange是设置远近的区间的。
glDepthRange函数是用来调整Z轴的位置的，也就是说调整远近的，就是我们相机看到的远近。
![](http://www.songho.ca/opengl/files/gl_transform13.png)
The viewport transform formula is simply acquired by the linear 
relationship between NDC and the window coordinates;
![](http://www.songho.ca/opengl/files/gl_transform14.png)
上面的两个公式，不是很容易理解的，但是也能理解。

## OpenGL Transformation Matrix
### Project Matrix
这部分的推倒很长，见补充知识部分投影矩阵。

### Textture Matrix
未完待续




### 补充知识
### 投影矩阵
#### 透视投影
概述 
显示器是2d的3d场景需要转换为2d图像才能显示在屏幕上。
投影矩阵（GL_PROJECTION）用于完成这个工作。
投影矩阵将观察坐标（eye coordinates）转换成裁剪坐标（clip coordinates）。
然后，裁剪坐标被除以w，转换为规范化的设备坐标（NDC）。 
需要记住的一点是，裁剪操作和规范化都由投影矩阵（GL_PROJECTION）完成。
下面介绍如何用6个参数（left，right，bottom，top，near，far）构建投影矩阵。 
裁剪（clipping）操作是在裁剪坐标上进行的，安排在透视除法执行之前。
裁剪坐标xc，yc，zc同wc比较，若每个分量都落在（-wc，wc）外，那么此坐标将被裁剪掉。
如下图所示：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵1.png?raw=true)

在透视投影中，3d场景中的点（观察坐标）从平截头体中映射到正方体（NDC）中；
x坐标从[l，r]映射到[-1，1]，
y坐标从[b，t]映射到[-1，1]，
z坐标从[n，f]映射到[-1，1]。 
注意到，观察坐标系是右手系，规范设备坐标系是左手系。
这就有，在观察坐标系中，摄像机朝向沿着-z，而在NDC中，方向沿着z。
由于glFrustum（）只接受正参数，所以构造投影矩阵的时候要变号。 
openGL中，3d场景中，观察坐标系下的点被投影到近投影面。
下图展示了观察坐标系点（xe，ye，ze）投影到近投影面上的点（xp，yp，zp）。

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵2.png?raw=true)

从Top View of Projection看，xe投影到xp，根据等比性质：
(这里是怎么看的呢，看图中的坐标轴的颜色，Top View of Projection是眼睛从下面往上面看的，
在上图中就是从平头锥形的下底面往上面看，途中的箭头； 那Silde View就是从右侧面看过去，
这样就只能看到三个坐标轴中的两个了，所以对照坐标轴，就能明白下面的图是怎么来的了)。

接着，按照相似三角形的理论，得到了以下的计算：（注意坐标轴的正负性）

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵3.png?raw=true)

第一个公式有一个负号，是因为它在Z负轴上面。         

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵4.png?raw=true)

注意到，xp和yp依赖于-ze，这一点要引起重视。
在观察坐标被投影矩阵转换为裁剪坐标后，
裁剪坐标仍然是同质坐标。在规范化阶段执行透视除法变为规范设备坐标（NDC）。

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵5.png?raw=true)

因此，可以将wc的值定为-ze。投影矩阵最后一行为（0，0，-1，0）

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵6.png?raw=true)

下一步，将xp，yp映射到xn，yn，此为线性映射[l，r]=>[-1，1]，[b，t]=>[-1，1]：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵7.png?raw=true)

这一段的意思是 原来我们的投影点是在left right top bottom组成的矩形中的，投影点的
坐标上面计算出来了，现在这个投影面转化为边长为1的正方形，通过线性映射来转换， 其中的
beta分量只是常数分量(这个分量出来得没有任何说明，姑且这样认为吧)。线性投影针对坐标系
其实和一次函数y=kx+b差不多。
这里有一个变换，将

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵8.png?raw=true) 

r, 1带入计算beta。
这里可以这样理解：将区间[l, r]映射到[-1, 1]上面，利用一次函数y=kx + b来计算。
图中说的替代其实是 当 xp = r时，xn = 1，同样当 xp = l xn = -1；是带入边界条件来
计算。可以带入(l, -1)得到的beta一样。
同理，计算y轴的变换，如图：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵9.png?raw=true)。

然后带入上面计算的xp yp，得到如下：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵10.png?raw=true)。

然后提取出矩阵的表达形式，得到如下，将wc赋值为-ze，是为了方便起见

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵11.png?raw=true)

图中矩阵只有三行的出来了，有一行没有计算，接下来就是计算这一行的值：
前面 yn = yc / wc, xn = xc / wc，所以依然有 zn = zc / wc
wc = - ze, 且zc和xe ye没有关系，所以设有下面：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵12.png?raw=true)

这里不知道为什么，但是这样计算：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵13.png?raw=true)

对应关系： (-n, -1) and (-f, 1)，这里的意思是 near是为-1 far为1，映射关系，但是

此处并不是线性映射

n表示 near   f 表示 far 是远近的东西，

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵14.png?raw=true)

下面是普通的换算：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵15.png?raw=true)

将width height属性对应到 right left  top bottom上去：

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵16.png?raw=true)

一张图展示远近的非线性引起的精度问题

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵17.png?raw=true)

以上讲的都是透视投影的计算过程。下面讲的是正交投影

#### 正交投影

不详细的介绍了，直接贴出过程，基本上都是线性映射。上面看懂了，下面的自然就清楚了。

![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵18.png?raw=true)
![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵19.png?raw=true)
![](https://github.com/wangjiangchuan/pictures/blob/master/投影矩阵20.png?raw=true)

## 参考资料及转载出处
很多里面的内容直接来自一下的两篇原文，在关键的地方加上了自己的理解。
[OpenGL Transformation](http://www.songho.ca/opengl/gl_transform.html#matrix)
[OpenGL Projection Matrix](http://www.songho.ca/opengl/gl_projectionmatrix.html)
