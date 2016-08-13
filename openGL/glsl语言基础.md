# GLSL语言基础
## 数据类型
``` c
float  bool  int    基本数据类型
vec2                包含了2个浮点数的向量
vec3                包含了3个浮点数的向量
vec4                包含了4个浮点数的向量
ivec2               包含了2个整数的向量
ivec3               包含了3个整数的向量
ivec4               包含了4个整数的向量
bvec2               包含了2个布尔数的向量
bvec3               包含了3个布尔数的向量
bvec4               包含了4个布尔数的向量
mat2                2*2维矩阵
mat3                3*3维矩阵
mat4                4*4维矩阵
sampler1D           1D纹理采样器
sampler2D           2D纹理采样器
sampler3D           3D纹理采样器
```

# 顶点着色器
## 示例代码
``` c
uniform mat4 uMVPMatrix;             // 应用程序传入顶点着色器的总变换矩阵
attribute vec4 aPosition;             // 应用程序传入顶点着色器的顶点位置
attribute vec2 aTextureCoord;          // 应用程序传入顶点着色器的顶点纹理坐标
attribute vec4 aColor                  // 应用程序传入顶点着色器的顶点颜色变量
varying vec4 vColor                  // 用于传递给片元着色器的顶点颜色数据
varying vec2 vTextureCoord;          // 用于传递给片元着色器的顶点纹理数据
void main()
{
    gl_Position = uMVPMatrix * aPosition; // 根据总变换矩阵计算此次绘制此顶点位置
    vColor = aColor;               // 将顶点颜色数据传入片元着色器
    vTextureCoord = aTextureCoord;   // 将接收的纹理坐标传递给片元着色器
}
```
### uniform变量
uniform变量是外部application程序传递给（vertex和fragment）shader的变量。
因此它是application通过函数glUniform**（）函数赋值的。
在（vertex和fragment）shader程序内部，uniform变量就像是C语言里面的常量（const ），
它不能被shader程序修改。（shader只能用，不能改）

如果uniform变量在vertex和fragment两者之间声明方式完全一样，
则它可以在vertex和fragment共享使用。
（相当于一个被vertex和fragment shader共享的全局变量）

### attribute变量
attribute变量是只能在vertex shader中使用的变量。
（它不能在fragment shader中声明attribute变量，也不能被fragment shader中使用）
一般用attribute变量来表示一些顶点的数据，如：顶点坐标，法线，纹理坐标，顶点颜色等。

在application中，一般用函数glBindAttribLocation()来绑定每个attribute变量的位置，
然后用函数glVertexAttribPointer()为每个attribute变量赋值。

### varying变量
varying变量是vertex和fragment shader之间做数据传递用的。
一般vertex shader修改varying变量的值，然后fragment shader使用该varying变量的值。
因此varying变量在vertex和fragment shader二者之间的声明必须是一致的。
application不能使用此变量。
``` c
// Vertex shader
uniform mat4 u_matViewProjection;
attribute vec4 a_position;
attribute vec2 a_texCoord0;
varying vec2 v_texCoord; // Varying in vertex shader
void main(void)
{
gl_Position = u_matViewProjection * a_position;
v_texCoord = a_texCoord0;
}


// Fragment shader
precision mediump float;
varying vec2 v_texCoord; // Varying in fragment shader
uniform sampler2D s_baseMap;
uniform sampler2D s_lightMap;
void main()
{
vec4 baseColor;
vec4 lightColor;
baseColor = texture2D(s_baseMap, v_texCoord);
lightColor = texture2D(s_lightMap, v_texCoord);
gl_FragColor = baseColor * (lightColor + 0.25);
}
```

### gl_Position变量

gl_Position为内建变量，表示变换后点的空间位置。 顶点着色器从应用程序中获得
原始的顶点位置数据，这些原始的顶点数据在顶点着色器中经过平移、旋转、缩放等数学变换后，
生成新的顶点位置。
新的顶点位置通过在顶点着色器中写入gl_Position传递到渲染管线的后继阶段继续处理。

### 一张图展示数据的变化

