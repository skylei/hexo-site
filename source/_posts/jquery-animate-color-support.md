title: 關於jQuery中“animate()”函數對顏色變化的支持
date: 2012-12-24 09:20:16
tags: [ 老博客備份歸檔, Javascript, jQuery ]
category: 老博客備份歸檔
---

　　最近在做一個汽車團購網的項目，由於老大力求簡潔，所以界面做得有些小清新。不過得說一下頁面不是我設計的，是一位美工同志。

　　廢話不多說，直接切入正題吧——

　　我要做得就是讓下面一段代碼生效：

```javascript
$("#yourid").stop().animate({ "backgroundColor" : "#rrggbb", "color" : "#rrggbb" }, "fast");
```

　　但是，很遺憾，一點也沒有動。本來效果應該跟這個版本的xcoder博客的天頭導航條一樣有個動態效果（只不過xcoder的導航條是透明度變化，而項目中我想讓它背景色變化）。

　　原因是什麼呢？死月上網查了很久，找到的東西都很簡單地說明瞭一下，貌似都可以。嘛，也許是jQuery新版本不支持這個特性了吧。

　　最後，死月在jQuery的官方文檔中找到了下面這段話——
  
> All animated properties should be animated to ***a single numeric value***, except as noted below; most properties that are non-numeric cannot be animated using basic jQuery functionality (For example, width, height, or left can be animated but background-color cannot be, unless the [jQuery.Color()](https://github.com/jquery/jquery-color) plugin is used). Property values are treated as a number of pixels unless otherwise specified. The units em and % can be specified where applicable.
>
> <p style="text-align: right">—— [jQuery官方文檔 .animate()](http://api.jquery.com/animate/)</p>

　　大致的意思就是說所有動畫屬性都必須是一個單數字值，所以說大多數非數字的屬性是不能被動畫化的。例如高度、寬度等可以被動畫化，但是背景色就不信了。<span style="color: red;">***除非你用了jQuery.Color()插件***</span>。

　　所以說問題找到了，我們必須得用一個jQuery.Color()插件來對一些顏色進行動畫操作。

　　話不多說，我們去下一個jQuery.Color()插件。把它加在我們的頁面中，然後就可以用如下方式來進行動畫操作了：

```javascript
$(this).stop().animate({
    "backgroundColor" : jQuery.Color("rrggbb"),
    "color" : jQuery.Color("rrggbb")
}, "fast");
```