title: 開發測試時給 Kafka 發消息的 UI 發送器——Mikasa
date: 2014-07-30 10:14:29
category: Programming
tags: [ Kafka, Mikasa, 花瓣 ]
---

## 起 (灬ºωº灬)

　　說來話長，自從入了花瓣，整個人就掉進連環坑了。

　　後端元數據採集是用 Storm 來走拓撲流程的，又因爲 @[Zola](http://weibo.com/zolazhou) 不是很喜歡 Java，所以退而求其次選擇了 Clojure，所以正在苦逼地學習 Clojure 和 Storm 中。

　　目前來說外面的 Storm 拓撲的 Spout 是從 Kafka 中流入數據的。但是我們要給 Kafka 發送測試數據的時候，就需要跑到 Kafka 的測試服務器打開它的一個發送腳本進去發送，非常蛋疼；要麼就是直接通過特定的發送業務邏輯代碼測試，沒有一個稍微泛一點的測試用發數據工具，於是 Mikasa 誕生了。

## 承 (ﾟ3ﾟ)～♪

　　講到 Mikasa 名字的來源，實際上看過『巨人』都知道，八塊腹肌的三爺。

　　這裏小爆料一下，又拍雲和花瓣（都是同宗）的項目名很大部分都是以海賊王的角色命名的——尤其是又拍雲更是喪心病狂。不過這讓我這個僞·二次元的小夥伴異常欣喜，因爲我也能用各種啪啪啪來命名我的角色了。比如我的第一個 Storm 相關的項目就叫 Suwako，即諏訪子大人，因爲腦子需要各種跳，於是就對諏訪子大人這位青蛙之神各種膜拜。

　　至於這個發射器爲什麼要用三爺呢？因爲三爺相當於先鋒軍哇！

![Mikasa](mikasa.jpeg)

　　這裏的 Kafka 依賴用了搜狐小夥伴 @[Crzidea](http://weibo.com/crzidea) 他們團隊寫的模塊。

## 轉 (ㄏ￣▽￣)ㄏ   ㄟ(￣▽￣ㄟ)

　　於是，話也不多說，直接上 repo 吧。在公司內網的 gitlab 裏面有一份，還有一個 repo 在 [GitHub](https://github.com/) 上。

> [點我](https://github.com/XadillaX/mikasa)

### Download || Clone

　　如果要直接下載的話就用這個鏈接：

> [https://github.com/XadillaX/mikasa/archive/master.zip](https://github.com/XadillaX/mikasa/archive/master.zip)

　　如果要克隆的話就：

```shell
$ git clone https://github.com/XadillaX/mikasa.git
```

### Setup

　　直接安裝一下依賴：

```shell
$ npm install
```

### Configuration

　　接下去就是簡單的配置一下了，其實就是配置下配置文件。由於是快速開發，直接用了自己之前的 [Exframess](https://github.com/XadillaX/exframess) 框架，所以很多無用代碼也懶得刪了。

#### config/server.js

　　這裏其實別的也不用動，主要是修改下端口即可。

#### config/kafka.js

　　這裏修改一下 Kafka 的 `Connection String` 就好了。

### Start up

　　最後啓動服務即可。

```shell
$ node app.js
# or
$ pm2 app.js
# or some other's
```

## 合 (ﾉ◕ヮ◕)ﾉ*:･ﾟ✧

　　最後的效果是這樣的：

![Preview](mikasa-preview.png)

　　只要在 Topics 欄裏面輸入你要發送的 Topic，然後再下面的消息欄裏面輸入你要傳的消息（字符串），最後點擊 `Send` 即可將你的測試消息發進 Kafka 中去了。

> 託大家的福，今天我的 Suwako 整個邏輯終於跑通了，撒花！ε٩(๑> ₃ <)۶з