![image](http://img1.tuicool.com/IFN3Mjy.jpg!web)

### in变量
``` c
#version 330  
layout(location = 0) in vec3 attrib_position;  
layout(location = 1) in vec2 attrib_texcoord;  
layout(location = 2) in vec3 attrib_normal;  
layout(location = 3) in int  attrib_clustercount;
```
在上面的Vertex Program代码中，第一行（#version 330），
表明我们现在使用的GLSL版本是GLSL3.3，以区别于以前的版本并允许我们使用基于GLSL3.3的功能。
在过去，OpenGL的版本和GLSL版本是不统一的（前文中的GL2.2所对应的是GLSL1.2，
而后来的对应关系是GL3.0-GLSL1.3，GL3.1-GLSL1.4，GL3.2-GLSL1.5），
直到2010年OpenGL3.3/4.0规范的提出，khronos委员会决定让两者版本统一，
所以就有了现在本博客所使用的OpenGL3.3-GLSL3.3的对应关系（注，ShaderModel4.0的显卡可达到的最高版本）。

接下来的几行声明了4个attribute变量。
在GL2.x中一个attribute变量通常是“attribute vec3 attrib_position;”这样来表示，
在GL3.x中，废弃了attribute关键字（以及varying关键字），
属性变量统一用in/out作为前置关键字，对每一个Shader stage来说，
in表示该属性是作为输入的属性，out表示该属性是用于输出的属性。
这里，attribute变量作为Vertex Shader的顶点输入属性，所以都用in标记。
另外，这里使用了layout关键字
（通常是layout(layoutAttrib1=XXX, layoutAttrib2=XXX, ...)这样的形式）。
这个关键字用于一个具体变量前，用于显式标明该变量的一些布局属性，
这里就是显式设定了该attribute变量的位置值(location)，
其作用跟ShaderProgram(着色程序)链接前
调用glBindAttribLocation来设定atribute变量的位置值是等效的。

为什么采用这种方式更好呢？其一当然是编码量减少了，
二来也避免了去Get某个attribute的location带来的开销，
三来，最重要的是，它重定义了OpenGL和GLSL之间attribute变量属性的依赖。
过去我们的OpenGL端必须首先要知道GLSL端某个attribute的名字，
才能设置/获得其位置值，如今两者只需要location对应起来就可以完成绘制时顶点属性流的传递了。
不再需要在ShaderProgram的compile和link之间插入代码也更方便于其模块化。
### out变量
指定变量为着色器阶段的一个输出，意思就是这个变量是这个阶段着色器的输出变量，肯能会用到
下一个作色器的输入变量

# 片段着色器
## 示例代码
``` c
precision mediump float;     // 设置工作精度
varying vec4 vColor;         // 接收从顶点着色器过来的顶点颜色数据
varying vec2 vTextureCoord;  // 接收从顶点着色器过来的纹理坐标
uniform sampler2D sTexture;  // 纹理采样器，代表一幅纹理
void main()
{                                                                                   
    gl_FragColor = texture2D(sTexture, vTextureCoord) * vColor;// 进行纹理采样
}
```
##基本介绍
片元着色器是一个处理片元值及其相关联数据的可编程单元，
片元着色器可执行纹理的访问、颜色的汇总、雾化等操作，每片元执行一次。
片元着色器替代了纹理、颜色求和、雾以及Alpha测试，
这一部分是需要开发者自己开发的。
![](http://img0.tuicool.com/my6fAv.jpg!web)

(1）varying指的是从顶点着色器传递到片元着色器的数据变量

(2）gl_FragColor为内置变量，用来保存片元着色器计算完成的片元颜色值，
此颜色值将送入渲染管线的后继阶段进行处理。

## precision变量
该变量是描述精度的属性
![image](http://my.csdn.net/uploads/201207/17/1342509878_2153.jpg)
最后是数据类型，及使用中等的float精度



# 参考资料
[shader相关](http://www.tuicool.com/articles/VZVJra)

[opengl向shader中传递数据](
http://blog.csdn.net/candycat1992/article/details/8830894)

[GLSL vary、atrribute、in、out的区别](http://blog.csdn.net/shenshen211/article/details/51802700)

[GLSL语法](http://www.tuicool.com/articles/NVNvmaR)