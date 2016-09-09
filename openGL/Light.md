# 光照  
## 环境光  
环境光不是直接的漫射光，但是能够充满整个空间。  

``` java  
final color = material color * ambient light color
//物体的最终颜色是物理的本质颜色与环境光的乘积，打个比方，颜色由RGB三种组成
//物体颜色{1, 0, 0}，红色的物体，环境光{0.1, 0.1, 0.1}，环境光为白色的
final color = {1, 0, 0} * {0.1, 0.1, 0.1} = {0.1, 0.0, 0.0}
//最终物体的颜色为淡红色，接近白色
```  

## 点光源  point light source  
点光源的计算和环境光的计算不同，需要加入衰减系数以及光源的位置，光源的位置是用来计算物体表面和光源之间的角度，
会影响到物体表面整个光的等级，同样光源的位置也可以用来计算光源和物体表面的距离，距离决定着某一点上面光照的强度。    
计算遵循以下步骤：  
* 计算兰伯特因子  lambert factor   
首先，需要计算出光源和物体表面的角度，垂直照射的面的强度为最大，背对光源的强度为最小；最快的计算方式利用兰伯特因子。
举例：有两个向量，一个是从光源到表面上某点，一个是面的法向量。首先将向量归一化，然后进行点积运算，得到向量之间的夹角，原文如下：
then we can calculate the cosine by first normalizing each vector so that it has a length of one, 
and then by calculating the dot product of the two vectors.   

``` java  
    light vector = light position - object position
    cosine = dot product(object normal, normalize(light vector))
    lambert factor = max(cosine, 0)
```  
第一行是用来计算光源到物体表面的向量，接着，在物体的法向量和光源到物体表面的归一化向量做点乘，物体的法向量已经归一化。
得到两个向量之间角度的cos值，大小在-1到1之间，第三行将区间截取为0到1之间。  
举例：  
``` java   
//              光源位置        点的坐标
light vector = {0, 10, -10} - {0, 0, 0} = {0, 10, -10}
// 物体表面的法向量
object normal = {0, 1, 0}
```    
计算归一化向量
``` java   
light vector length = square root(0*0 + 10*10 + -10*-10) = square root(200) = 14.14
normalized light vector = {0, 10/14.14, -10/14.14} = {0, 0.707, -0.707}
```   
计算点积
``` java   
dot product({0, 1, 0}, {0, 0.707, -0.707}) = (0 * 0) + (1 * 0.707) + (0 * -0.707) = 0 + 0.707 + 0 = 0.707
```   

``` java   
lambert factor = max(0.707, 0) = 0.707  
```   
* 计算衰减因子   
第一步中得到的 light vector length = 14.14，由兰伯特因子计算公式：  
```  java    
luminosity = 1 / (distance * distance)
```   
``` java   
luminosity = 1 / (14.14*14.14) = 1 / 200 = 0.005
```   

* 计算最终的颜色   
由公式：   
``` java   
final color = material color * (light color * lambert factor * luminosity)
```     
根据上面的例子，可以得到最终的颜色值为：   
``` java    
//             物体颜色       光源颜色    兰伯特因子  衰减因子   
final color = {1, 0, 0} * ({1, 1, 1} * 0.707 * 0.005}) = {1, 0, 0} * {0.0035, 0.0035, 0.0035} = {0.0035, 0, 0}
``` 

** 总结如下：  
``` java     
//Step one
light vector = light position - object position
cosine = dot product(object normal, normalize(light vector))
lambert factor = max(cosine, 0)
 
//Step two
luminosity = 1 / (distance * distance)
 
//Step three
final color = material color * (light color * lambert factor * luminosity)
```   

