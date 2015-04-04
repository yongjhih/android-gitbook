# Image Loader - Android-Universal-Image-Loader, picasso, glide, fresco

為什麼要 Image Loader ，什麼是 Image Loader ？

1. 顯示快取 - 首先，為了再次顯示一樣的圖片時，可以快速顯示，所以我們會把 bitmap 暫存在記憶體 memory cache 。那在有限的 memory cache ，我們要如何管理 ? 常見的是 LRU cache 策略，memory cache 有限，最近看過的優先留下來，其他捨去。
2. 儲存快取 - 再來，網路來的圖片、網址圖片，下載儲存快取管理 - Disk Cache。

在 fresco 出現之前，我會比較推崇 AUIL(Android Universal Image Loader)，接著 glide 、picasso 。

追求小 code size 可以選 picasso ，輕簡。

AUIL 是因為在記憶體、儲存空間的快取策略都可以訂製

只有 fresco 需要更換 layout class 原因是因為它為了效能，操作較低階的畫布。

## Fresco - facebook

## picasso - square

## glide

glide 目前看起來 google 有些演講有提過