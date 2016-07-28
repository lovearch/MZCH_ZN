# WindowAnimation动画

## 与window有关的一些动画
<declare-styleable name="WindowAnimation">
    <attr name="windowEnterAnimation" format="reference" />
    <attr name="windowExitAnimation" format="reference" />
    <attr name="windowShowAnimation" format="reference" />
    <attr name="windowHideAnimation" format="reference" />

## 与activity有关的一些动画
    <attr name="activityOpenEnterAnimation" format="reference" />
    <attr name="activityOpenExitAnimation" format="reference" />
    <attr name="activityCloseEnterAnimation" format="reference" />
    <attr name="activityCloseExitAnimation" format="reference" />
    <attr name="taskOpenEnterAnimation" format="reference" />

## 首先对四种启动模式的掌握
四种启动模式是:

1. standard启动模式

![1]
每次跳转系统都会在task中生成一个新的FirstActivity实例，并且放于栈结构的顶部，当我们按下后退键时，才能看到原来的FirstActivity实例。
standard启动模式，不管有没有已存在的实例，都生成新的实例。








[1]: /root/AtomFolder/MZCH_ZN/系统动画学习/文档所需图片/standard启动模式.png