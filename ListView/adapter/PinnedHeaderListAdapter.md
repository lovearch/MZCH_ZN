# PinnedHeaderListAdapter   
继承关系：  
``` java   
class PinnedHeaderListAdapter extends MultiCursorPartitionAdapter
        implements PinnedHeaderListView.PinnedHeaderAdapter
```  
所以要实现的函数主要有以下这些。  

## 父类MultiCursorPartitionAdapter接口实现  
### 


## interface PinnedHeaderAdapter接口实现  
### getPinnedHeaderView  
``` java   
    @Override
    public View getPinnedHeaderView(int partition, View convertView, ViewGroup parent) {
        if (hasHeader(partition)) {
            View view = null;
            if (convertView != null) {
                Integer headerType = (Integer)convertView.getTag();
                if (headerType != null && headerType == PARTITION_HEADER_TYPE) {
                    view = convertView;
                }
            }
            int position = getPositionForPartition(partition);
            if (view == null) {
                view = newHeaderView(mContext, position, partition, parent);
                view.setTag(PARTITION_HEADER_TYPE);
                view.setFocusable(false);
                view.setEnabled(false);
            }
            bindHeaderView(view, mContext, position, partition);
            return view;
        } else {
            return null;
        }
    }
```    
如果有convertView， 则检查类型是否为PARTITION_HEADER_TYPE，如果不是则创建新的headerView。最后将headerView绑定到分组。  

``` java   
    @Override
    public void configurePinnedHeaders(PinnedHeaderListView listView) {
        if (!mPinnedPartitionHeadersEnabled) {
            return;
        }

        int size = getPartitionCount();

        // Cache visibility bits, because we will need them several times later on
        if (mHeaderVisibility == null || mHeaderVisibility.length != size) {
            mHeaderVisibility = new boolean[size];
        }
        for (int i = 0; i < size; i++) {
            boolean visible = isPinnedPartitionHeaderVisible(i);
            mHeaderVisibility[i] = visible;
            if (!visible) {
                listView.setHeaderInvisible(i, true);
            }
        }

        int headerViewsCount = listView.getHeaderViewsCount();

        // Starting at the top, find and pin headers for partitions preceding the visible one(s)
        int maxTopHeader = -1;
        int topHeaderHeight = 0;
        // 遍历分组
        for (int i = 0; i < size; i++) {
            if (mHeaderVisibility[i]) {
                // position为屏幕0位置处的item在整个listview中的位置，出去了headerview
                int position = listView.getPositionAt(topHeaderHeight) - headerViewsCount;
                // 得到position的分组
                int partition = getPartitionForPosition(position);
                if (i > partition) {
                    break;
                }

                listView.setHeaderPinnedAtTop(i, topHeaderHeight, false);
                topHeaderHeight += listView.getPinnedHeaderHeight(i);
                maxTopHeader = i;
            }
        }

        // Starting at the bottom, find and pin headers for partitions following the visible one(s)
        int maxBottomHeader = size;
        int bottomHeaderHeight = 0;
        int listHeight = listView.getHeight();
        for (int i = size; --i > maxTopHeader;) {
            if (mHeaderVisibility[i]) {
                int position = listView.getPositionAt(listHeight - bottomHeaderHeight)
                        - headerViewsCount;
                if (position < 0) {
                    break;
                }

                int partition = getPartitionForPosition(position - 1);
                if (partition == -1 || i <= partition) {
                    break;
                }

                int height = listView.getPinnedHeaderHeight(i);
                bottomHeaderHeight += height;
                // Animate the header only if the partition is completely invisible below
                // the bottom of the view
                int firstPositionForPartition = getPositionForPartition(i);
                boolean animate = position < firstPositionForPartition;
                listView.setHeaderPinnedAtBottom(i, listHeight - bottomHeaderHeight, animate);
                maxBottomHeader = i;
            }
        }

        // Headers in between the top-pinned and bottom-pinned should be hidden
        for (int i = maxTopHeader + 1; i < maxBottomHeader; i++) {
            if (mHeaderVisibility[i]) {
                listView.setHeaderInvisible(i, isPartitionEmpty(i));
            }
        }
    }
``` 
这个函数有些长.  




