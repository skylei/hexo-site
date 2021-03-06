title: 關於JavaScript中callback函數的this指針重定義
date: 2013-07-15 14:20:29
tags: [ 老博客備份歸檔, Javascript ]
category: 老博客備份歸檔
---

　　最近在寫 **NBUT Virtual Judge** 的內核框架，用的又是 **Node.JS** 了，把它當作一個本地運行的腳本不斷進行輪詢。

　　衆所周知JS中的一個精髓就是異步回調。

　　所以在我自己寫的框架中也經常會出現類似於下面的代碼：

```javascript
foo.bar(a, b, function(){});
```

　　總而言之就是寫一個函數，這個函數將會調用一個回調函數。

　　但是問題出現了：在那個回調函數 `function` 中，你如果使用了一個 `this` 指針的話，它將會指向根，而不是 `foo` 的本體。

　　那麼如果我們想在 `function` 中也用 `this` 來指代這個 `foo` 對象該怎麼辦呢？

　　結果還是IRC有用。本人跑 **Node.JS** 的 **IRC** 上問了這個問題，結果有人就這樣回覆我了：

> 13:07 &lt;shama> xadillax: foo(a, b callback.bind(foo))
>
> 13:10 &lt;olalonde> foo (a, b fn) { fn = fn.bind(this); …. }

　　然後還很熱心地給了我個網址：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

　　總之最後得出的結論就是說：

　　你只要給你的 `callback` 函數指定一個 `this` 指針即可。

　　如：

```javascript
var cb = callback.bind(foo);
foo.bar(a, b, cb);
```

　　這樣就能在回調函數中使用foo來作爲其this指針了。
