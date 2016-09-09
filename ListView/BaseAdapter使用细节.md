# BaseAdapter  
## getItem()  
``` java   
    @Override
    public Object getItem(int position) {

    . ...
    }
```   
官方解释是Get the data item associated with the specified position in the data set.即获得相应数据集合中特定位置的数据项.
通过查看源代码发现，getItem方法不是在Baseadapter类中被调用的，而是在Adapterview中被调用的。adapterView类中,我们找到了如下方法,  

``` java   
    public Object getItemAtPosition(int position) {
        T adapter = getAdapter();
        return (adapter == null || position < 0) ? null : adapter.getItem(position);
    }
```   
那么getItemAtPosition(position) 又是什么时候被调用？
答案：它也不会被自动调用，它是用来在我们设置setOnItemClickListener、setOnItemLongClickListener、setOnItemSelectedListener的点击选择处理事件中方便地调用来获取当前行数据的。

## getItemId(int position)  
它返回的是该postion对应item的id,adapterview也有类似方法：  
``` java   
    public long getItemIdAtPosition(int position) {
        T adapter = getAdapter();
        return (adapter == null || position < 0) ? INVALID_ROW_ID : adapter.getItemId(position);
    }
```   
不同getItem的是，某些方法（如onclicklistener的onclick方法）有id这个参数，而这个id参数就是取决于getItemId()这个返回值的。
