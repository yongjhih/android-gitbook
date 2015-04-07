# json to POJO

* Gson
* Jackson
* Instagram/ig-json-parser
* LoganSquare

基本上 ig-json-parser、LoganSquare 是基於 Jackson 衍伸的，編譯時期的欄位處理，搭配 Jackson stream parsing 達成。

Gson 而是執行時期作欄位處理，所以肯定速度上比較慢。

而 LoganSquare 啟發於 Instagram/ig-json-parser 所以應該會有較好的包裝。


## See Also

* http://instagram-engineering.tumblr.com/post/97147584853/json-parsing