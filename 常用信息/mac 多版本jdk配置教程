通过命令’jdk6′, ‘jdk7′,’jdk8′轻松切换到对应的Java版本：

1.首先安装所有的JDk：
* Mac自带了的JDK6，安装在目录：/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/下。
* JDK7，JDK8则需要自己到Oracle官网下载安装对应的版本。自己安装的JDK默认路径为：/Library/Java/JavaVirtualMachines/jdk1.8.0.jdk

2、配置
创建.bash_profile配置文件（已经有该文件就跳过此步骤）

touch ~/.bash_profile
#vim编辑.bash_profile文件

vim ~/.bash_profile
#如果不习惯vim命令就使用自带的文本编辑器打开

open ~/.bash_profile
设置jdk版本

export JAVA_6_HOME=/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
export JAVA_7_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0.jdk/Contents/Home
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home
 
export JAVA_HOME=$JAVA_6_HOME
alias命令动态切换JAVA_HOME的配置

alias jdk8='export JAVA_HOME=$JAVA_8_HOME'
alias jdk7='export JAVA_HOME=$JAVA_7_HOME'
alias jdk6='export JAVA_HOME=$JAVA_6_HOME’
#输入完成后保存执行下面命令
#重新执行.bash_profile文件

source ~/.bash_profile

3、验证：
使用：jdk6、jdk7、jdk8 即可切换jdk版本
PS：http://www.yneit.com/2014/11/mac-jdk%E7%89%88%E6%9C%AC%E5%88%87%E6%8D%A2.html

　　vim操作又忘记了 常用命令可以参看http://www.yneit.com/?p=393

