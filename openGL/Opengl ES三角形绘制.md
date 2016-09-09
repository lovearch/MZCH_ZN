# Open GL ES 三角形绘制   
画三角形是open GL ES中最简单的入门项目，下面讲解具体的流程，方便自己总结工具类，没有别的意思。

## 在Android项目中引进open GL  
为了使用OpenGL ES 2.0 API，需要添加如下声明： 

`<uses-feature android:glEsVersion="0x00020000" android:required="true" />`

## 创建GLSurfaceView   
GLSurfaceView是用来放置图形view的容器。所有的东西都是绘制在GLSurfaceView上面的，就相当于画布的概念，
这里先实现一个GLSurfaceView。扩展自GLSurfaceView，实现自己的MyGLSurfaceView  

``` java   
    public class MyGLSurfaceView extends GLSurfaceView {

        private Context mContext;

        public MyGLSurfaceView(Context context) {
            super(context);
        }

        public MyGLSurfaceView(Context context, AttributeSet attrs) {
            super(context, attrs);
        }
    }  
```   

## 创建Render  
Renderer类（渲染器类），即 GLSurfaceView.Renderer的实现类，它控制了与它相关联的 GLSurfaceView 上绘制什么。  
需要实现一下接口：    
``` java   
 @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        
    }

    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {

    }

    @Override
    public void onDrawFrame(GL10 gl) {

    }
```   
* onSurfaceCreated()函数在surface创建的时候调用，所以初始化的工作在里面完成。  
* onSurfaceChanged()函数在surface发生改变的时候调用；  
* onDrawFrame()函数是完成surfaceview上面显示内容的绘制，每一帧的绘制都会去调用。   

### 基础参数设置   
``` java   
    //给shader中的变量传参数时候用到的
    private final int mStrideBytes = 7 * 4;    //3 + 4   3表示坐标， 4表示颜色  一共7个float变量，每个变量4字节//一次性读取 7 x 4个字节
    private final int mPositionOffset = 0;   //顶点坐标的偏移量
    private final int mPositionDataSize = 3;   //3个为一组，表示一个顶点坐标
    private final int mColorOffset = 3;    //颜色数据的变异量为3， 也就是每次读取数据，从第三个开始是表示颜色的
    private final int mColorDataSize = 4;   //4个数据都是表示颜色的
```   
* mStrideBytes是指定buffer在读取数据的时候一次读取多少，7表示个数，7个数据， 4表示字节，7*4表示一次读取多少字节的数据。
比如：  
``` java
 -0.5f, -0.25f, 0.0f,    //point
1.0f, 0.0f, 0.0f, 1.0f, //color
```    
* mPositionOffset表示顶点坐标的偏移量   
* mPositionDataSize表示顶底每个顶点坐标用多少个数据表示，三个：` -0.5f, -0.25f, 0.0f,    //point `   
* mColorOffset读取颜色数据时的偏移量，因为顶点坐标用3个数据表示，所以偏移量为3   
* mColorDataSize表示多少个数据表示一个颜色，4个参数分别为ARGB    


### 顶点坐标和颜色坐标   

```
//数据
    private final float vertexData[] = {
            // X, Y, Z,
            // R, G, B, A
            -0.5f, -0.25f, 0.0f,    //point
            1.0f, 0.0f, 0.0f, 1.0f, //color

            0.5f, -0.25f, 0.0f,     //point
            0.0f, 0.0f, 1.0f, 1.0f, //color

            0.0f, 0.559016994f, 0.0f,//point
            0.0f, 1.0f, 0.0f, 1.0f     //color
    };
```   
可以看到，每组数据有7个，前面3个表示位置坐标，后面4个表示颜色值，可以结合前面的参数设置来理解。
这个数据是程序传入openGL的数据。   

### 创建Buffer存放数据   
``` java
    //数据的buffer
    private FloatBuffer mShaderDateBuffer;
    private FloatBuffer getVertexBuffer(float[] data) {
        //先 创建内存地址
        ByteBuffer vbb = ByteBuffer.allocateDirect(data.length * 4);   //每个float是4个字节
        //ByteOrder.nativeOrder()返回本地jvm运行的硬件的字节顺序.使用和硬件一致的字节顺序可能使buffer更加有效.
        vbb.order(ByteOrder.nativeOrder());
        FloatBuffer vertexBuffer = vbb.asFloatBuffer();
        vertexBuffer.put(data);
        vertexBuffer.position(0);

        return vertexBuffer;
    }
```   
上面代码中的注释已经很清楚了，里面的代码大多数是固定的写法。   

