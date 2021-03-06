title       : 一起擼Node.JS（負叄）——概述
date        : 2013-08-13
category    : NodeJS
tags        : [ Node.js, JavaScript, 一起擼Node.JS ]
---

　　本系列教程主要是寫給我帶的那幫熊孩子們看的。我自己的 **Node.js** 水平半斤八兩，措辭之中也免不了有自己錯誤的理解，會誤人子弟。但是對於初學者來說，某些自己助記的理解還是可取的。有些概念性的錯誤可以等他們進一步深入研究之後再自行更正。

　　由於那幫人大多還處於使用 **M$ Windows** 的令人不愉快的階段，所以本教程將會退而求其次，使其在 **Cygwin** 中模擬 **linux** 的命令（Windows的bat腳本實在是讓人不敢恭維）。以及在這裏會講述一些 **Git** 操作的初步。當然，如果你已經在使用 **linux** 進行開發的話，可以跳過前面一堆令人感到厭煩的環境配置章節。或者你在使用 **M$ Windows** 但卻不想改變自己的腳本習慣的話，也可以選擇性地跳過一些章節和步驟。


<!-- 我是小小分割符 -->
## Node.JS是什麼？

很多人都知道JS是一門語言，而且是一門腳本語言，其全稱就是 **JavaScript**，而且與所謂的 **Java** 沒有一個屁的關係。

### 前端 JavaScript

在好多年前，**JavaScript** 是網頁的一個寄生蟲，它必須依賴於網頁的瀏覽器中才能執行，並且作爲網頁的一部分，以

{% code html %}
<script type="text/javascript">
//blahblah...
</script>
{% endcode %}

標籤進行包含，這樣才能提供其上下文環境。或者說將其單獨寫入一個 `*.js` 文件中，並且在網頁裏以

{% code html %}
<script src="foo/bar.js" type="text/javascript"></script>
{% endcode %}

的形式將其包含進來。

但總而言之，**JavaScript** 只是寄生在網頁裏面的一隻小小可憐蟲罷了。它的作用無非就是使網頁的交互性更強，頁面效果更多而已。

後來，這幫不甘寂寞的人類將 **JavaScript** 從網頁（或者說前端）的帝國中獨立了出來（小心快遞），於是就出現了 **CommonJS**。

### CommonJS

**CommonJS** 其實不是一門新的語言，甚至都不能說它是一個新的解釋器——實際上它只是一個概念或者是一個規範。

在這個規範中，它定義了很多 **API** ，講通俗點或者直截了當點就是函數啊類啊什麼的，而這些 **API** 是爲那些普通應用程序（Native App）而非瀏覽器應用使用。它的終極目標就是提供一個類似於 **Python**、**Ruby** 之類的腳本一樣的標準庫，開發者可以用這樣的東西一樣來做到 **Python**、**Ruby** 能做到的事，而非僅僅侷限於網頁中的效果或者功能實現，它也可以跑在本地。

所以說下面的事情對於 **JavaScript** 來說不再是夢：

  + 服務端JavaScript應用
  + 命令行工具
  + 圖形界面應用
  + 混合應用（Titanium、Adobe AIR等）

那麼，它具體彌補了 **前端JavaScript** 的哪些空白呢？其實這也涉及了很多 **前端JavaScript** 所沒有涉及的東西，如二進制、編碼、IO、文件、系統、斷言測試、套接字、事件隊列、Worker、控制檯等等。

關於 **CommonJS** 的更進一步瞭解可以翻閱一下其 **[Wiki](http://wiki.commonjs.org/wiki/CommonJS)**。

### Node.JS

上面講了那麼多，卻始終停留在“規範”這個層面上。而 **Node.JS** 的出現便是讓 **CommonJS** 成爲了現實。

這裏要大家明確的一點的就是 **Node.JS** 並不是一門新的語言，它的語言還是 **JavaScript** ，硬要說是一門新的語言那也應該是 **Common JavaScript**。**Node.JS** 只是 **CommonJS** 的一個[解釋器](http://zh.wikipedia.org/wiki/%E8%A7%A3%E9%87%8A%E5%99%A8)罷了。

它是基於 **Google** 的 **V8虛擬機**(Chrome瀏覽器所使用的JavaScript執行環境) 的一個解釋器。

很多人印象中的概念還是沒能擺脫 **前端JavaScript** 的陰霾，認爲 **JavaScript** 就是做網站的， **Node.JS** 也是如此。

包括本人在 **[cnodejs.org](http://cnodejs.org/)** 中看到的帖子大多也都是講 **Node.JS** 如何如何做網站（服務端）云云，如何如何使用 **Express** 模塊來搭建一個網站云云。

> 這是一個誤區。

**PHP** 還能用 **[PHP-CLI](http://www.php-cli.com/)** 來寫個腳本放本地跑呢，**Node.JS** 更是可以寫任何程序。雖然這麼講有些誇大了，但是我這麼說的理由是希望大家能擺脫這麼一個誤區。

舉個簡單的例子吧，大家都是搞過 **ACM** 的孩子了，總對終端窗口的輸入輸出有一定感覺了吧。現在給我以最快速度碼一個 ***[A + B Problem](http://acm.nbut.edu.cn/problem/view.xhtml?id=1000)*** 給我看看。

輕車熟路，我知道。但是你們現在做的事用 **Node.JS** 同樣能做到。

{% code javascript %}
process.stdin.resume();
process.stdin.setEncoding("utf8");
process.stdin.on("data", function(chunk) {
    var datas = chunk.trim().split("\n");
    for(var i = 0; i < datas.length; i++) {
        var ab = datas[i].trim().split(" ");
        var a = parseInt(ab[0]);
        var b = parseInt(ab[1]);
        console.log(a + b);
    }
});
{% endcode %}

由於~~我們學校~~我的前任學校OJ不支持 **Node.JS**，所以請你們移步到 **[AIZU OJ](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1000)** 去把上面的代碼交過去看看結果看。

> **注意**：語言要選擇 **JavaScript**。

怎麼樣，同樣能過題的對吧？

## 小結

上面對這些東西做了個簡單的介紹，我需要你們知道的東西很簡單：

  1. **Node.JS** 是一個腳本解釋器，用的語言是 **JavaScript**。
  2. **Node.JS** 功能很強大，不是隻能拿來做網站的，眼光放開闊些。
  3. 給我好好學。

## 番外

> 有個碼畜老了，想學學書法來修身養性。當他展開宣紙，猶豫了半天之後，終於揮毫潑墨，在紙上龍飛鳳舞寫下幾個大字：
>
> > ***Hello World***

雖然這一篇文章沒有講到任何 **Node.JS** 的語法，但是還是可以讓你們練練書法的。

**C語言** 的標準輸出函數是 `printf`，而 **Node.JS** 的標準輸出則是：

{% code javascript %}
console.log("blahblah...");
{% endcode %}

好的，即使沒有裝上 **Node.JS** 環境也阻止不了我們向世界問好。

打開 **[IDEOne](http://ideone.com/)**，將你的 `Hello World` 貼到編輯框中，然後在左側的語言欄裏面選中 **Node.JS** ，點擊送出，你就能看到你的第一個 **Node.JS** 程序的運行結果了。

***To be continued...***
