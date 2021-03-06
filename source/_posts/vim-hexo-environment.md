title: 搭建 VIM 下的 Hexo 編輯環境 
date: 2014-06-02 04:52:30
tags: [ vim, hexo ]
---

　　本文只講兩個函數，對於 `markdown` 如何高亮之類的問題還請自行谷歌。

　　然後請打開你自己的 `.vimrc` 文件。

## 預備工作

　　首先定義一個變量——你自己的 `hexo` 目錄，如果要跨平臺可以做個判斷之類的，如下：

```vimrc
if has("win32")
    let g:hexoProjectPath="E:\\cygwin\\home\\XadillaX\\hexo"
else
    let g:hexoProjectPath="~/hexo/"
endif
```

## 幾個函數

### 進入 Hexo 目錄

　　這個函數大致就是讓你進入你自己的 `Hexo` 路徑：

```vimrc
fun! OpenHexoProjPath()
    execute "cd " . g:hexoProjectPath
endfun
```

### 打開一篇 Post

　　接下去就是一個打開 `Post` 的函數了：

```vimrc
function! OpenHexoPost(...)
    call OpenHexoProjPath()

    let filename = "source/_posts/" . a:1 . ".md"
    execute "e " . filename
endfunction
```

> 解析：上面的代碼大意就是進入 Hexo 路徑，然後設定好文件名，最後執行 `:e filename` 即可打開文件了。

### 新建一篇 Post

　　新建的流程跟打開相似，只不過首先要在 `Hexo` 目錄下執行一遍 `hexo new FOO` 的命令而已，命令執行完畢之後再打開即可。

```vimrc
function! NewHexoPost(...)
    call OpenHexoProjPath()

    let filename = a:1
    execute "!hexo new " . filename

    call OpenHexoPost(a:1)
endfunction
```

## 指令映射

　　函數寫好後我們最後把函數映射成類似於 `:e`, `:w` 之類的後面能跟着參數的指令即可。

　　以前木有接觸過的同學可以參考一下[這裏](http://vimdoc.sourceforge.net/htmldoc/usr_40.html#40.2)的文檔。

### 打開指令

```vimrc
command -nargs=+ HexoOpen :call OpenHexoPost("<args>")
```

### 新建指令

```vimrc
command -nargs=+ HexoNew :call NewHexoPost("<args>")
```

## 使用方法

　　當你做完以上步驟的時候，你就可以無論在什麼目錄下在 VIM 裏面通過下面的指令進行新建一篇日誌了：

```vim
:HexoNew artical-name
```

　　以及下面的指令來打開一篇已存在的日誌：

```vim
:HexoOpen artical-name
```

## 遺留問題

　　相信看到這裏之後，大家也能自己寫出一個生成的指令了，這裏就不累述了，無非就是：

```vim
:!hexo generate
```

