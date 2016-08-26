# 安卓Camera 3D使用  
## Camera使用的坐标系  
![左手坐标系](http://www.2cto.com/uploadfile/Collfiles/20140415/2014041509410277.jpg)  
X轴是手机的水平方向，Y轴是手机的竖直方向，Z轴是垂直于手机向里的那个方向。  

**举例说明**  
1、camera位于坐标点（0,0），也就是视图的左上角；  
2、camera.translate(10,20,30)的意思是把观察物体右移10，上移20，向前移30（即让物体远离camera，这样物体将会变小）；  
3、camera.rotateX(45)的意思是绕X轴顺时针旋转45度。
   举例来说，如果物体中间线和X轴重合的话，绕X轴顺时针旋转45度就是指物体上半部分向里翻转，
   下半部分向外翻转；  
4、camera.rotateY(45)的意思是绕Y轴顺时针旋转45度。
   举例来说，如果物体中间线和Y轴重合的话，绕Y轴顺时针旋转45度就是指物体左半部分向外翻转，
   右半部分向里翻转；  
5、camera.rotateZ(45)的意思是绕Z轴逆时针旋转45度。
   举例来说，如果物体中间线和Z轴重合的话，绕Z轴顺时针旋转45度就是物体上半部分向左翻转，
   下半部分向右翻转。 

## Camera API 接口  
![Camera API接口](http://www.2cto.com/uploadfile/Collfiles/20140415/2014041509410378.jpg)  

方法的说明：  
1、applyToCanvas(Canvas canvas) 根据当前的变换计算出相应的矩阵，然后应用到制定的画布上去，
   注意是由画布来设置矩阵的。  
2、rotateX(float degree) 绕着x轴旋转degree个度数  
3、rotateY(float degree) 绕着y轴旋转degree个度数   
4、rotateZ(float degree) 绕着z轴旋转degree个度数
   Z轴的旋转，基本上就是在XY轴构成的平面上进行旋转，   
5、translate(float x,float y,float z) 在x、y、z坐标轴上执行变换操作  
   这个函数就是translate，在上面图中的Camera中的坐标上面进行平移，包括X,Y,Z三个方面。
6、save()和restore() 保存原状态，操作完之后，恢复到原状态。

## Demo演示  
``` java  
        final Matrix matrix = t.getMatrix();
        mCamera.save();
        mCamera.translate(0.0f, 0.0f, degrees);
        mCamera.rotateY(degrees);
        mCamera.getMatrix(matrix);
        // 把经过计算的 Matrix 传递给 目标的Matrix
        mCamera.restore();
        // 下面作用是将动画的中心点移动到 对象的正中间
        matrix.preTranslate(-mCenterX, -mCenterY);
        matrix.postTranslate(mCenterX, mCenterY);
```  
首先将amera对象保存起来，这里涉及到一个常用的和Canvas一样的两个函数：restore和save  
这两个函数的作用是保存Camera状态和恢复Camera的状态，而且保存和恢复是一一对应的，如果  
保存的调用少于恢复的调用，就会Error。
保存和恢复的概念理解：Camera.save()就相当于存档，之后再调用rotate和translate相关的代码，这样  
后面对Camera所作的内容就与存档无关了，当调用restore时候，相当于读取了存档内容，之前所作的  
都不会保存起来；为什么不需要保存，是因为每次rotate或translate一般都是在原始的基础上进行的  
比如 rotate旋转一个角度的，角度从0-180度，逐渐增加的，所以旋转的时候是在没有旋转的原始  
图像上面进行的，如果不保存，那么每次旋转的角度都会保存在Camera中，每次都是在上次旋转的  
基础上面进行的，最终的角度就是 1 +。。。。。+180了，因为没有保存状态，并在每次最开始的  
时候回复状态。

**preTranslate   postTranslate**  
这个函数是在做变换之前调用的，打个比方，scale缩放时候，是在图像的正中间进行的，但是  
scale动画时在(0, 0)点进行的，也就是屏幕的左上角的位置，所以在做动画的时候需要先平移  
到(0, 0)的位置，scale完了之后，再postTranslate回来到原来的位置。  

**mCamera.getMatrix(matrix)**  
这个方法是在Camera做完了变换之后调用的，就是将Camera所做的变化保存到matrix中，这个matrix是  
Transformation.getMatrix获得的，这个Matrix是作用在所做动画的Object上面的，将Camera  
的变化传递给Matrix，所以这里的Camera实际上只是计算而已。