###  Matrix的设置  
openGL 中有三个类型份额举证，分别是:  
* Model Matrix   模型矩阵
* View Matrix    视图矩阵
* Projection Matrix  投影矩阵  
由于这几个概念很绕，是个人都要糊弄一会儿才能搞清楚，下面就这几个矩阵，好好的糊弄糊弄。  
所谓的坐标变换，就是将一个坐标系下的坐标，在另外一个坐标系中表示出来。  
下图中世界坐标系下的相机:   
![坐标系中的相机](http://image79.360doc.com/DownloadImg/2014/10/2810/46548286_50)  
将相机定位在(1,0,1,1)(注意这个地方是用4维向量来表示，最后一个维度取值只能为0或者1，1表示点，0表示向量)。
照相机的指向可以用 n = (-1,0,-1,0) 用原点坐标(0,0,0,1)减去相机坐标得到的。同时设置照相机的观察正向和世界坐标系下的
y轴的方向一致， v = (0,1,0,0), 此时，利用向量的叉乘可以得到 相机坐标系下的第三个坐标轴的方向，u = n x v ;
计算得到u = (1,0,-1,0) ,这样照相机自己构成的坐标系为(u, v, n, P).  

![照相机坐标系](http://image79.360doc.com/DownloadImg/2014/10/2810/46548286_51)  
这个东西就是将坐标从世界坐标转换到相机坐标的矩阵，那世界坐标的原点为例(0,0,0,1), 有  
![原点从世界坐标转换到相机坐标](http://image79.360doc.com/DownloadImg/2014/10/2810/46548286_52)   

在opengl中，数据从用户构建的局部坐标系，经过一系列的处理，最终渲染在屏幕上面，主要经过了一下过程：  
![](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_1)  
Open gl中只定义了裁剪坐标系、规范化设备坐标系以及屏幕坐标系，而局部坐标系、世界坐标系和相机坐标系是为了
方便用户的，用户在OpenGL中的转换如下：  
![](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_2)  
从坐标来看，就是一下过程   
![](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_3)  
下图中茶壶在Model Matrix中的定义  
![Model Matrix下的茶壶](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_5)  
世界坐标系下的茶壶：  
![世界坐标系下的茶壶](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_5)   
从世界坐标系到相机坐标系  
![](http://image79.360doc.com/DownloadImg/2014/10/2809/46547825_13)   
为什么要将世界坐标系转化到相机坐标系，我们最终看到的就是相机拍到的，而不是上帝视角下看到的一切(纯属个人理解，不喜勿喷)。  
从相机坐标系到裁剪坐标系，通过投影完成的。分为正交投影和透视投影两种。 
最后读下来还是感觉很晕，的确很晕。那这些东西和上面提到的三个矩阵有什么关系呢？  
基本上就是： Model Matrix是模型坐标系转换到世界坐标系用，View Matrix就是视图坐标系，用来转换到相机坐标系用的，Projection Matrix转化裁剪坐标系的。  

### ES中坐标系矩阵的计算  
计算模型矩阵，里面调用了rotateM接口  
``` java  
    private float[] mModelMatrix = new float[16];       //模型矩阵
    private void initModelMatrix() {
        // Do a complete rotation every 10 seconds.
        long time = SystemClock.uptimeMillis() % 10000L;
        float angleInDegrees = (360.0f / 10000.0f) * ((int) time);

        // Draw the triangle facing straight on.
        // 模型矩阵设为单位矩阵
        Matrix.setIdentityM(mModelMatrix, 0);
        // angleInDegrees是旋转的角度，(0.0f, 0.0f, 1.0f)是模型矩阵
        Matrix.rotateM(mModelMatrix, 0, angleInDegrees, 0.0f, 0.0f, 1.0f);
    }
``` 
计算ViewMatrix，需要设置相机的位置，相机观察的方向，以及相机的观察正向向量   
``` java  
    private float[] mViewMatrix = new float[16];   //视图矩阵
    private void initViewMatrix() {
        //放置eye眼睛的位置
        final float eyeX = 0.0f;
        final float eyeY = 0.0f;
        final float eyeZ = 1.5f;

        //设置look方向
        // look也成为center， center到eye所形成的向量，称为视线方向，与真正的视线看过去的方向相反
        final float lookX = 0.0f;
        final float lookY = 0.0f;
        final float lookZ = -5.0f;

        //设置up坐标
        //eye的位置本身只代表一个坐标而已，但是 look向量和up向量结合右手螺旋准则才能唯一的确定一个坐标系
        //这个坐标系就是eye看到的坐标系，两者垂直知识为了与常用的三位坐标系一样
        final float upX = 0.0f;
        final float upY = 1.0f;
        final float upZ = 0.0f;

        //经过计算的到了 viewMatrix，
        Matrix.setLookAtM(mViewMatrix, 0, eyeX, eyeY, eyeZ, lookX, lookY, lookZ, upX, upY, upZ);

    }
``` 

投影矩阵，设置关于远近 以及投影屏幕的大小等属性。   
``` java  
    private float[] mProjectionMatrix = new float[16];   //投射矩阵
    private void initProjectionMatrix(int width, int height) {
        // Set the OpenGL viewport to the same size as the surface.
        GLES20.glViewport(0, 0, width, height);
        // Create a new perspective projection matrix. The height will stay the same
        // while the width will vary as per aspect ratio.
        final float ratio = (float) width / height;
        final float left = -ratio;
        final float right = ratio;
        final float bottom = -1.0f;
        final float top = 1.0f;
        final float near = 1.0f;
        final float far = 10.0f;

        Matrix.frustumM(mProjectionMatrix, 0, left, right, bottom, top, near, far);
    }
```   

### Shader数据传入   
使用工具类：   
``` java   
    public static int compileShader(final int shaderType, final String shaderSource) {
        // 创建shader句柄
        int shaderHandle = GLES20.glCreateShader(shaderType);
        if (shaderHandle != 0) {
            // Pass in the shader source.
            // 使用glShaderSource()分别将顶点着色程序的源代码字符数组绑定到顶点着色器对象，将片段着色程序的源代码字符数组绑定到片段着色器对象；
            // 绑定作用
            GLES20.glShaderSource(shaderHandle, shaderSource);

            // Compile the shader.
            // 编译
            GLES20.glCompileShader(shaderHandle);

            // Get the compilation status.
            // 得到计算结果
            final int[] compileStatus = new int[1];
            GLES20.glGetShaderiv(shaderHandle, GLES20.GL_COMPILE_STATUS, compileStatus, 0);

            // If the compilation failed, delete the shader.
            // 结果审查
            if (compileStatus[0] == 0) {
                Log.e(TAG, "Error compiling shader: " + GLES20.glGetShaderInfoLog(shaderHandle));
                GLES20.glDeleteShader(shaderHandle);
                shaderHandle = 0;
            }
        }

        if (shaderHandle == 0) {
            throw new RuntimeException("Error creating shader.");
        }

        return shaderHandle;
    }
```  
shaderType为两种，一种是GLES20.GL_VERTEX_SHADER，一种是GLES20.GL_FRAGMENT_SHADER。最终返回的是shader的句柄，后面要用到这个句柄传入参数，
计算。   
链接程序  
vertexShader
``` c 
    uniform mat4 u_MVPMatrix;   //应用程序传入的变换矩阵 ，MVP 是modle view projection的意思，通过这个来计算最终的坐标
    attribute vec4 a_Position;  //应用程序传入的 顶点的坐标
    attribute vec4 a_Color;     //应用程序传入的 顶点颜色的坐标

    varying vec4 v_Color;   //这个变量会传到 fragment shader中处理

    void main() {
        v_Color = a_Color;
        gl_Position = u_MVPMatrix * a_Position;
    }
```   
fragmentShader  
``` java   
    precision mediump float;   //精度

    varying vec4 v_Color;   //这个名字一定要和在 vertex shader中声明的一样

    void main() {
        gl_FragColor = v_Color;
    }
```   
变量,这些变量是在shader中定义的，这里的名称和c语言中定义的名称一致。通过这个个名称获取变量，传入参数      
``` java   
private String[] mAttributes = {
            "u_MVPMatrix",
            "a_Position",
            "a_Color"
    };
```  
``` java   
    public static int createAndLinkProgram(final int vertexShaderHandle, final int fragmentShaderHandle, final String[] attributes) {
        // 创建程序句柄 
        int programHandle = GLES20.glCreateProgram();

        if (programHandle != 0) {
            // Bind the vertex shader to the program.
            // 绑定shader到program
            GLES20.glAttachShader(programHandle, vertexShaderHandle);

            // Bind the fragment shader to the program.
            GLES20.glAttachShader(programHandle, fragmentShaderHandle);

            // Bind attributes
            // 绑定参数到program
            if (attributes != null) {
                final int size = attributes.length;
                for (int i = 0; i < size; i++) {
                    GLES20.glBindAttribLocation(programHandle, i, attributes[i]);
                }
            }

            // Link the two shaders together into a program.
            GLES20.glLinkProgram(programHandle);

            // Get the link status.
            final int[] linkStatus = new int[1];
            GLES20.glGetProgramiv(programHandle, GLES20.GL_LINK_STATUS, linkStatus, 0);

            // If the link failed, delete the program.
            if (linkStatus[0] == 0) {
                Log.e(TAG, "Error compiling program: " + GLES20.glGetProgramInfoLog(programHandle));
                GLES20.glDeleteProgram(programHandle);
                programHandle = 0;
            }
        }

        if (programHandle == 0) {
            throw new RuntimeException("Error creating program.");
        }

        return programHandle;
    }
```  

### 最后一步，绘制图形   
onSurfaceCreated函数主要是做初始化用的。  
``` java   
    @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        //清屏指令
        GLES20.glClearColor(0f, 0f, 0f, 0f);
        //初始化相机的位置
        initViewMatrix();

        initShader();
    }

    private void initShader() {
        int vertexShaderHandle = ShaderHelper.compileShader(GLES20.GL_VERTEX_SHADER, TextResourceReader.readTextFileFromResource(mContext, R.raw.vertex_shader));
        int fragmentShaderHandle = ShaderHelper.compileShader(GLES20.GL_FRAGMENT_SHADER, TextResourceReader.readTextFileFromResource(mContext, R.raw.fragment_shader));

        int programHandle = ShaderHelper.createAndLinkProgram(vertexShaderHandle, fragmentShaderHandle, mAttributes);

        //者三个变量是我们在glsl文件中定义的三个变量，现在链接程序之后把他们取出来用，是为了后面赋值
        mMVPMatrixHandle = GLES20.glGetUniformLocation(programHandle, "u_MVPMatrix");
        mPositionHandle = GLES20.glGetAttribLocation(programHandle, "a_Position");
        mColorHandle = GLES20.glGetAttribLocation(programHandle, "a_Color");

        GLES20.glUseProgram(programHandle);
    }
```   
onSurfaceChanged()函数  
``` java  
    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
        initProjectionMatrix(width, height);
    }
```      

onDrawFrame()绘制  
``` java   
    @Override
    public void onDrawFrame(GL10 gl) {

        GLES20.glClear(GLES20.GL_DEPTH_BUFFER_BIT | GLES20.GL_COLOR_BUFFER_BIT);
        initModelMatrix();
        drawFrame(mShaderDateBuffer);

    }
    private void drawFrame(final FloatBuffer frameBuffer) {
        //移动到 表示坐标的起始位置
        frameBuffer.position(mPositionOffset);
        //glVertexAttribPointer(positionSlot, 2, GL_FLOAT, GL_FALSE, stride, pCoords);
        /* 为顶点着色器位置信息赋值，
            1.positionSlot表示顶点着色器位置属性（即，Position）；就是在glsl文件中声明的attribute变量
            2.表示每一个顶点信息由几个值组成，这个值必须位1，2，3或4；
            3.GL_FLOAT表示顶点信息的数据类型；
            4.GL_FALSE表示不要将数据类型标准化（即fixed-point）；
            5.stride表示数组中每个元素的长度；pCoords表示数组的首地址
        */
        GLES20.glVertexAttribPointer(mPositionHandle, mPositionDataSize, GLES20.GL_FLOAT, false,
                mStrideBytes, frameBuffer);
        //开启顶点属性数组
        GLES20.glEnableVertexAttribArray(mPositionHandle);

        //定位到color数据首地址
        frameBuffer.position(mColorOffset);
        GLES20.glVertexAttribPointer(mColorHandle, mColorDataSize, GLES20.GL_FLOAT, false,
                mStrideBytes, frameBuffer);
        //开启顶点属性数组
        GLES20.glEnableVertexAttribArray(mColorHandle);

        Matrix.multiplyMM(mMVPMatrix, 0, mViewMatrix, 0, mModelMatrix, 0);

        // This multiplies the modelview matrix by the projection matrix, and stores the result in the MVP matrix
        // (which now contains model * view * projection).
        Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mMVPMatrix, 0);

        GLES20.glUniformMatrix4fv(mMVPMatrixHandle, 1, false, mMVPMatrix, 0);
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);

    }
```   
``` 
GLES20.glVertexAttribPointer(mPositionHandle, mPositionDataSize, GLES20.GL_FLOAT, false,
                mStrideBytes, frameBuffer);
```  
mPositionHandle是`GLES20.glGetAttribLocation(programHandle, "a_Position");`绑定的变量，mPositionDataSize表示position坐标的数据个数，
也就是多少个数据表示一个坐标，mStrideBytes表示每次读取多少字节数据。  
同样地，`GLES20.glVertexAttribPointer(mColorHandle, mColorDataSize, GLES20.GL_FLOAT, false,mStrideBytes, frameBuffer);`
是传入color数据。  
Matrix.multiplyMM是计算MVP矩阵。

###  glDrawArrays参数详解  
在OpenGl中所有的图形都是通过分解成三角形的方式进行绘制。
绘制图形通过GL10类中的glDrawArrays方法实现，

该方法原型:
glDrawArrays(int mode, int first,int count)

*参数1：有三种取值
1.GL_TRIANGLES：每三个顶之间绘制三角形，之间不连接
2.GL_TRIANGLE_FAN：以V0V1V2,V0V2V3,V0V3V4，……的形式绘制三角形
3.GL_TRIANGLE_STRIP：顺序在每三个顶点之间均绘制三角形。这个方法可以保证从相同的方向上所有三角形均被绘制。以V0V1V2,V1V2V3,V2V3V4……的形式绘制三角形
*参数2：从数组缓存中的哪一位开始绘制，一般都定义为0
*参数3：顶点的数量

### MainActivity   
MainActivity中增加opengl版本支持相关代码。  
``` java   
    //GLSurfaceView
    private MyGLSurfaceView mGLSurfaceView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initGL();
        final ActivityManager activityManager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
        final ConfigurationInfo configurationInfo = activityManager.getDeviceConfigurationInfo();
        final boolean supportsEs2 = configurationInfo.reqGlEsVersion >= 0x20000;
        if (supportsEs2)
        {
            // Request an OpenGL ES 2.0 compatible context.
            mGLSurfaceView.setEGLContextClientVersion(2);
            // Set the renderer to our demo renderer, defined below.
            mGLSurfaceView.setRenderer(new MyRender(this));
        }
        else
        {
            // This is where you could create an OpenGL ES 1.x compatible
            // renderer if you wanted to support both ES 1 and ES 2.
            return;
        }
        setContentView(mGLSurfaceView);
    }
    //初始化 opengl相关
    private void initGL() {
        mGLSurfaceView = new MyGLSurfaceView(this);
    }
    //在下面的两个方法中，必须有对  GLSurfaceView的处理，当Activity暂停时，在onPause中处理
    //同样在onResume中有相应的恢复处理
    //下面是最长见的处理方法
    @Override
    protected void onPause() {
        super.onPause();
        mGLSurfaceView.onPause();
    }
    @Override
    protected void onResume() {
        super.onResume();
        mGLSurfaceView.onResume();
    }
```   

## 总结  
画一个三角形和写一个Hello World一样难！！！  

## References   
[open GL空间坐标系的理解](http://www.360doc.com/content/14/1028/10/19175681_420517010.shtml)  






