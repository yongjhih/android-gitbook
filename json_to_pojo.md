# json to POJO

* Gson
* Jackson
* Instagram/ig-json-parser
* LoganSquare

基本上 ig-json-parser、LoganSquare 是基於 Jackson 衍伸的，編譯時期的欄位處理，搭配 Jackson stream parsing 達成。

Gson 而是執行時期作欄位處理，所以肯定速度上比較慢。

而 LoganSquare 啟發於 Instagram/ig-json-parser 所以有較好的包裝。


## LoganSquare

```java
// String jsonString:
//
// {
//   name: "Andrew",
//   id: 1
// }
User user = LoganSquare.parse(jsonString, User.class);
System.out.println("name: " + user.name);
System.out.println("id: " + user.id);
```

```
@JsonObject
public class User {
    @JsonField
    public String name;

    @JsonField
    public int id;
}
```

## See Also

* http://instagram-engineering.tumblr.com/post/97147584853/json-parsing