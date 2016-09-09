# MultiCursorPartitionAdapter  
[参考文档](http://redmine.meizu.com/documents/992)  

## BasePartitionAdapter  
实现了父类的方法有
|function|Description|  
|---|---|  
|abstract Object getItem(int partitionIndex, int offset)|BasePartitionAdapter重写BaseAdapter.getItem里面的调用，实现的是获取分组item的数据|  
|abstract long getItemId(int partitionIndex, int offset)|BasePartitionAdapter重写BaseAdapter.getItemId里面的调用，实现的是获取分组item的id|  
|abstract View getView(int position, int partitionIndex, int offset,int itemBgType, View convertView, ViewGroup parent)|getView， 为分组的Item实现getView方法，类似BaseAdapter的getView|  

### getView()方法的复写  
``` java  
    @Override
    protected View getView(int position, int partitionIndex, int offset, int itemBgType,
                           View convertView, ViewGroup parent) {
        //得到游标
        Cursor cursor = getPartition(partitionIndex).mCursor;
        if (cursor == null ) {
            throw new IllegalStateException("the partition " + partitionIndex + " cursor is null");
        }
        // 移动游标到 partitionIndex
        int cursorPos = getDataPosition(partitionIndex, offset);
        if (!cursor.moveToPosition(cursorPos)) {
            throw new IllegalStateException("Couldn't move cursor to position " + cursorPos);
        }
        View view;
        if (convertView == null) {
            view = newView(mContext, position, partitionIndex, cursor, offset, itemBgType, parent);
        } else {
            view = convertView;
        }
        //
        bindView(view, mContext, position, partitionIndex, cursor, offset, itemBgType);
        return view;
    }
```   
给子类留下了这几个接口：   
bindView(view, mContext, position, partitionIndex, cursor, offset, itemBgType);  
newView(mContext, position, partitionIndex, cursor, offset, itemBgType, parent);  
这几个方法上面一个是绑定view用的，就是在这个函数里面实现view上面数据的初始化。下面的方法是创建新的item的view视图。  


### getItem 和 getItemId  
``` java   
    protected Object getItem(int partitionIndex, int offset) {
        Cursor cursor = getPartition(partitionIndex).mCursor;
        if (cursor == null || cursor.isClosed()) {
            return null;
        }

        int pos = getDataPosition(partitionIndex, offset);
        if (pos < 0 || !cursor.moveToPosition(pos)) {
            return null;
        }

        return cursor;
    }

    /**
     * Get the row id associated with the specified offset of the specified partition.
     */
    protected long getItemId(int partitionIndex, int offset) {
        CursorPartition cg = getPartition(partitionIndex);
        if (cg.mRowIDColumnIndex == -1) {
            return 0;
        }

        Cursor cursor = cg.mCursor;
        if (cursor == null || cursor.isClosed()) {
            return 0;
        }

        int pos = getDataPosition(partitionIndex, offset);
        if (pos < 0 || !cursor.moveToPosition(pos)) {
            return 0;
        }
        // 返回该列数据
        return cursor.getLong(cg.mRowIDColumnIndex);
    }
```   
这个联系上了cursor，主要是取出对应item上面显示的数据，offset表示的是当前的item在这个分组Partition中的位置。



## 包装了Partition 分组类   
``` java  
    public static class CursorPartition extends Partition {
        Cursor mCursor;
        int mRowIDColumnIndex;

        /**
         * 构造CursorPartition对象
         * @param showIfEmpty 组项数目count为0时是否显示该分组
         * @param hasHeader 是否显示header
         * @param c 组数据对应的Cursor
         */
        public CursorPartition(boolean showIfEmpty, boolean hasHeader, Cursor c) {
            super(showIfEmpty, hasHeader , c == null ? 0 : c.getCount());
            mCursor = c;
        }
    }
```   
给每个Partition分组提供了一个Cursor，用于查询数据。 

## 其余的接口有  
一共有这些，主要是对分组的一半操作。  
|接口| 作用|  
|---|---|
|int addPartition(boolean showIfEmpty, boolean hasHeader, Cursor c)||
|int addPartition(CursorPartition partition)||
|void removePartition(int partitionIndex)||
| void clearPartitions()|| 
|void clearCursors()||  
|CursorPartition getPartition(int partitionIndex)||
|Cursor getCursor(int partitionIndex)||
|int getDataPosition(int position)||
|int getDataPosition(int partitionIndex, int offset)||
|void changeCursor(int partitionIndex, Cursor cursor)||
|Cursor swapCursor(int partitionIndex, Cursor cursor)||

## 

## some functions   

``` java    
    @Override
    protected View getView(int position, int partitionIndex, int offset, int itemBgType,
            View convertView, ViewGroup parent) {
        // 
        Cursor cursor = getPartition(partitionIndex).mCursor;
        if (cursor == null ) {
            throw new IllegalStateException("the partition " + partitionIndex + " cursor is null");
        }
        
        int cursorPos = getDataPosition(partitionIndex, offset);
        if (!cursor.moveToPosition(cursorPos)) {
            throw new IllegalStateException("Couldn't move cursor to position " + cursorPos);
        }
        
        View view;
        if (convertView == null) {
            // 创建item视图，方法由子类实现
            view = newView(mContext, position, partitionIndex, cursor, offset, itemBgType, parent);
        } else {
            view = convertView;
        }
        //绑定视图，这个方法由子类实现
        bindView(view, mContext, position, partitionIndex, cursor, offset, itemBgType);
        return view;
    }
```  

获取item在分组中的位置
``` java  
    protected int getDataPosition(int partitionIndex, int offset) {
        return offset - mPartitions[partitionIndex].mHeaderViewsCount;
    }
```   

实现BasePartitionAdapter的getItem接口，返回的是Cursor对象
``` java  
    Object getItem(int partitionIndex, int offset) {
        Cursor cursor = getPartition(partitionIndex).mCursor;
        if (cursor == null || cursor.isClosed()) {
            return null;
        }
        
        int pos = getDataPosition(partitionIndex, offset);
        if (pos < 0 || !cursor.moveToPosition(pos)) {
            return null;
        }
        
        return cursor;
    }
```  

实现BasePartitionAdapter的getItemId,返回mRowIDColumnIndex列的数据
``` java  
protected long getItemId(int partitionIndex, int offset) {
        CursorPartition cg = getPartition(partitionIndex);
        if (cg.mRowIDColumnIndex == -1) {
            return 0;
        }

        Cursor cursor = cg.mCursor;
        if (cursor == null || cursor.isClosed()) {
            return 0;
        }
        
        int pos = getDataPosition(partitionIndex, offset);
        if (pos < 0 || !cursor.moveToPosition(pos)) {
            return 0;
        }
        // 返回该列数据
        return cursor.getLong(cg.mRowIDColumnIndex);
    }
```  

abstract 接口： 
新建item的View 
``` java  
abstract View newView(Context context, int position, int partitionIndex, Cursor cursor, int offset, int itemBgType, ViewGroup parent);
```  
绑定视图
``` java  
abstract void bindView(View v, Context context, int position, int partitionIndex, Cursor cursor, int offset, int itemBgType);
```    


