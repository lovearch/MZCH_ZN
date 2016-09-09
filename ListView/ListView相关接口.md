# ListView相关接口   

|接口|作用|参数说明|
|---------|---------|---------|
|setSelectionFromTop(int position, int y)|ListView定位到特定的位置|position是item的位置， y为偏移量，以顶部为基准的偏移量|
|setSelection(int position)|ListView定位到指定的位置|position为item的位置|
|getChildAt(int index)|获取ListView中某个item|使用getChildAt(index)的取值，只能是当前可见区域（列表可滚动）的子项,即取值范围在 >= ListView.getFirstVisiblePosition() &&  <= ListView.getLastVisiblePosition(); |
|isStackFromBottom()|用于检测当前的状态是否为 从下到上 的方式||


# PinnedHeaderListView   

|API|功能|参数说明|  
|------------|-----------|----------|  
|void setPinnedHeaderAnimationDuration(int duration)|设置Header动画的时间||
|void setAdapter(ListAdapter adapter)|为PinnedHeaderListView设置Adapter|该Adapter需要实现PinnedHeaderAdapter的几个接口|
|void setOnScrollListener(OnScrollListener onScrollListener)|设置滑动事件监听|| 
|void setOnItemSelectedListener(OnItemSelectedListener listener)|设置ListView中Item的选中事件监听||  
|void setHeaderPinnedAtTop(int viewIndex, int y, boolean animate)|设置置顶的Header|viewIndex是置顶第几个Header置顶， animate是指定对此Header是否做动画|
|void setHeaderPinnedAtBottom(int viewIndex, int y, boolean animate)|设置置底的Header|viewIndex是置底的Header的index， animate是指定置底是否有动画，y是指定置底的Header的位置，单位是pixels|
|void setFadingHeader(int viewIndex, int position, boolean fade)|设置||
|void setHeaderInvisible(int viewIndex, boolean animate)|设置指定的Header显示为INVISIBLE|viewIndex为Header的Index， animate是是否做动画|
|int getTotalTopPinnedHeaderHeight()|获取所有置顶并且可见的Header的总高度||  
|int getPositionAt(int y)|得到屏幕上显示的某个位置的Item的Position|y为item的位置， 单位是pixels，是指ListView在其parent中的位置值|
|void setHeaderPaddingTop(int padding)|设置Header与顶部之间的padding值||
|int getHeaderPaddingTop()|获取Header与顶部的padding值||
|void setHeaderBackground(Drawable drawable)|设置Header的背景||
|int getCurrentOverScrollDistance()|？？？|？？？|

# PinnedHeaderAdapter接口  
|API|功能|参数说明|   
|---|---|---|
|int getPinnedHeaderCount()|返回PinnedHeader的个数||
|View getPinnedHeaderView(int viewIndex, View convertView, ViewGroup parent)|创建pinned header的View,包括Header的刷新|参数功能类似ListView的Adapter里面的getView函数的参数|
|void configurePinnedHeaders(PinnedHeaderListView listView)|设置Header的显示，包括置顶，置底，不可见等|参数是当前的PinnedHeaderListView对象|
|int getScrollPositionForHeader(int viewIndex)|？？？|？？？|
