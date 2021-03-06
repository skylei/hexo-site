title: 圖片主題色提取算法小結
date: 2014-09-17 11:34:54
tags: [ Algorithm, Theme Color ]
---

> 所謂主題色提取，就是對於一張圖片，近似地提取出一個調色板，使得調色板裏面的顏色能組成這張圖片的主色調。

　　以上定義爲我個人胡謅的。

　　大家不要太把我的東西當成嚴謹的文章來看，很多東西什麼的都是我用我自己的理解去做，並沒有做多少考證。

　　解析中都會以 Node.js 來寫一些小 Demo。

## 引子

　　寫該文章主要是爲了對我這幾天對於『主題色提取』算法研究進行一個小結。

　　花瓣網需要做一件事，就是把圖片的主題色提取出來加入到花瓣網搜索引擎的索引當中，以供用戶搜索。

　　於是有了一個需求：提取出圖片中在某個規定調色板中的顏色，加入到搜索引擎。

　　接下去就開始解析兩種不同的算法以及在這種業務場景當中的應用。

## 算法解析

### ~~魔法數字法~~

　　這個算法大家可以忽略，可能是我使用的姿勢不對，總之提取出來（也許它根本就不是這麼用的）的東西錯誤很大。

　　不過看一下也好開闊下眼界，尤其是我這種想做遊戲又不小心掉進互聯網的坑裏的蒟蒻來說。

