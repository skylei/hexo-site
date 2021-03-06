title: Storm 中 Clojure 的 Prepare Bolt 實現
date: 2014-08-04 11:50:21
category: Programming
tags: [ Storm, Clojure, Huaban ]
---

## 起因

　　Storm 中的 Bolt 都是通過 Nimbus 這個服務將序列化好的 Bolt 斷章取義地發到各個 worker 中。所以，任何在 bolt 之外你自認爲加載期間初始化計算好的上下文環境並不會被打包上去，Java 我不懂也不知道，但是至少在 Clojure 這個類的概念被淡化的 LIST 方言中，你要做的就是把所有跟 bolt 初始化計算相關的代碼放到其 `prepare` 的代碼當中去。

　　你想一下，當你在文件加載的時候初始化了一個 MongoDB 鏈接，這個鏈接總不能被序列化到遠程去吧？所以說辦法就是把 bolt 搞上去之後，bolt 自動去初始化一個鏈接——這就是 `prepare` 的作用了。

　　說白了，這個還是我們在 ***Suwako*** 當中踩到的坑。

## 做法

　　大致的骨架如下：

```clojure
(defbolt bolt [...] {:prepare true}
 [...]
 (let [...]
  (bolt
   (prepare [...]
    (...))
   (execute [tuple]
    (...))))
```

　　首先就是 `{:prepare true}` 代表了它是一個需要初始化的 Bolt。

　　然後在 `(bolt)` 的作用域之內有兩個 form——`prepare` 和 `execute`。

　　其中 `prepare` 就是你要初始化的語句了。舉個例子，我們讓這裏面初始化一個 [Monger](http://clojuremongodb.info/)，於是我們要在 `let` 裏面定義一個用於鏈接的 `atom {}`。

```clojure
(defbolt bolt ["..."] {:prepare true}
 [conf context collector]
 (let [conn (atom {})
       db (atom {})]
   (bolt
    (prepare [conf context collector]
     (reset! conn (mg/connect ...))
     (reset! db (mg/get-db @conn ...)))
    (execute [tuple]
     (...)))))
```

　　這樣一來，當 Bolt 被 Nimbus 打包傳到各個 worker 之後，Bolt 執行起來的時候會自動執行 `prepare` 當中的代碼，即初始化 MongoDB 的鏈接，並且將其賦值給 `conn` 和 `db` 兩個 atom。

　　那麼，我們就能在本體 `execute` 當中使用 `@conn` 和 `@db` 來使喚 MongoDB 了。

## 思考

　　可能很多人不解，不是說儘量保持 LISP 語系當中值的不變性的麼？

　　其實不變性只是爲了提高程序在運行時的效率——而事實上是，上面那段代碼並沒有在運行時去做變量。

　　雖然說這麼說有點牽強，但是的確就是這個意思——因爲我們是在程序執行真正有用的好邏輯的時候沒有去改變一些值，相反只是在 Bolt 啓動的時候做一些變量的操作。

　　換句話說，雖然嚴謹的講那個時候是算運行時，但是在運行時裏面我們卻可以把它歸類爲預處理——這一類東西反正程序還沒真正開始跑有用的東西，效率慢一點無所謂，而且就初始化這麼屁大點事兒能有多少影響？

　　效率和效果之間權衡上面的還是要仁者見仁智者見智了。

## 小結

　　本以爲 `Suwako` 終於可以暫時告一段落了，緊要關頭居然還是阻塞了。

　　說多都是淚，不說了，找 Bug 去了。

![泄矢諏訪子](suwako.jpg)
