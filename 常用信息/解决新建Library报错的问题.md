# 解决新建Library build报错的问题

出现该问题
**unspecified on project app resolves to an APK archive which is not supported as a compilation dependency**
的情形可能是：创建了两个Module，其中一个Module依赖另一个Module而导致了出现该问题；
如果在Android Studio中，有ModuleA和ModuleB，我们希望ModuleA依赖ModuleB，运行时候可能会出现该问题，
查看被依赖的ModuleB的build.gradle，里面可以看到：  
``` java   
apply plugin: 'com.android.application'  
```  
这句话告诉了Gradle将ModuleB编译称为application，也就是apk，这就是问题的所在；  

## 解决办法
解决方法：将上面该句改为:   
``` java  
apply plugin: 'com.android.library'  
```  

## 改完继续出现问题有  
此时，Gradle将编译称为一个Library，也就是库，运行之后，
如果出现这个问题：
**Error:Library projects cannot set applicationId. applicationId is set to 'package_name' in default config.**，
那是因为一个库不允许设置applicationId，
需要将 **builde.gradle — android — defaultConfig中的applicationId** 删除；

如果说，我们ModuleB仍然需要生成apk，则我们需要将其中公共的代码放到一个Module，作为一个support的库；

http://stackoverflow.com/questions/27536491/how-to-import-android-project-as-library-and-not-compile-it-as-apk-android-stud