　　首先該算法我是從[這裏](http://dev.gameres.com/Program/Visual/Other/256color.htm)找到的。想當年我還是經常逛 [GameRes](http://www.gameres.com/) 的。ヾ(;ﾟ;Д;ﾟ;)ﾉﾞ

　　然後輾轉反側最終發現這段代碼是提取自 [Allegro](https://github.com/liballeg/allegro5/blob/4.3/src/color.c#L268-L328) 遊戲引擎。

　　具體我也就不講了，畢竟找不到資料，只是粗粗瞄了眼代碼裏面有幾個魔法數字（在遊戲和算法領域魔法數字倒是非常常見的），也沒時間深入解讀這段代碼。

　　我把它翻譯成了 Node.js，然後放在了 [Demo](https://github.com/XadillaX/theme-color-test/blob/master/version1/magicnumber.js) 當中。大家有興趣可以自己去看看。

### 八叉樹提取法

　　這個算法在顏色量化中比較常見的。

> 該算法最早見於 1988 年，***M. Gervautz*** 和 ***W. Purgathofer*** 發表的論文***《A Simple Method for Color Quantization: Octree Quantization》***當中。其時間複雜度和空間複雜度都有很大的優勢，並且保真度也是非常的高。

　　大致的思路就是對於某一個像素點的顏色 **R / G / B** 分開來之後，用二進制逐行寫下。

　　如 `#FF7800`，其中 **R** 通道爲 `0xFF`，也就是 `255`，**G** 爲 `0x78` 也就是 `120`，**B** 爲 `0x00` 也就是 `0`。

　　接下去我們把它們寫成二進制逐行放下，那麼就是：

```
R: 1111 1111
G: 0111 1000
B: 0000 0000
```

　　**RGB** 通道逐列黏合之後的值就是其在某一層節點的子節點編號了。每一列一共是三位，那麼取值範圍就是 `0 ~ 7` 也就是一共有八種情況。這就是爲什麼這種算法要開八叉樹來計算的原因了。

　　舉個例子，上述顏色的第一位黏合起來是 `100(2)`，轉化爲十進制就是 4，所以這個顏色在第一層是放在根節點的第五個子節點當中；第二位是 `110(2)` 也就是 6，那麼它就是根節點的第五個兒子的第七個兒子。

　　於是我們有了這樣的一個節點結構：

```javascript
var OctreeNode = function() {
    this.isLeaf = false;
    this.pixelCount = 0;
    this.red = 0;
    this.green = 0;
    this.blue = 0;

    this.children = new Array(8);
    for(var i = 0; i < this.children.length; i++) this.children[i] = null;

    // 這裏的 next 不是指兄弟鏈中的 next 指針
    // 而是在 reducible 鏈表中的下一個節點
    this.next = null;
};
```

+ `isLeaf`: 表明該節點是否爲葉子節點。
+ `pixelCount`: 在該節點的顏色一共插入了幾次。
+ `red`: 該節點 **R** 通道累加值。
+ `green`: **G** 累加值。
+ `blue`: **B** 累加值。
+ `children`: 八個子節點指針。
+ `next`: ***reducible*** 鏈表的下一個節點指針，後面會作詳細解釋，目前可以先忽略。

#### 插入顏色

　　根據上面的理論，我們大致就知道了往八叉樹插入一個像素點顏色的步驟了。

　　就是每一位 **RGB** 通道黏合的值就是它在樹的那一層的子節點的編號。

　　大致可以看下圖：

![八叉樹插入](http://www.microsoft.com/msj/archive/wicked1.gif)
<small>圖片來源：http://www.microsoft.com/msj/archive/S3F1.aspx</small>

　　由此可以推斷，在沒有任何顏色合併的情況下，插入一種顏色最壞的情況下是進行 64 次檢索。

> **注意：**我們將會把該顏色的 RGB 分量分別累加到該節點的各分量值中，以便最終求平均數。

　　大致的流程就是從根節點開始 DFS，如果到達的節點是葉子節點，那麼分量、顏色總數累加；否則就根據層數和該顏色的第層數位顏色黏合值得到其子節點序號。若該子節點不存在就創建一個子節點並與該父節點關聯，否則就直接搜下一層去。

　　創建的時候根據層數來確定它是不是葉子節點，如果是的話需要標記一下，並且全局的葉子節點數要加一。

　　還有一點需要注意的就是如果這個節點不是葉子節點，就將其丟到 ***reducible*** 相應層數的鏈表當中去，以供之後顏色合併的時候用。關於顏色合併的內容後面會進行解釋。

　　下面是創建節點的代碼：

```javascript
function createNode(idx, level) {
    var node = new OctreeNode();
    if(level === 7) {
        node.isLeaf = true;
        leafNum++;
    } else {
        // 將其丟到第 level 層的 reducible 鏈表中
        node.next = reducible[level];
        reducible[level] = node;
    }

    return node;
}
```

　　以及下面是插入某種顏色的代碼：

```javascript
function addColor(node, color, level) {
    if(node.isLeaf) {
        node.pixelCount++;
        node.red += color.r;
        node.green += color.g;
        node.blue += color.b;
    } else {
        // 由於 js 內部都是以浮點型存儲數值，所以位運算並沒有那麼高效
        // 在此使用直接轉換字符串的方式提取某一位的值
        //
        // 實際上如果用位運算來做的話就是這樣子的：
        //   https://github.com/XadillaX/thmclrx/blob/7ab4de9fce583e88da6a41b0e256e91c45a10f67/src/octree.cpp#L91-L103
        var str = "";
        var r = color.r.toString(2);
        var g = color.g.toString(2);
        var b = color.b.toString(2);
        while(r.length < 8) r = '0' + r;
        while(g.length < 8) g = '0' + g;
        while(b.length < 8) b = '0' + b;

        str += r[level];
        str += g[level];
        str += b[level];
        var idx = parseInt(str, 2);

        if(null === node.children[idx]) {
            node.children[idx] = createNode(node, idx, level + 1);
        }

        if(undefined === node.children[idx]) {
            console.log(color.r.toString(2));
        }

        addColor(node.children[idx], color, level + 1);
    }
}
```

#### 合併顏色

　　這一步就是八叉樹的空間複雜度低和保真度高的另一個原因了。

> 勿忘初心。

　　我們用這個算法做的是顏色量化，或者說我要拿它提取主題色、調色板，所以肯定是提取幾個有代表性的顏色就夠了，否則茫茫世界中 **RRGGBB** 一共有 419430400 種顏色，怎麼歸納？

　　我們可以讓指定一棵八叉樹不超過多少多少葉子節點（也就是最後能歸納出來的主題色數），比如 8，比如 16、64 或者 256 等等。

　　所以當葉子節點數超過我們規定的葉子節點數的時候，我們就要合併其中一個節點，將其所有子節點的數據都合併到它身上去。

　　舉個例子，我們有一個節點有八個子節點，並且都是葉子節點，那麼我們把八個葉子節點的通道分量全累加到該節點中，顏色總數也累加到該節點中，然後刪除八個葉子節點並把該節點設置爲葉子節點。那麼一下子我們就合併了八個節點有木有！

　　爲什麼這些節點可以被合併呢？

　　我們來看看某個節點下的子節點顏色都有神馬相似點吧——它們的三個分量前七位（或者說如果已經不是最底層的節點的話那就是前幾位）是相同的，那麼比如說剛纔的 `FF7800`，跟它同級並且擁有相同父節點（也就是它的兄弟節點）的顏色都是什麼呢：

```
R: 1111 111(0,1)
G: 0111 100(0,1)
B: 0000 000(0,1)
```

　　整合出來一看：

```
FE7800
FE7801
FE7900
FE7901
FF7800
FF7801
FF7900
FF7901
```

　　怎麼樣？是不是確實很相近？這就是八叉樹的精髓了，所有的兄弟節點肯定是在一個相近的顏色範圍內。

　　所以說我們要合併就先去最底層的 ***reducible*** 鏈表中尋找一個可以合併的節點，把它從鏈表中刪除之後合併葉子節點並且刪除其葉子節點就好了：

```javascript
function reduceTree() {
    // 找到最深層次的並且有可合併節點的鏈表
    var lv = 6;
    while(null === reducible[lv]) lv--;

    // 取出鏈表頭並將其從鏈表中移除
    var node = reducible[lv];
    reducible[lv] = node.next;

    // 合併子節點
    var r = 0;
    var g = 0;
    var b = 0;
    var count = 0;
    for(var i = 0; i < 8; i++) {
        if(null === node.children[i]) continue;
        r += node.children[i].red;
        g += node.children[i].green;
        b += node.children[i].blue;
        count += node.children[i].pixelCount;
        leafNum--;
    }

    // 賦值
    node.isLeaf = true;
    node.red = r;
    node.green = g;
    node.blue = b;
    node.pixelCount = count;
    leafNum++;
}
```

　　這樣一來，就合併了一個最深層次的節點了，如果滿打滿算的話，一次合併最多會燒掉 7 個節點（我大 FFF 團壯哉）。

#### 建樹

　　上面的函數都有了，我們可以開始建樹了。

　　實際上建樹的過程就是遍歷一遍傳入的像素顏色信息，對於每個顏色都插入到八叉樹當中；並且每一次插入之後都判斷下葉子節點數有沒有溢出，如果滿出來的話需要及時合併。

```javascript
function buildOctree(pixels, maxColors) {
    for(var i = 0; i < pixels.length; i++) {
        // 添加顏色
        addColor(root, pixels[i], 0);

        // 合併葉子節點
        while(leafNum > maxColors) reduceTree();
    }
}
```

　　整棵樹建好之後，我們應該得到了最多有 `maxColors` 個葉子節點的高保真八叉樹。其根節點爲 `root`。

#### 主題色提取

　　有了這麼一棵八叉樹之後我們就可以從裏面提取我們想要的東西了。

　　主題色提取實際上就是把八叉樹當中剩下的葉子節點 ***RGB*** 通道分量求平均，求出來的就是近似的主題色了。（也許有更好的，不是求平均的方法能獲得更好的主題色結果，但是我沒有深入去研究，歡迎大家一起來指正 (❀╹◡╹)）

　　於是我們深度遍歷這棵樹，每遇到葉子節點，就求出顏色加入到我們所存結果的數組或者任意數據結構當中了：

```javascript
function colorsStats(node, object) {
    if(node.isLeaf) {
        var r = parseInt(node.red / node.pixelCount).toString(16);
        var g = parseInt(node.green / node.pixelCount).toString(16);
        var b = parseInt(node.blue / node.pixelCount).toString(16);
        if(r.length === 1) r = '0' + r;
        if(g.length === 1) g = '0' + g;
        if(b.length === 1) b = '0' + b;

        var color = r + g + b;
        if(object[color]) object[color] += node.pixelCount;
        else object[color] = node.pixelCount;
        
        return;
    }

    for(var i = 0; i < 8; i++) {
        if(null !== node.children[i]) {
            colorsStats(node.children[i], object);
        }
    }
}
```

#### 算法小結

　　八叉樹主題色提取算法提取出來的主題色是一個無固定調色板（Non-palette）的顏色羣，它有着對原圖的儘量保真性，但是由於沒有固定的調色板，有時候對於搜索或者某種需要固定值來解釋的場景中還是欠了點火候。但是活靈活現非它莫屬了。比如某種圖片格式裏面預先存調色板然後存各像素的情況下，我們就可以用八叉樹提取出來的顏色作爲該圖片調色板，能很大程度上對這張圖片進行保真，並且圖片顏色也減到一定的量。

　　該算法的完整 Demo 大家可以在我的 [Github](https://github.com/XadillaX/theme-color-test/blob/master/version3/octree.js) 當中找到。

### 最小差值法

　　這是一個非常簡單又實用的算法。

　　大致的思想就是給定一個調色板，過來一個顏色就跟調色板中的顏色一一對比，取最小差值的那個調色板裏的顏色作爲這個顏色的代表。

　　對比的過程就是分別將 **R / G / B** 通道的值兩兩相減取絕對值，將三個差相加，得到的這個值就是顏色差值了。

　　反正最後就是調色板中哪個顏色跟對比的顏色差值最小，就拿過來就是了。

```javascript
var best = 0;
var bestv = pal[0];
var bestr = Math.abs(r - bestv.r) + Math.abs(g - bestv.g) + Math.abs(b - bestv.b);

for(var j = 1; j < pal.length; j++) {
    var p = pal[j];
    var res = Math.abs(r - p.r) + Math.abs(g - p.g) + Math.abs(b - p.b);
    if(res < bestr) {
        best = j;
        bestv = pal[j];
        bestr = res;
    }
}

r = pal[best].r.toString(16);
g = pal[best].g.toString(16);
b = pal[best].b.toString(16);

if(r.length === 1) r = "0" + r;
if(g.length === 1) g = "0" + g;
if(b.length === 1) b = "0" + b;

if(colors[r + g + b] === undefined) colors[r + g + b] = -1;
colors[r + g + b]++;
```

## 我是怎麼做的

　　八叉樹的缺點我在之前的八叉樹小結中提到過了，就是顏色不固定。對於需要有一定固定值範圍的主題色提取需求來說不是那麼合人意。

　　而最小差值法的話又太古板了。

　　於是我的做法是將這兩種算法都過一遍。

　　比如我要將一張圖片提取出少於 256 種顏色，我會用八叉樹過濾一遍得出保證的兩百多種顏色，然後拿着這批顏色和其數量再扔到最小插值法裏面將顏色規範化一遍，得出的最終結果可能就是我想要的結果了。

　　這期間第二步的效率可以忽略不計，畢竟如果是上面的需求的話第一步的結果也就那麼兩百多種顏色。

　　這個方法我已經實現並且用在我自己的顏色提取包 ***[thmclrx](https://github.com/XadillaX/thmclrx)*** 裏了。大致的代碼可以看[這裏](https://github.com/XadillaX/thmclrx/blob/7ab4de9fce583e88da6a41b0e256e91c45a10f67/lib/x.js#L95-L145)。

## 主題色提取 Node.js 包——thmclrx

　　在這幾天的辛勤勞作下，總算完成了某種意義上我的第一個 Node.js C++ Addon。

　　跟算法相關（八叉樹、最小差值）的計算全放在了 [C++ 層](https://github.com/XadillaX/thmclrx/tree/master/src)進行計算。大家有興趣的可以去讀一下並且幫忙指出各種各樣的缺點，算是拋磚引玉了。

　　這個包的 Repo 在 Github 上面：

> https://github.com/XadillaX/thmclrx

　　文檔自認爲還算完整吧。並且也可以通過

```sh
$ npm install thmclrx
```

　　進行安裝。

## 本文小結

　　進花瓣兩個月了，這一次終於如願以償地碰觸到了一點點的『算法相關』的活。（我不會告訴你這不是我的任務，是我從別人手中搶來的 2333333 *ଘ(੭*ˊᵕˋ)੭* ੈ✩‧₊˚

　　總之幾種算法和實現在上方介紹了，具體需要怎麼用還是要看大家自己了。我反正大致找到了我使用的途徑，那你們呢。( ´･･)ﾉ(._.`)

