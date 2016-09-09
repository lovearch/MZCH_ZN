# AdsBasePartitionAdapter   
启示就是继承了BaseAdapter。重写的接口几乎没写。
``` java   
ublic class AbsBasePartitionAdapter extends BaseAdapter {

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

}
```  