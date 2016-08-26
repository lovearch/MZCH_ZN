# 解bug
## android studio中导入相关的moudle
* 首先在 app 的 build.gradle中的dependencies中添加如下的依赖项:

``` xml
    compile project(':FlymeAppCompat')
    compile project(':MzRecyclerView')
    compile project(':ColorTheme:ColorTheme-Coral')
    compile project(':ColorTheme:ColorTheme-DodgerBlue')
    compile project(':ColorTheme:ColorTheme-FireBrick')
    compile project(':ColorTheme:ColorTheme-LimeGreen')
    compile project(':ColorTheme:ColorTheme-SeaGreen')
    compile project(':ColorTheme:ColorTheme-Crimson')
    compile project(':ColorTheme:ColorTheme-Grey')
    compile project(':ColorTheme:ColorTheme-Blue')
    compile project(':ColorTheme:ColorTheme-Tomato')
    compile project(':PtrPullRefreshLayout')
    compile project(':MzTextInputLayout')
    compile project(':MzPasswordLock')
```

* 在setting.gradle中指定这些依赖的moudle 的位置

``` java
include ':app'

include ':MeizuCommon'
project(':MeizuCommon').projectDir = new File(settingsDir, '../../MeizuCommon')

include ':res-meizu-common'
project(':res-meizu-common').projectDir = new File(settingsDir, '../../../../flyme/frameworks/flyme-res/res-meizu-common')

include ':ColorTheme:ColorTheme-Tomato'
project(':ColorTheme:ColorTheme-Tomato').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-Tomato')


include ':ColorTheme:ColorTheme-Crimson'
project(':ColorTheme:ColorTheme-Crimson').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-Crimson')


include ':ColorTheme:ColorTheme-Coral'
project(':ColorTheme:ColorTheme-Coral').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-Coral')


include ':ColorTheme:ColorTheme-DodgerBlue'
project(':ColorTheme:ColorTheme-DodgerBlue').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-DodgerBlue')


include ':ColorTheme:ColorTheme-FireBrick'
project(':ColorTheme:ColorTheme-FireBrick').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-FireBrick')


include ':ColorTheme:ColorTheme-LimeGreen'
project(':ColorTheme:ColorTheme-LimeGreen').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-LimeGreen')

include ':ColorTheme:ColorTheme-SeaGreen'
project(':ColorTheme:ColorTheme-SeaGreen').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-SeaGreen')

include ':ColorTheme:ColorTheme-Blue'
project(':ColorTheme:ColorTheme-Blue').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-Blue')

include ':ColorTheme:ColorTheme-Grey'
project(':ColorTheme:ColorTheme-Grey').projectDir = new File(settingsDir, '../../ColorTheme/ColorTheme-Grey')


include ':FlymeAppCompat'
project(':FlymeAppCompat').projectDir = new File(settingsDir, '../../MeizuWidget/FlymeAppCompat')

include ':MzRecyclerView'
project(':MzRecyclerView').projectDir = new File(settingsDir, '../../MeizuWidget/MzRecyclerView')

include ':PtrPullRefreshLayout'
project(':PtrPullRefreshLayout').projectDir = new File(settingsDir, '../../MeizuWidget/PtrPullRefreshLayout')

include ':MzTextInputLayout'
project(':MzTextInputLayout').projectDir = new File(settingsDir, '../../MeizuWidget/MzTextInputLayout')

include ':MzPasswordLock'
project(':MzPasswordLock').projectDir = new File(settingsDir, '../../MeizuWidget/MzPasswordLock')

include ':flyme-support-v4'
project(':flyme-support-v4').projectDir = new File(settingsDir, '../../MeizuWidget/flyme-support-v4')

```

上面的
`new File(settingsDir, '../../MeizuWidget/flyme-support-v4')`
指定moudle的路径, `../../MeizuWidget/flyme-support-v4`
只要这个正确了就可以.

* 接下来 sync projects

同步项目,即可加载Moudle.



bug已经发现问题的原因:
原因是 在FastScrollLetter 的  getSection方法里面, 定义的 section  = -1; 这个section在
else if (mDefaultSectionType == SECTION_COMPARE_TYPE1) {    //TYPE1
       int position = mSectionIndexer.getPositionForSection(section); 
的getPositionForSection里面,由于开发者使用的是 直接调用AlphabetIndexer.getPositionForSection(sectionIndex)方法,
这个方法里面 char letter = mAlphabet.charAt(sectionIndex); 这个由于传入的是 -1 导致数组越界,崩溃.

修改方法如下:
1. 重写SectionIndexer  getPositionForSection(int sectionIndex)的时候,出错的写法是直接调用 AlphabetIndexer.getPositionForSection(sectionIndex), 两个sectionIndex一样,但是AlphabetIndexer在原生的getPositionForSection方法里面会直接按照创建对象时的赋值的第三个参数(CharSequence alphabet)(赋值的是 A到Z)中查找sectionIndex对应的分组,但是listview中的数据不一定A到Z都有,造成sectionIndex定位的section不准确(原生是直接二分法去查找的),那么 getPositionForSection返回的结果就是错的. 之前你的能用,不报错是因为原生算法是就算找不到那个分组,也能返回最近一个分组的第一个item的位置,而且由于FastScroolLetter类型判定,不会执行后面的,能正常运行.

2. 重写SectionIndexer getSectionForPosition(int position)的时候,由于SECTION_COMPARE_TYPE2类型的FastScrollLetter.getSection()方法后面会去
调用这个方法, 如果不按照自己的数据去找到这个position对应的section的话,数据会出错,FastScrollLetter不能正常定位.
所以这个方法不能直接依靠 AlphabetIndexer.getSectionForPosition(), 里面的源码还是按照创建对象时赋值的A到Z去找,
返回的结果就是错误的.

3. 在FastScrollLetter.getSection()方法里面加上一句判定,如下:
if (mDefaultSectionType == SECTION_COMPARE_TYPE1) {    //TYPE1\
    //修改的地方
    if(section != -1) {
    //修改结束
        int position = mSectionIndexer.getPositionForSection(section);     //获得section中第一个item的位置
        // 修复Bug,id:209454, 判断 ListView 中是否含有 FooterView,[start]
        if (mAbsListView instanceof ListView) {
            footerViewCount = ((ListView) mAbsListView).getFooterViewsCount();
        }
        //修复Bug,id:209454,[end]  //Bug 375725: 数组越界
        if (position < mAbsListView.getCount() - footerViewCount) {
            if (mSectionIndexer.getSectionForPosition(position) == section) {
                return section;
            }
        }
    }
}