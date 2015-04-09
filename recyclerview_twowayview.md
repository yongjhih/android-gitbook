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

https://gist.github.com/yongjhih/d8db8c69190293098eec

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
        icon.setImageURI(IconViewHolder(Uri.parse(item)));
    }
}
```

基於 MVVM 概念，以 Avatar 頭像為例：


```java
// 1. listAdapter.createViewHolder(): 決定使用哪種 layout + viewHolder
// 2. ViewModel.builder(): 以及如何把原始資料降階至流通性資料格式
// 不需要瞭解具備 View 相關設定知識，如圖片顯示現在是用 SimpleDraweeView.setImageURI(Uri) 如何設定，還是用 ImageLoader.getInstance().displayImage(ImageView iv, String url) 該如何使用之類的知識。只要乖乖把資料按照流通資料介面填好就好。
// 如果畫面要微調，直接複製 layout 調整風格。
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ...
    

    //ListRecyclerAdapter<String, IconViewHolder> listAdapter = ListRecyclerAdapter.create();
    //listAdapter.getList().add("http://example.com/a.png");
    ListRecyclerAdapter<AvatarViewModel, AvatarViewHolder> listAdapter = ListRecyclerAdapter.create();
    
    listAdapter.createViewHolder((parent, viewType) -> new IconViewHolder(LayoutInflater.from(context).inflate(R.layout.item_icon, parent, false)));

    listAdapter.getList().add(AvatarViewModel.builder().icon("http://example.com/a.png").name("Andrew Chen").build());
    for (User user : getUsers()) { // 新增其他使用者
        listAdapter.getList().add(AvatarViewModel.builder().icon(user.getPicture()).name(user.getDisplayName()).build());
    }
    // via RxJava
    // listAdapter.getList().addAll(Observable.from(getUsers()).map(user -> AvatarViewModel.builder().icon(user.getPicture()).name(user.getDisplayName()).build()).toList().toBlocking().single());
    
    ...
}
```

```java
// 只需具備 View 相關知識，設定流通性資料
// 不需要瞭解原始資料來源格式(不管是從 Facebook/Parse 還是 local sqlite 或 local SharePreferences)，只瞭解流通性基本資料格式即可。
public class AvatarViewHolder extends BindViewHolder<AvatarViewModel> {
    @InjectView(R.id.icon)
    public SimpleDraweeView icon;
    @InjectView(R.id.text1)
    public TextView name;

    @Override
    public void onBind(int position, AvatarViewModel item) {
        //icon.setImageURI(IconViewHolder(Uri.parse(item)));
        icon.setImageURI(Uri.parse(item.icon()));
        name.setText(item.name());
    }
}
```

```java
// 提供流通性基本資料的介面
@AutoParcel
public abstract class AvatarViewModel implements Parcelable {
    public abstract String icon();
    public abstract String name();
    
    public static Builder builder() {
        return new AutoParcel_AvatarViewModel.Builder();
    }   

    @AutoParcel.Builder
    public interface Builder {
        public Builder icon(String x);
        public Builder name(String x);
        public AvatarViewModel build();
    }
}
```

* http://stackoverflow.com/questions/26649406/nested-recycler-view-height-doesnt-wrap-its-content/28510031