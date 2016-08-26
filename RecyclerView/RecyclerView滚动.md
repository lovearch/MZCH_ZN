# RecyclerView滚动

设置滚动监听  
``` java
 /**
     * Set a listener that will be notified of any changes in scroll state or position.
     *
     * @param listener Listener to set or null to clear
     *
     * @deprecated Use {@link #addOnScrollListener(OnScrollListener)} and
     *             {@link #removeOnScrollListener(OnScrollListener)}
     */
    // 设置滑动监听， 这个方法已经过时了，不建议使用
    @Deprecated
    public void setOnScrollListener(OnScrollListener listener) {
        mScrollListener = listener;
    }

    /**
     * Add a listener that will be notified of any changes in scroll state or position.
     *
     * <p>Components that add a listener should take care to remove it when finished.
     * Other components that take ownership of a view may call {@link #clearOnScrollListeners()}
     * to remove all attached listeners.</p>
     *
     * @param listener listener to set or null to clear
     */
    // 设置多个滚动监听
    // 可能是因为支持多监控，所以抛弃了上面的方法
    public void addOnScrollListener(OnScrollListener listener) {
        if (mScrollListeners == null) {
            mScrollListeners = new ArrayList<>();
        }
        mScrollListeners.add(listener);
    }

```  
这两个方法的参数都是 OnScrollListener ，这是一个抽象类，定义如下：  
``` java
/**
     * An OnScrollListener can be added to a RecyclerView to receive messages when a scrolling event
     * has occurred on that RecyclerView.
     * <p>
     * @see RecyclerView#addOnScrollListener(OnScrollListener)
     * @see RecyclerView#clearOnChildAttachStateChangeListeners()
     *
     */
    // wjc OnScrollListener
    // 重写这两个方法，在滚动的过程中会调用
    public abstract static class OnScrollListener {
        /**
         * Callback method to be invoked when RecyclerView's scroll state changes.
         *
         * @param recyclerView The RecyclerView whose scroll state has changed.
         * @param newState     The updated scroll state. One of {@link #SCROLL_STATE_IDLE},
         *                     {@link #SCROLL_STATE_DRAGGING} or {@link #SCROLL_STATE_SETTLING}.
         */
        public void onScrollStateChanged(RecyclerView recyclerView, int newState){}

        /**
         * Callback method to be invoked when the RecyclerView has been scrolled. This will be
         * called after the scroll has completed.
         * <p>
         * This callback will also be called if visible item range changes after a layout
         * calculation. In that case, dx and dy will be 0.
         *
         * @param recyclerView The RecyclerView which scrolled.
         * @param dx The amount of horizontal scroll.
         * @param dy The amount of vertical scroll.
         */
        public void onScrolled(RecyclerView recyclerView, int dx, int dy){}
    }
```
这两个方法的回调条件是： 
``` 
onScrollStateChanged : 滚动状态变化时回调
onScrolled : 滚动时回调
```  
**onScrollStateChanged(RecyclerView recyclerView, int newState)**  
1. RecyclerView 当前在滚动的 recyclerView
2. newState当前的滚动状态      

newState有三种值，分别是  
```  
/**
    * The RecyclerView is not currently scrolling.
    * @see #getScrollState()
    */
public static final int SCROLL_STATE_IDLE = 0;

/**
    * The RecyclerView is currently being dragged by outside input such as user touch input.
    * @see #getScrollState()
    */
public static final int SCROLL_STATE_DRAGGING = 1;

/**
    * The RecyclerView is currently animating to a final position while not under
    * outside control.
    * @see #getScrollState()
    */
public static final int SCROLL_STATE_SETTLING = 2;   
```
***onScrolled(RecyclerView recyclerView, int dx, int dy)**
1. recyclerView : 当前滚动的view
2. dx : 水平滚动距离
3. dy : 垂直滚动距离
