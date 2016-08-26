# FastScrollLetter使用简介
这篇文章的内容是针对前面的FastScrollLetter使用做出的补充，方便在使用这个
控件的过程中清晰逻辑，避免潜在的问题。

[FastScrollLetter使用基础](http://redmine.meizu.com/documents/135)  
这篇文章中，对FastScrollLetter的基本api做了介绍，以及xml中常用的属性做了说明，并且附上了简单的Demo演示，很实用。但是，最近发现在使用FastScrollLetter时，出现了一些错误的用法，导致bug，所以在这里对这篇文章进行补充。

## 错误的写法 
在使用FastScorllLetter时，都会创建一个Adapter类，并且实现SectionIndexer三个接口；这时，会给Adapter添加一个FastScrollLetter对象，用于快速导航条实现，一般都是这样创建的：
  
``` java  
AlphabetIndexer mIndexer;
mIndexer = new AlphabetIndexer(new IndexCursor(this), 0, "ABCDEFGHIJKLMNOPQRSTUVWXYZ");
```
这里直接设置AlphabetIndexer的Index Section是从A到Z的，当然也可以设置别的。
下面是容易出错的地方： 
  
``` java  
public int getPositionForSection(int sectionIndex);
public int getSectionForPosition(int position);
public Object[] getSection();
```
一般的写法都是这样子的：  

``` java
 @Override
    public int getPositionForSection(int sectionIndex) {
        return mIndexer.getPositionForSection(sectionIndex);
    }
    
 @Override
    public int getSectionForPosition(int position) {
        return mIndexer.getSectionForPosition(position);
    }
 @Override
    public Object[] getSections() {
        return mStrArr; //ABCDEFGHIJKLMNOPQRSTUVWXYZ
    }
```  
这样写会存在很大的一个漏洞，我们从 AlphabetIndexer 源码说起。

## 原因
### getPositionForSection方法正确写法  

AlphabetIndexer内部也重写了getPositionForSection()方法,源码有些长,大概如下:  

``` java  
public int getPositionForSection(int sectionIndex) {        
        ......
        // 下面这一部分是确定二分法查找的 区间和中点
        // 以及二分法查找的关键 字母 也就是 section 
        char letter = mAlphabet.charAt(sectionIndex);
        String targetLetter = Character.toString(letter);
        int key = letter;
        if (Integer.MIN_VALUE != (pos = alphaMap.get(key, Integer.MIN_VALUE))) {
            if (pos < 0) {
                pos = -pos;
                end = pos;
            } else {
                // Not approximate, this is the confirmed start of section, return it
                return pos;
            }
        }
        if (sectionIndex > 0) {
            int prevLetter =
                    mAlphabet.charAt(sectionIndex - 1);
            int prevLetterPos = alphaMap.get(prevLetter, Integer.MIN_VALUE);
            if (prevLetterPos != Integer.MIN_VALUE) {
                start = Math.abs(prevLetterPos);
            }
        }
        pos = (end + start) / 2;
        String targetLetter = Character.toString(letter);
        int key = letter;
        ........
        pos = (end + start) / 2;

        // 二分法查找代码
        ......
```  
上面有一个关键的地方, `char letter = mAlphabet.charAt(sectionIndex);` 它的作用是找到  
分组section的index值,就是第几组,它是直接使用mAlphabet来查找的,这个就是创建AlphabetIndexer  
时候赋值的A到Z的一串索引,也就是分组section.  
如果,在重写getPositionForSection方法时,直接调用AlphabetIndexer.getPositionForSection,  
当mAlphabet和adapter里面数据分组section一致的情况下,这里不会出错.  
但是如果A到Z里面缺了一组(比如C), 就会有问题, getPositionForSection(int sectionIndex)里面  
的sectionIndex是分组所在的Index值, 打个比方, 我的数据分组是A C E F, 这个C分组的Index值  
是1, 但是mAlphabet是在ABCDEFG ... XYZ中的分组中去找的,mAlphabet.charAt(1)得到的是B,  
所以,后面的二分法找出来的都是有问题的, 而且这个二分法还有优化,即使所找的分组不存在,会返回字母最近  
的一个有Item的分组. 

正确的写法:  

``` java  
@Override
public int getPositionForSection(int sectionIndex) {
    // mStrArr是Adapter里面数组的分组
    // 获取原数据分组 里面sectionIndex对应的分组字母
    String target = mStrArr[sectionIndex];
    // 在AlphabetIndexer的mAlphabet中找到对应的分组index
    int currentPos = -1;
    for(int i = 0 ; i < mLetters2.length; i++) {
        if(target.equals(mLetters2[i])) {
            currentPos = i;
        }
    }
    // 然后再去调用
    return mIndexer.getPositionForSection(currentPos );
}
```

### getSectionForPosition方法正确写法  
这个方法是返回当前位置的Item所在的分组,AlphabetIndexer里面的源码比较简单,如下:   

``` java  
@Override
public int getSectionForPosition(int position) {
    int savedCursorPos = mDataCursor.getPosition();
    mDataCursor.moveToPosition(position);
    String curName = mDataCursor.getString(mColumnIndex);
    mDataCursor.moveToPosition(savedCursorPos);
    // Linear search, as there are only a few items in the section index
    // Could speed this up later if it actually gets used.
    for (int i = 0; i < mAlphabetLength; i++) {
        char letter = mAlphabet.charAt(i);
        String targetLetter = Character.toString(letter);
        if (compare(curName, targetLetter) == 0) {
            return i;
        }
    }
    return 0; // Don't recognize the letter - falls under zero'th section
}
```
这里依然是使用mAlphabet.charAt方法,所以还是会有上面一样的问题. 正确的写法是:   

``` java   
@Override
public int getSectionForPosition(int position) {
    // 这部分就是遍历Adapter的数据, 找到对应的分组,
    // 所以,你需要自己找到当前position的分组,不能调用AlphabetIndexer的方法
    // 当然如果数据一样,或者section分组一样,是可以调用的.
    // 以下只是我给的一个示例而已
    int section = -1;
    String obj = mContactList.get(position);
    String target = obj.substring(0, 1);

    for(int i = 0 ; i < mStrArr.length; i++) {
        if(target.equals(mStrArr[i])) {
            section = i;
        }
    }
    return section;
}
```

### getSections方法  
这个方法在FastScrollLetter里面会回调,用来设置在侧边栏上面显示的快捷导航字母,所以  
当设置为A到Z的时候,点击快速导航跳上对应字母位置,旁边显示放大的字母. 正常情况下,如果没有分组
是不会显示的. 所以这里应该返回数据源的分组信息,而不是AlphabetIndexer的分组.










