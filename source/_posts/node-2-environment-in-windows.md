title       : 一起擼Node.JS（負貳）——環境
category    : NodeJS
date        : 2013-08-15
tags        : [ Node.js, JavaScript, 一起擼Node.JS ]
---

　　由於[Linux]({{ page.url }}#linux-環境)中的環境搭建比較簡單，所以草草略過。

　　其實[Windows]({{ page.url }}#windows-環境)下也不算麻煩，但是這裏會講一定量的別的環境的搭建。


<!-- 我是小小分割符 -->
## Linux 環境

講到這個就很簡單了，跟着下面的 **bash** 操作即可：

{% code sh %}
$ cd /usr/local/bin
$ wget http://nodejs.org/dist/v0.00.00/node-v0.00.00-linux-x00.tar.gz
$ tar zxf node-v0.00.00-linux-x00.tar.gz
$ cd node-v0.00.00-linux-x00
{% endcode %}

> 其中將上方的 **v0.00.00** 替換成 **Node.js** 最新的版本號，把 **x00** 替換成你自己電腦的位數。
>
> 也可以直接去官網 [http://nodejs.org/download/](http://nodejs.org/download/) 找到相應的地址。

最後將其的連接加入到 `/usr/bin` 下即可。

{% code sh %}
$ cd bin
$ ln node /usr/bin
$ ln npm /usr/bin
{% endcode %}

> **注意**： 該用 `sudo` 的地方就用 `sudo` 或者 `su` 。

至此，**Linux** 下的 **Node.js** 環境基本搭建完畢。

## Windows 環境

### Cygwin 安裝和配置

***Cygwin*** 是一個在 ***Windows*** 平臺上運行的 ***Unix*** 模擬環境。對於學習 ***Unix/Linux*** 操作環境，或者從 ***Unix*** 到 ***Windows*** 的應用程序移植，或者進行某些特殊的開發工作，尤其是使用 ***GNU工具集*** 在 ***Windows*** 上進行嵌入式系統開發，非常有用。

#### Cygwin 安裝

我們先跑到 **Cygwin** 的官網上去把東西下來：

> [http://cygwin.com/install.html](http://cygwin.com/install.html)
>
> > 注意，最好下 **x86** 的包，因爲我們之後要講一個 `cyg-apt` 的腳本插件，這是一個能讓 **Cygwin** 能跟 **Linux** 一樣通過腳本從源安裝軟件包的腳本。爲了方便修改，我們將其下成 **x86** 的版本。

然後就是安裝步驟了。

<center>![從網絡安裝](http://blog-xcoder-in.qiniudn.com/cygwin-install-1.png)</center>
<center><small>[圖2.1]</small></center>

到 **[圖2.1]** 這個步驟的時候，選擇默認的 `Install from Internet` 即可。

<center>![選擇安裝路徑](http://blog-xcoder-in.qiniudn.com/cygwin-install-2.png)</center>
<center><small>[圖2.2]</small></center>

在 **[圖2.2]** 的時候選一個安裝路徑。

> **注意**：儘可能讓這個安裝路徑簡單，而不要是類似於
>
> `c:\Program Files\blahblah`
>
> 這樣的文件路徑。

<center>![本地包路徑](http://blog-xcoder-in.qiniudn.com/cygwin-install-3.png)</center>
<center><small>[圖2.3]</small></center>

**[圖2.3]** 的時候選一個本地包的路徑，我這裏選的是 `e:\cygwin\tmp`。

<center>![直連](http://blog-xcoder-in.qiniudn.com/cygwin-install-4.png)</center>
<center><small>[圖2.4]</small></center>

**[圖2.4]** 選擇直接連接。

<center>![163](http://blog-xcoder-in.qiniudn.com/cygwin-install-5.png)</center>
<center><small>[圖2.5]</small></center>

我們國內的用戶源還是選擇 `163` 的速度比較快。所以在 **[圖2.5]** 這一步的時候就直接選用默認的 `163` 的源了。如果不是默認的話，請選中它。

在 **Select Package** 也就是選擇預安裝的軟件的時候，把下列表中的軟件包勾選起來：

> + **wget**: 在 **Utils** 中
> + **vim**: 在 **Editors** 中
> + **gcc**: 在 **Devel** 中
> + **gcc-g++**: 在 **Devel** 中
> + **make**: 在 **Devel** 中
> + **cmake**: 在 **Devel** 中

若是這些選項已經被選起來了就不用再選了，如果沒有選起來則把它選中。

勾選好了之後就可以下一步安裝了，直至安裝完畢，你就可以打開你的 **Cygwin** 了。

<center>![Cygwin](http://blog-xcoder-in.qiniudn.com/cygwin-install-6.png)</center>
<center><small>[圖2.6]</small></center>

> **提示**：你可以點擊窗口左上角的小圖片，然後裏面的 **Options** 中，你可以調整你自己的 **Cygwin** 外觀。

### vim 配置

上一步我們已經選中了 **vim** ，也就是說我們已經在 **Cygwin** 中裝上了 **vim**。但是由於這裏的 **vim** 默認配置非常蛋疼，所以我們得改一下。

在你的 **Cygwin** 中一句句輸入下面的命令：

{% code sh %}
$ cd /home/<你自己的用戶名>
$ wget http://blog-xcoder-in.qiniudn.com/.vimrc
$ mkdir .vim
$ cd .vim
$ mkdir colors
$ cd colors
$ wget http://blog-xcoder-in.qiniudn.com/molokai.vim
{% endcode %}

這樣你的 **vim** 就用上了上面的那個地址的配置文件，當然你也可以編輯你自己的配置文件或者說從網上下別的配置文件以滿足你的個性化需求。

**vim** 配置以及使用請參照：[https://wiki.archlinux.org/index.php/Vim](https://wiki.archlinux.org/index.php/Vim)

> 事無鉅細問 **ArchWiki**。
> <div style="text-align: right;">*-- [kalxd](https://github.com/kalxd)*</div>

### apt-cyg

> apt-cyg is a command-line installer for Cygwin which cooperates with Cygwin Setup and uses the same repository. The syntax is similar to apt-get.
>
> <div style="text-align: right;">*-- From apt-cyg googlecode page*</div>

總之意思就是說 `apt-cyg` 是類似於 **Linux** 中的 `apt-get`， `yum`, `zypper` 等命令行軟件包安裝器一樣，可以通過

+ `apt-cyg install <package names>` 來安裝軟件包
+ `apt-cyg remove <package names>` 來移除軟件包
+ `apt-cyg update` 來更新 setup.ini
+ `apt-cyg show` 來列出已安裝的軟件包
+ `apt-cyg find <pattern(s)>` 來查找符合條件的軟件包
+ `apt-cyg describe <pattern(s)>` 來描述符合條件的軟件包
+ `apt-cyg packageof <commands or files>` 來定位其父軟件包

#### apt-cyg 安裝

其實也不能說是安裝，純粹是把腳本從網絡上拷到自己的 **Cygwin** 的環境目錄中。

在你的 **Cygwin** 中輸入以下命令：

{% code sh %}
$ cd /usr/local/bin
$ wget http://apt-cyg.googlecode.com/svn/trunk/apt-cyg
{% endcode %}

這樣你就“安裝”好了 **apt-cyg** 了。不過這裏用的是默認的源，所有東西都是默認的。

如果你現在已經心安理得或者不想折騰了可以跳過 **[2.1.3.2. apt-cyg 修改](#apt-cyg-修改)**，如果你想把源換成 `163` 的話那麼稍微看一下吧。

#### apt-cyg 修改

接下去我們要對 **apt-cyg** 做一些編輯。

你有下面兩個選擇：

1. 如果你想學習 **vim** 操作或者你已經熟悉了，那麼直接使用 `vim apt-cyg` 來進行編輯。
2. 如果你是懶人還是想要直接編輯的話，請跑到你的 **Cygwin** 的安裝目錄，找到 **usr** 文件夾，飛進 **local/bin** 目錄中去，用你自己喜歡的文本編輯器打開並編輯。

大約是 `68` 行上下吧，有一句是：

{% code bash %}
  mirror=ftp://mirror.mcs.anl.gov/pub/cygwin
{% endcode %}

將其改成：

{% code bash %}
  mirror=http://mirrors.163.com/cygwin
{% endcode %}

還有就是大概在 `98` 行和 `105` 行左右：

{% code bash %}
    wget -N $mirror/setup.bz2
    ...
    wget -N $mirror/setup.ini
{% endcode %}

修改成：

{% code bash %}
    wget -N $mirror/x86/setup.bz2
    ...
    wget -N $mirror/x86/setup.ini
{% endcode %}

至此，你的 **Cygwin** 環境基本完成，以後可以再慢慢完善。

### Node.js 安裝

這個就很簡單了，打開 **[Node.js](http://nodejs.org/download/)** 官網下載安裝即可。

> 選擇 **Windows Installer (.msi)** 或者 **Windows Binary (.exe)**。

安裝好後就能直接在 **Cygwin** 裏面使用了。

## 真·Hello World

現在，無論你是 **Linux** 用戶還是 **Windows** 用戶，都可以用一樣的步驟來完成下面的 `Hello World` 了。

隨便跑一個目錄裏面新建一個文件並且用 **vim** 編輯：

{% code sh %}
$ vim hello.js
{% endcode %}

在裏面輸入下面的東西：

{% code javascript %}
console.log("Hello world!");
{% endcode %}

然後退出 **vim** 執行：

{% code sh %}
$ node hello.js
{% endcode %}

終於，**真·Hello world** 出現在了你的眼前，而不需要藉助 **[IDEOne](http://ideone.com/)** 了。

***To be continued...***
