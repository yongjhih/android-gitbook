# RecyclerView

基本使用方法:

Before:

```java
@InjectView(R.id.icons)
RecyclerView icons;

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final List<String> list = new ArrayList<>(Arrays.asList("http://example.com/a.png")));
    
    RecyclerAdapter<IconViewHolder> listAdapter = new RecyclerAdapter<>() {
        @Override
        public IconViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new IconViewHolder(LayoutInflater.from(context).inflate(R.layout.item_icon, parent, false)));
        }
        
        @Override
        public void onBindViewHolder(IconViewHolder viewHolder, int position) {
            viewHolder.icon.setImageURI(Uri.parse(list.get(position)));
        }
    }
    
    icons.setLayoutManager(new LinearLayoutManager(activity));
    icons.setAdapter(listAdapter);
    
    list.add("http://example.com/b.png");
    listAdapter.notifyDataSetChanged();
}
```

```java
public class IconViewHolder extends BindViewHolder<String> {
    @InjectView(R.id.icon)
    public SimpleDraweeView icon;

    public IconViewHolder(View itemView) {
        super(itemView);
        ButterKnife.inject(this, itemView);
    }
}
```
有資料外漏的情形。

改用筆者簡單寫的 `ListRecyclerAdapter<T, VH>`

After:

```java
@InjectView(R.id.icons)
public RecyclerView icons;

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    ListRecyclerAdapter<String, IconViewHolder> listAdapter = ListRecyclerAdapter.create();
    listAdapter.getList().add("http://example.com/a.png");
    
    listAdapter.createViewHolder((parent, viewType) -> new IconViewHolder(LayoutInflater.from(context).inflate(R.layout.item_icon, parent, false)));
    
    icons.setLayoutManager(new LinearLayoutManager(activity));
    icons.setAdapter(listAdapter);
    
    listAdapter.getList().add("http://example.com/b.png");
    listAdapter.notifyDataSetChanged(); // TODO hook List.add(), List.addAll(), etc. modifitable operations
}
```

IconViewHolder:

```java
public class IconViewHolder extends BindViewHolder<String> {
    @InjectView(R.id.icon)
    public SimpleDraweeView icon;

    public IconViewHolder(View itemView) {
        super(itemView);
        ButterKnife.inject(this, itemView);
    }

    @Override
    public void onBind(int position, String item) {
        icon.setImageURI(IconViewHolder(Uri.parse(item));
    }
}
```

* https://gist.github.com/yongjhih/d8db8c69190293098eec
* http://stackoverflow.com/questions/26649406/nested-recycler-view-height-doesnt-wrap-its-content/28510031