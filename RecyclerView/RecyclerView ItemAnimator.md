# RecyclerView ItemAnimator
自定义的动画需要继承自ItemAnimator，它有以下这些方法要去继承：

``` java
    @Override
    public boolean animateDisappearance(RecyclerView.ViewHolder viewHolder, ItemHolderInfo preLayoutInfo, ItemHolderInfo postLayoutInfo) {
        return false;
    }

    @Override
    public boolean animateAppearance(RecyclerView.ViewHolder viewHolder, ItemHolderInfo preLayoutInfo, ItemHolderInfo postLayoutInfo) {
        return false;
    }

    @Override
    public boolean animatePersistence(RecyclerView.ViewHolder viewHolder, ItemHolderInfo preLayoutInfo, ItemHolderInfo postLayoutInfo) {
        return false;
    }

    @Override
    public boolean animateChange(RecyclerView.ViewHolder oldHolder, RecyclerView.ViewHolder newHolder, ItemHolderInfo preLayoutInfo, ItemHolderInfo postLayoutInfo) {
        return false;
    }

    @Override
    public void runPendingAnimations() {

    }

    @Override
    public void endAnimation(RecyclerView.ViewHolder item) {

    }

    @Override
    public void endAnimations() {

    }

    @Override
    public boolean isRunning() {
        return false;
    }
```
在这些方法中，有几个是比较总要的，有：

|函数|作用|
|-----|-----|
|animateDisappearance|当RecyclerView中的item在屏幕上由可见变为不可见时调用此方法|
|animateAppearance|当RecyclerView中的item显示到屏幕上时调用此方法|
|animateChange|当RecyclerView中的item状态发生改变时调用此方法(notifyItemChanged(position))|
|runPendingAnimations|统筹RecyclerView中所有的动画，统一启动执行|