## OpenGL ES 中的代码    
结合上面的分析过程，在Opengl ES 中的代码如下形式：   
``` java   
// uniform为外部传递的变量
final String vertexShader =
    "uniform mat4 u_MVPMatrix;      \n"     // A constant representing the combined model/view/projection matrix.
  + "uniform mat4 u_MVMatrix;       \n"     // A constant representing the combined model/view matrix.
  + "uniform vec3 u_LightPos;       \n"     // The position of the light in eye space.
 
  + "attribute vec4 a_Position;     \n"     // 顶点坐标，有程序传入   Per-vertex position information we will pass in.
  + "attribute vec4 a_Color;        \n"     // 颜色坐标，由程序传入   Per-vertex color information we will pass in.
  + "attribute vec3 a_Normal;       \n"     // Per-vertex normal information we will pass in.
 
  + "varying vec4 v_Color;          \n"     // 顶点颜色的坐标，会传递给片元着色器，是最终计算的到的颜色坐标    This will be passed into the fragment shader.
 
  + "void main()                    \n"     // The entry point for our vertex shader.
  + "{                              \n"
// Transform the vertex into eye space.
  + "   vec3 modelViewVertex = vec3(u_MVMatrix * a_Position);              \n"
// Transform the normal's orientation into eye space.
  + "   vec3 modelViewNormal = vec3(u_MVMatrix * vec4(a_Normal, 0.0));     \n"
// Will be used for attenuation.
// 这个modelViewVertex就是得到的object position，
  + "   float distance = length(u_LightPos - modelViewVertex);             \n"
// Get a lighting direction vector from the light to the vertex.
  + "   vec3 lightVector = normalize(u_LightPos - modelViewVertex);        \n"
// Calculate the dot product of the light vector and vertex normal. If the normal and light vector are
// pointing in the same direction then it will get max illumination.
// 计算兰伯特因子，这个是由平面该点的法线向量与归一化光源到点矩阵的点积
  + "   float diffuse = max(dot(modelViewNormal, lightVector), 0.1);       \n"
// Attenuate the light based on distance.
// 计算衰减因子 并与 兰伯特常数相乘   (1.0 / (1.0 + (0.25 * distance * distance)))衰减因子
  + "   diffuse = diffuse * (1.0 / (1.0 + (0.25 * distance * distance)));  \n"
// Multiply the color by the illumination level. It will be interpolated across the triangle.
// 计算最终的颜色
  + "   v_Color = a_Color * diffuse;                                       \n"
// gl_Position is a special variable used to store the final position.
// Multiply the vertex by the matrix to get the final point in normalized screen coordinates.
  + "   gl_Position = u_MVPMatrix * a_Position;                            \n"
  + "}                                          
```   
里面用到了MVMatrix，Modle/View Matrix.  一下是详细的介绍：   
``` java   
// Transform the vertex into eye space.
  + "   vec3 modelViewVertex = vec3(u_MVMatrix * a_Position);              \n"

Since we're passing in the position of the light in eye space, we convert the current vertex position to a coordinate in eye space so we can calculate the proper distances and angles.


// Transform the normal's orientation into eye space.
  + "   vec3 modelViewNormal = vec3(u_MVMatrix * vec4(a_Normal, 0.0));     \n"

We also need to transform the normal's orientation. Here we are just doing a regular matrix multiplication like with the position, but if the model or view matrices have been scaled or skewed, this won't work: we'll actually have to undo the effect of the skew or scale by multiplying the normal by the transpose of the inverse of the original matrix. This website best explains why we have to do this.


// Will be used for attenuation.
  + "   float distance = length(u_LightPos - modelViewVertex);             \n"

As shown before in the math section, we need the distance in order to calculate the attenuation factor.


// Get a lighting direction vector from the light to the vertex.
  + "   vec3 lightVector = normalize(u_LightPos - modelViewVertex);        \n"

We also need the light vector to calculate the Lambertian reflectance factor.


// Calculate the dot product of the light vector and vertex normal. If the normal and light vector are
// pointing in the same direction then it will get max illumination.
  + "   float diffuse = max(dot(modelViewNormal, lightVector), 0.1);       \n"

This is the same math as above in the math section, just done in an OpenGL ES 2 shader. The 0.1 at the end is just a really cheap way of doing ambient lighting (the value will be clamped to a minimum of 0.1).


// Attenuate the light based on distance.
  + "   diffuse = diffuse * (1.0 / (1.0 + (0.25 * distance * distance)));  \n"

The attenuation math is a bit different than above in the math section. We scale the square of the distance by 0.25 to dampen the attenuation effect, and we also add 1.0 to the modified distance so that we don't get oversaturation when the light is very close to an object (otherwise, when the distance is less than one, this equation will actually brighten the light instead of attenuating it).


// Multiply the color by the illumination level. It will be interpolated across the triangle.
  + "   v_Color = a_Color * diffuse;                                       \n"
// gl_Position is a special variable used to store the final position.
// Multiply the vertex by the matrix to get the final point in normalized screen coordinates.
  + "   gl_Position = u_MVPMatrix * a_Position;                            \n"

Once we have our final light color, we multiply it by the vertex color to get the final output color, and then we project the position of this vertex to the screen.
```   



## Fragment Shader   
``` java    
final String fragmentShader =
  "precision mediump float;       \n"     // Set the default precision to medium. We don't need as high of a
                                          // precision in the fragment shader.
+ "varying vec4 v_Color;          \n"     // This is the color from the vertex shader interpolated across the
                                          // triangle per fragment.
+ "void main()                    \n"     // The entry point for our fragment shader.
+ "{                              \n"
+ "   gl_FragColor = v_Color;     \n"     // Pass the color directly through the pipeline.
+ "}               
```   




* 

``` java   

```  

