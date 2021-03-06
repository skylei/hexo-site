title: 關於Node.js下的MongoDB阻塞模式實現
date: 2013-03-29 09:31:38
tags: [ 老博客備份歸檔, Node.JS, MongoDB ]
category: 老博客備份歸檔
---

　　***注：本文僅爲我初學 Node.JS 的時候的稚嫩筆記。是從 [http://web.archive.org/](http://web.archive.org/) 扒回來的。現在看來已無多大參考價值，各位可以略過。我只是把它扒回來紀念一下而已，以記錄我的歷程。而那個相對應的 `SevenzJS` 也已經被遺棄***

## 背景

　　最近在做公司項目的一個模塊，主要用於 **JSON Api** 的傳輸，所以開發環境的目標就鎖定在了 **Node.js**。而這一塊的登陸用戶又是存在 **MongoDB** 裏面的，所以就有瞭如下的問題。

+ 網上的 Node.JS 框架都比較重型或者臃腫的，學了 Node 之後還需要學額外的東西。
+ 所以我就打算自己寫一個專注於 JSON Api 的快速開發框架，於是有了 SevenJS。
+ 問題出現了，雖然 Node.JS 極度推崇異步非阻塞模式，但是阻塞模式在平常開發中還是太常用了。

　　我們試想一下，如果我們有幾句MongoDB的查詢之類的，用node-mongodb-native來寫的話是這樣的：

```javascript
var client = new Db('test', new Server("127.0.0.1", 27017, {}));
var test = function (err, collection) {
    collection.insert({a:2}, function(err, docs) {
        collection.count(function(err, count) {
            test.assertEquals(1, count); });

            // Locate all the entries using find
            collection.find().toArray(function(err, results) {
            test.assertEquals(1, results.length);
            test.assertTrue(results[0].a === 2);

            // Let's close the db
            client.close();
        });
    });
};

client.open(function(err, client) {
    client.collection('test_insert', test);
});
```

　　各種嵌套回調有木有！這不是我們想要的，尤其是我的那個框架，因爲我的框架是流式的。

　　所以我就想有這樣的一種方案：

```javascript
var client = mongodb.connect();
var collection = mongodb.getCollection(client, "dbname");
var result = mongodb.find({ "foo" : "bar" });
```

　　使得這樣就能找出dbname表下的foo爲bar值的記錄了。

## 正題

　　出於這樣的想法，我在網上找遍了大江南北，除了 CNode 社區有人問到了類似的問題以外，再也找不到音信了，而且那裏也沒有一個好的回答。

　　不過這也正常，因爲 Node.js 官方本身就不推薦這麼做——他們認爲異步非阻塞是非常優雅的一件事情。

　　包括我在 Node.js 的 IRC 聊天室裏面問了這個問題，也有人是這麼回答我的：

> You can’t use a car as a boat. If you want a boat, use a boat.

　　言簡意賅，直截了當地說明 Node.js 是不支持這樣的，如果你想這樣做，就用 python 或者 ruby 去吧。

　　不過好在後來 IRC 裏面有人推薦了我一個模塊：[fibers](https://github.com/laverdet/node-fibers)。

　　有了這個模塊好啊，直接能用了有木有！

　　接下來就來講一下如何使用吧：

```javascript
function find(collection, selector, callback) {
    collection.find(selector).toArray(callback);
}

var Fiber = require('fibers');
var Future = require('fibers/future');
var wait = Future.wait;

Fiber(function(){
    var wrapper = Future.wrap(fund);

    /** 這裏就是正題了，我們假設已經獲取一個collection了 */
    var result = wrapper(collection, { "foo" : "bar" }).wait();
    console.log(JSON.stringify(result));
}).run();
```

　　這就是一個非常簡單的同步查詢 MongoDB 的例子了，實際上本質還是一個異步，注意到沒有，其實 `Fiber()` 內部的那個 `function` 本質上還是一個回調函數，只不過在這個回調函數裏面，裏面的所有 `callback` 都可以被同步了。不過我們只需要小動一些手腳就能加上這個外殼了。具體請參見 [sRouter.js](https://github.com/XadillaX/SevenzJS/blob/a0a0476000c492dd8e70c062cfa432f559edbd16/sevenz/sRouter.js) 約 121 行的外殼以及 [sMongoSync.js](https://github.com/XadillaX/SevenzJS/blob/a0a0476000c492dd8e70c062cfa432f559edbd16/sevenz/sMongoSync.js) 的實現，加上 [index.js](http://web.archive.org/web/20130726042859/https://github.com/XadillaX/SevenzJS/blob/a0a0476000c492dd8e70c062cfa432f559edbd16/actions/index.js) 中的查詢 demo。

## 結尾

　　所以說當我們做不到某件事的時候，多去IRC看看，多去社區混混，也多去找找模塊，要真沒有的話就只能自己豐衣足食了（我還沒到那水平，笑）。總之這次Fibers幫了我一個大忙。

　　最後，SevenzJS 歡迎 [Fork](https://github.com/XadillaX/SevenzJS)。