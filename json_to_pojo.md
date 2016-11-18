# json to POJO

* Gson
* Jackson
* Instagram/ig-json-parser
* LoganSquare

基本上 ig-json-parser、LoganSquare 是基於 Jackson 衍伸的，編譯時期的欄位處理，搭配 Jackson stream parsing 達成。

Gson 而是執行時期作欄位處理，自然有改善的空間。

LoganSquare 啟發於 Instagram/ig-json-parser 而有較好的包裝。


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

```java
@JsonObject
public class User {
    @JsonField
    public String name;

    @JsonField
    public int id;
}
```

先撇除物件化描述，可以先探究，為什麼 JSONObject for loop 去爬會比較慢呢？

一個初步的原因是因為先要把 json string 塞成 JSONObject ，表示已經繞完過一輪了。然後你再次繞一輪 JSONObject 就已經是兩輪了。

而 Jackson 提供的 streaming 就是正在繞的當下就會呼叫 callback 了。

## flatbuffers & protobuf

They are lightweight serialized data format.

The retrofit supports protobuf.

..

## See Also

* http://instagram-engineering.tumblr.com/post/97147584853/json-parsing
