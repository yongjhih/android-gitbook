# flow + mortar

從 2014/01 Square 部落格初次登板 [mortart and flow](https://corner.squareup.com/2014/01/mortar-and-flow.html) 的開頭可以得知，
一開始要解決的問題是 Fragment 換頁問題。

從 Android 4.0 之後大量改成 Fragment 導致一個 Activity 要掌管 Fragment transaction 換頁。

在一般用途上，筆者偷懶直接用 ViewPager + FragmentStatePagerAdapter + ViewPager.Transformer + 特製的 PagerIndicator ，基本上堪用。

## FragmentMaster

另一種 ViewPager + FragmentStatePagerAdapter + ViewPager.Transformer 的整合方案。

## See Also

* https://github.com/square/flow/
* https://corner.squareup.com/2014/01/mortar-and-flow.html
* https://www.bignerdranch.com/blog/an-investigation-into-flow-and-mortar/
* https://github.com/WeMakeBetterApps/MortarLib
