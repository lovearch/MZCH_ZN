# BaseParitionAdapter接口
|API|function|description|
|-----|------|-----|
|boolean isPartitionEmpty(int partitionIndex)|分组为空判断||  
|int getPartitionForItemIndex(int itemIndex)|查询item所在分组||  
|int getPartitionForPosition(int position)|查询listView上某个位置的分组，包括Header、Footer等||
|int getOffsetInPartition(int position)|查询item在当前分组中的位置||

## getView()  
留出了getView(position, i, offset, itemBgType, convertView, parent)方法，子类去实现。  
``` java  
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ensureCacheValid();
        int start = 0;
        for (int i = 0; i < mPartitionCount; i++) {
            int end = start + mPartitions[i].mSize;
            if (position >= start && position < end) {
                int offset = position - start;
                if (mPartitions[i].mHasHeader) {
                    --offset;
                }

                int itemBgType = getItemBackgroundType(i, offset);
                View view;
                if (offset == -1) {
                    view = getHeaderView(position, i, convertView, parent);
                } else if (isHeaderView(i, offset)) {
                    view = mPartitions[i].mHeaderViewInfos.get(offset).view;
                } else if (isFooterView(i, offset)) {
                    int footerStart = mPartitions[i].mCount - mPartitions[i].mFooterViewsCount;
                    view = mPartitions[i].mFooterViewInfos.get(offset - footerStart).view;
                } else {
                    view = getView(position, i, offset, itemBgType, convertView, parent);
                }

                if (view == null) {
                    throw new NullPointerException("View should not be null, partition: " + i
                            + " position: " + position);
                }
                //#ifndef Flyme_EDIT
                //#zhangxin@SDK.BasePartitionAdapter.Feature do not background on Flyme4.0
//                if (offset != -1) {
//                    setViewBackground(view, itemBgType);
//                }
                //#else
                //#endif /* Flyme_EDIT */ 

                return view;
            }
            start = end;
        }

        throw new ArrayIndexOutOfBoundsException(position);
    }  
    
```  

getItem(int position)函数 ,里面留出了getItem(i, offset)接口，子类实现，其余的是返回Header或者Footer的data。      
``` java   
// Get the data item associated with the specified position in the data set
    // 返回Item下对应的数据
    @Override
    public Object getItem(int position) {
        ensureCacheValid();
        int start = 0;
        for (int i = 0; i < mPartitionCount; i++) {
            int end = start + mPartitions[i].mSize;
            if (position >= start && position < end) {
                int offset = position - start;
                if (mPartitions[i].mHasHeader) {
                    --offset;
                }

                if (offset == -1) {
                    return null;
                } else if (isHeaderView(i, offset)) {
                    return mPartitions[i].mHeaderViewInfos.get(offset).data;
                } else if (isFooterView(i, offset)) {
                    int footerStart = mPartitions[i].mCount - mPartitions[i].mFooterViewsCount;
                    return mPartitions[i].mFooterViewInfos.get(offset - footerStart).data;
                } else {
                    return getItem(i, offset);
                }
            }
            start = end;
        }

        return null;
    }
```    

getItemId()函数，留出了getItemId(i, offset)接口，用于子类实现，实现的是返回item的ID    
``` java   
    @Override
    public long getItemId(int position) {
        ensureCacheValid();
        int start = 0;
        for (int i = 0; i < mPartitionCount; i++) {
            int end = start + mPartitions[i].mSize;
            if (position >= start && position < end) {
                int offset = position - start;
                if (mPartitions[i].mHasHeader) {
                    --offset;
                }
                if (offset == -1) {
                    return 0;
                } else if (isHeaderView(i, offset) || isFooterView(i, offset)) {
                    return -1;
                } else {
                    return getItemId(i, offset);
                }
            }
            start = end;
        }
        return 0;
    }
```     
其余的接口是管理Partition用的，管理分组。包括怎删改查。  

## 子类必须实现以下方法，用于BaseAdapter的接口实现功能。   
``` java  
// 重写
    protected abstract View getView(int position, int partitionIndex, int offset,
                                    int itemBgType, View convertView, ViewGroup parent);
    // 重写
    protected abstract long getItemId(int partitionIndex, int offset);
    // 重写
    protected abstract Object getItem(int partitionIndex, int offset);
```   



# AbsBasePartitionAdapter 
``` java  
@Override
	public int getCount() {
		return 0;
	}

	@Override
	public Object getItem(int position) {
		return null;
	}

	@Override
	public long getItemId(int position) {
		return 0;
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		return null;
	}
	
    /**
     * 判断首列表项是否是group header
     */
	public boolean isTopHeader() {
		return false;
	}  
```   

## 还有几个方法值得注意
这几个方法在BasePartitionAdapter中本身是没有写的，知识定义出来，估计是留给子类去实现的。  
``` java   
    protected View newHeaderView(Context context, int position, int partitionIndex, ViewGroup parent) {
        return null;
    }

    protected void bindHeaderView(View v, Context context, int position, int partitionIndex) {
    }

    protected abstract View getView(int position, int partitionIndex, int offset,
                                    int itemBgType, View convertView, ViewGroup parent);
```   


