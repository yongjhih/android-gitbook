# RecyclerView, TwoWayView


```java
ListRecyclerAdapter<String, IconViewHolder> listAdapter = ListRecyclerAdapter.create();
listAdapter.createViewHolder((parent, viewType) -> new IconViewHolder(LayoutInflater.from(context).inflate(R.layout.item_icon, parent, false)));
icons.setLayoutManager(new LinearLayoutManager(activity));
icons.setAdapter(listAdapter);
listAdapter.getList().addAll(Arrays.asList("http://example.com/a.png"));
listAdapter.notifyDataSetChanged(); // TODO hook List.add(), List.addAll(), etc. modifitable operations
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
            avatar.setImageUIconViewHolderRI(Uri.parse(item));
        }
    }
}
```

https://gist.github.com/yongjhih/d8db8c69190293098eec
