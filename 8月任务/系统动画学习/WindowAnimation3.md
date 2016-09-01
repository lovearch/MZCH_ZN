# systemAnimation

|WindowAnimation|Complete|
|-----------|-----------|
|windowEnterAnimation| yes |
|windowExitAnimation | yes|
|windowShowAniamtion| yes |
|*windowHideAniamtion*|*no*|


|ActivityAnimation| Compelte |
|------|------|
|activityOpenEnterAnimation| yes |
|activityOpenExitAnimation | yes|
|activityCloseEnterAnimation|yes|
|activityCloseExitAnimation|yes|


|TaskAnimation| Complete |
|----------|---------|
|taskOpenEnterAnimation | yes  |
|taskOpenExitAnimation | yes |
|launchTaskBehindTargetAnimation |intent.FLAG_ACTIVITY_LAUNCH_BEHIND 没有找到这个标志位|
|launchTaskBehindSourceAnimation|intent.FLAG_ACTIVITY_LAUNCH_BEHIND 没有找到这个标志位|
|taskCloseEnterAnimation|yes|
|taskCloseExitAnimation|yes |
|taskToFrontEnterAnimation| yes|
|taskToFrontExitAnimation|yes |
|taskToBackEnterAnimation| yes|
|taskToBackExitAnimation| yes|


|wallpaper Animation | Complete | Tips |
|----------|---------|
|wallpaperOpenEnterAnimation||当从一个不显示Wallpaper的activity切换到一个显示WallPaper的activity时会做的动画|
|wallpaperOpenExitAnimation| yes|
|wallpaperCloseEnterAnimation|yes|
|wallpaperCloseExitAnimation|yes|
|wallpaperIntraOpenEnterAnimation|yes|
|wallpaperIntraOpenExitAnimation|yes|
|wallpaperIntraCloseEnterAnimation|yes|
|wallpaperIntraCloseExitAnimation|yes|

## 对一下动画的使用场景介绍
|WallPaper Animation| Sence |
|-------------------|--------|
|wallpaperOpenEnterAnimation|打开一个带WallPaper的Activity，但当前的Activity不带wallpaper时，新打开的Activity所作的动画|
|wallpaperOpenExitAnimation|打开一个带WallPaper的Activity，但当前的Activity不带wallpaper时，不带WallPaper的Activity所做的动画|
|wallpaperCloseEnterAnimation|打开一个不带wallpaper的Activity，但当前的Activity带wallpaper事，不带wallpaper的Activity的进入动画|
|wallpaperCloseExitAnimation|打开一个不带wallpaper的Activity，但当前的Activity带wallpaper事，带wallpaper的Activity的退出动画|
|wallpaperIntraOpenEnterAnimation||
|wallpaperIntraOpenExitAnimation||
|wallpaperIntraCloseEnterAnimation||
|wallpaperIntraCloseExitAnimation||
