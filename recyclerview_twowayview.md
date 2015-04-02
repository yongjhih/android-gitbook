# RecyclerView

Before:

```java
@InjectView(R.id.icons)
public RecyclerView icons;

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final List<String> list = new ArrayList<>(Arrays.asList("http://example.com/a.png")));
    
    RecyclerAdapter<IconViewHolder> listAdapter = new RecyclerAdapter<IconViewHolder>() {
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
    
    list.add("http://example.com/a.png");
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

使用筆者簡單寫的 `ListRecyclerAdapter<T, VH>`

After:

```java
@InjectView(R.id.icons)
public RecyclerView icons;

public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    ListRecyclerAdapter<String, IconViewHolder> listAdapter = ListRecyclerAdapter.create();
    
    listAdapter.createViewHolder((parent, viewType) -> new IconViewHolder(LayoutInflater.from(context).inflate(R.layout.item_icon, parent, false)));
    
    icons.setLayoutManager(new LinearLayoutManager(activity));
    icons.setAdapter(listAdapter);
    
    listAdapter.getList().addAll(Arrays.asList("http://example.com/a.png"));
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
    public void bind(int position, String item) {
        icon.setImageUIconViewHolderRI(Uri.parse(item));
    }
}
```

https://gist.github.com/yongjhih/d8db8c69190293098eec