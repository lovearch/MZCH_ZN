# 自定义View详细流程

[自定义View官方文档](https://developer.android.com/training/custom-views/index.html) <br/>
下面的内容来自于博客，附上Demo代码
## 示例1
自定义TextView

## 示例2


## Tips
res与res-auto的区别
通常我们在布局文件中使用自定义属性的时候
会这样写
xmlns:app="http://schemas.android.com/apk/res/包路径"
但如果你当前工程是做为lib使用，那么你如上所写 ，会出现找不到自定义属性的错误 。
这时候你就可以 写成
xmlns:app="http://schemas.android.com/apk/res-auto"

