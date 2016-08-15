# Open GL ES 2.0 Hello World
这是OpenGL ES在android平台的最简单的例子，实现的是三角形的绘制，
下面将介绍整个实现的流程，从而展示OpenGL ES代码流程。
本实例使用OpenGL ES 2.0。

## 基本概念
* 图形有所谓的正反面，如果我们看向一个图形，它的顶点序列是逆时针方向，
那我们看到的就是正面。
* Vertex shader，控制顶点的绘制，指定坐标、变换等。
* Fragment shader，控制形状内区域渲染，纹理填充内容。

## 坐标系的介绍
![](https://github.com/wangjiangchuan/pictures/blob/master/1.png?raw=true)
三维坐标系，原点在中间，x 轴向右，y 轴向上，z 轴朝向我们，x y z 取值范围都是 [-1, 1]：

## 参考资料
[安卓 OpenGL ES 2.0 完全入门（一）：基本概念和 hello world](http://blog.piasy.com/2016/06/07/Open-gl-es-android-2-part-1/)

## 流程介绍
### 在Manifest中添加声明
为了使用OpenGL ES 2.0 API，需要添加如下声明： 

`<uses-feature android:glEsVersion="0x00020000" android:required="true" />`

### GLSurfaceView
GLSurfaceView是用来放置图形view的容器。
所有的东西都是绘制在GLSurfaceView上面的，就相当于画布的概念，
这里先实现一个GLSurfaceView。
扩展自GLSurfaceView，实现自己的MyGLSurfaceView

``` java
public class MyGLSurfaceView extends GLSurfaceView {
    
    public MyGLSurfaceView(Context context) {
        super(context);
    }

    public MyGLSurfaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```
这里有两个构造方法：
*  public MyGLSurfaceView(Context context)
*  public MyGLSurfaceView(Context context, AttributeSet attrs)
类似自定义View的构造方法一样，context传入activity的上下文信息，
AttributeSet是参数设置相关。

使用OpenGL ES 2.0还需要加上这句话：
``` java
// Create an OpenGL ES 2.0 context
setEGLContextClientVersion(2);
```
有一点需要注意的是，在构造函数里面，要调用setRender()来设置Render对象。
Render相关的见下面。

### Render类创建
Renderer类（渲染器类），即 GLSurfaceView.Renderer的实现类，
它控制了与它相关联的 GLSurfaceView 上绘制什么。
``` java
public class MyRender implements GLSurfaceView.Renderer {

    @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {

    }

    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {

    }

    @Override
    public void onDrawFrame(GL10 gl) {

    }
}
```
这里面有三个主要的回调方法：
* public void onSurfaceCreated(GL10 gl, EGLConfig config)
onSurfaceCreated 在 surface 创建时被回调，通常用于进行初始化工作，
只会被回调一次；

* public void onSurfaceChanged(GL10 gl, int width, int height)
onSurfaceChanged 在每次 surface 尺寸变化时被回调，注意，
第一次得知 surface 的尺寸时也会回调；

* public void onDrawFrame(GL10 gl)
onDrawFrame 则在绘制每一帧的时候回调。

有时候，初始化工作可能需要依赖 surface 的尺寸，
所以可以把初始化工作放到了 onSurfaceChanged 方法中。
其中的GL10 参数是 OpenGL 1.0遗留下来的，2.0之后不用使用了。
### vertexShaer 脚本

### fragmentShader 脚本


### Opengl中矩阵的详细介绍
[OpenGL矩阵变换](http://www.songho.ca/opengl/gl_transform.html)

由于该文档是英文文档，此处提供中文解释：

[多图解矩阵变化](http://www.flakor.cn/2014-05-15-384.html)




