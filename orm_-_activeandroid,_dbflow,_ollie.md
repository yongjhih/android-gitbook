# Orm - ActiveAndroid, DBFlow, Ollie

ORM, Object Relational Mapping

以往的 Orm lib 其實有些瓶頸，
例如：query 回來的都是 `List<Model>` ，這象徵著 query 萬一很多筆，Orm 必須全部繞完把每個都組成 Model ，真正有用過會發現會非常的緩慢，就算用 Transaction 包起來加速，也無法有效改善。這部份我們團隊寫了一個 CursorList 來作 Lazy 是等用到才去組，效果很好。
這個概念 sprinkles Orm lib 也有發現，所以做了跟我們團隊一樣的事情。(DBFlow 有採納)

sprinkles 還有作自動升級程式也就是在 Model 新增欄位不用自己寫 alter table 。這點 DBFlow 應該也有採納。

目前我們是 ActivieAndroid + CursorList + EventBus 來作 Observing model，是由於開發時沒有 sprinkles 與 DBFlow，到現在已經沒法方便轉過去了。

另外 DBFlow 很巧妙的使用 `@interface Table RetentionPolicy.SOURCE` 編譯時期產生一個承載欄位名稱的子類別, 所以可以用 `ProfileModel$Table.DISPLAYNAME` 取出 "displayname" 欄位名稱，來有效減少 hard code 。

```java
ProfileModel extends BaseModel {
    @Column String displayname;
}
```

緩慢確實有一部份是 reflection 所造成，而 reflection 緩慢的主因在於解析 annotation 。所以通常我們會把 NoteModel.class 所有的 column attribute 都存到一個 annotations cache map 避免重複解析 annotation 。所以只會慢在建立 annotations cache map 那第一次而已，Ollie ModelAdapter 主要解決的首重，就是這個廣泛存在於 Orm lib 初始化過慢問題。

```java
List<Note> notes = new Select().from(Note.class).execute(); // huge notes
```

在萬筆資料的存取時，造成的緩慢是在於建立萬筆 Model 的累計時間，而不在於 reflection annotations (already in cache)。

1. 試想為什麼 SQLiteDatabase 存取時，回傳的不是 List<?> 而是 Cursor。 
2. 試想 cursor 所有資料已經都從 DB 拉出來了？還是當跑 cursor.next(), cursor.get*() 才去拉資料？(DB 應該只是快速扼要的把數量與索引準備好，就馬上拋回 cursor ，當 cursor.next() 才把該筆資料找出來 ，cursor.getString() 才動用 io 去拉資料。)
3. 再試想另一個：為什麼需要 CursorAdapter 而不是 loop cursor 塞進 ArrayAdapter 就好。 `// while (cursor.moveToNext()) notes.add(Note.load(cursor)); new ArrayAdapter(notes);`

問題都指向同一個 -> 因為大量的 io (loop cursor)以及 建立大量的物件(Note.load(cursor)) 累計起來過慢。這些資料通常運算的部份已經在 query 條件過程中結束，僅為了拿出來顯示，但是這麼大量的資料全都要顯示出來的機會，其實很低，就這個狀況，可以 lazy 概念緩解，指到哪拿到哪。SQLiteCursor 基本上就是一種 lazy 的存在。

整理問題與解法：

1. 解析 annotation -> cache
2. loop cursor -> lazy
3. load from cursor -> lazy

量測方法：

2. 把 com.activeandroid.util.SQLiteUtils.processCursor() do {} while (cursor.moveToNext()) 做空，單純測量 loop cursor 的耗時
3. 量測 Model.loadFromCursor()
