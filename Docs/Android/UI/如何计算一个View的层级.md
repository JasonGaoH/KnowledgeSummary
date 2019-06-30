```
int i = 0;
private void getParents(ViewParent view){
    if (view.getParent() == null) { 
        Log.v("tag", "最终==="+i); return;
    }
    i++;
    ViewParent parent = view.getParent(); 
    Log.v("tag", "i===="+i);
    Log.v("tag", "parent===="+parent.toString());
    getParents(parent); 
}
```

> 因为public abstract class ViewGroup extends View implements ViewParent
ViewGroup 是 ViewParent 的实现类，所以可以直接转， LinearLayout 是 ViewGroup 的子类