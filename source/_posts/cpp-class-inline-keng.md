title: C++中類成員函數 inline 的坑
date: 2014-04-05 16:55:57
tags: [ Programming, C++ ]
category: Programming
---

　　今天我來講一講 `C++` 中類成員函數 `inline` 修飾符的一個坑。

　　這個坑是我在嘗試着寫我的第一個 `Node.js` 擴展 `simpleini` 時候遇到的。

## 坑描述

　　因爲只是嘗試着寫，所以懶得自己實現，於是網上找了個開源的 `C++` 閱讀 ini 文件的項目，名不見經傳，叫 [miniini](http://miniini.tuxfamily.org/)。

　　好了，問題來了，當我寫好我的源文件的時候，然後寫好了我的 `binding.gyp` ，總之一切大功告成開始編譯的時候—— `Windows` 下沒問題，`MacOS` 下也可以正常運行，但是在 `Linux` 下就出問題了：

```sh
node: symbol lookup err: .../simpleIni.node: undefined symbol: _ZNK10INISection10ReadStringEPKcRS1_
```

　　大致的意思呢就是說找不到 `INISection` 的 `ReadString` 函數符號。

## 問題分析

　　又是懷着崇敬的心情去 [SO](http://stackoverflow.com/questions/22868307/undefined-symbol-in-node-js-c-addon-under-linux-why) 求解了。

　　最後的解答大概[如下](http://isocpp.org/wiki/faq/inline-functions#inline-member-fns)：

> 內聯成員函數的聲明看起來像一個非內聯函數的聲明：
>```cpp
class Fred {
public:
    void f(int i, char c);
};
```

> 但是你的內斂成員函數定義前面又加了 `inline` 這個關鍵字時，你必須把這個定義放到頭文件中：
>```cpp
inline
void Fred::f(int i, char c)
{
    // ...
}
```

> 這麼做的原因就是爲了避免鏈接器 `unresolved external` 的發生。如果你不這麼做，這個錯誤就將會在你從另外一個 `.cpp` 文件中調用它時出現。


　　好嘛，原來是原作者自己寫的代碼有問題啊。但是不得不說一下又漲姿勢了。C++還真是有千奇百怪的坑和錯誤啊。

## 解決方案

　　最後的解決方案大致就是把函數定義放到頭文件中去，或者在函數聲明前面也加上 `inline` 關鍵字。
  
## simpleini

　　我的第一個 `C++` 模塊，叫 `simpleini` ，其實只是抱着試試看 `Node.j` 的 `C++` 模塊是不是這麼寫的而已，並沒有多大實際用處。Repo 在 [Github](https://github.com/XadillaX/node-simple-ini) 上。

　　然後用法很簡單，先安裝：

```sh
$ npm install simpleini
```

　　然後下面的代碼就是例子了：

```javascript
var simpleIni = require("simpleini");

console.log(simpleIni.open("./node_modules/simpleini/src/miniini-0.9/test/test.ini"));
console.log(simpleIni.read("a", "b", "c"));
console.log(simpleIni.read("a", "b", "d"));
console.log(simpleIni.read("SETTINGS", "sections"));
console.log(simpleIni.read("vals", "float"));
```

　　讀取配置的時候第一個參數是 `Section`，第二個參數是 `Key`，第三個參數是取不到該值時的默認值。
