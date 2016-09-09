# PinnedHeaderAdapter   
## 源码  
``` java   
public interface PinnedHeaderAdapter {
        /**
         * Returns the overall number of pinned headers, visible or not.
         */
        int getPinnedHeaderCount();

        /**
         * Creates or updates the pinned header view.
         */
        View getPinnedHeaderView(int viewIndex, View convertView, ViewGroup parent);

        /**
         * Configures the pinned headers to match the visible list items. The
         * adapter should call {@link PinnedHeaderListView#setHeaderPinnedAtTop},
         * {@link PinnedHeaderListView#setHeaderPinnedAtBottom},
         * {@link PinnedHeaderListView#setFadingHeader} or
         * {@link PinnedHeaderListView#setHeaderInvisible}, for each header that
         * needs to change its position or visibility.
         */
        void configurePinnedHeaders(PinnedHeaderListView listView);

        /**
         * Returns the list position to scroll to if the pinned header is touched.
         * Return -1 if the list does not need to be scrolled.
         */
        int getScrollPositionForHeader(int viewIndex);
    }
```   
主要是定义了  PinnedHeaderAdapter 接口， 这个接口里面的三个函数的作用是用来创建分组header的视图view用的，和初始化Header。  

