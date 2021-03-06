title: Cocos2d-x 3.1.1 開發環境搭建（Win篇）
date: 2014-06-07 19:34:02
tags: [ cocos2d ]
category: 遊戲開發
---

　　由於偷懶，所以在此感謝 Etond 的指導（喂喂喂，明明是自己懶得看文檔，明明 [`READEME.md`](README) 裏面就有！(´≖◞౪◟≖)

　　另，在搭建環境的時候，最好保證你在<span style="background: #222;">牆外</span>。以及我默認覺得大家已經有了 `Python` 環境和 `JDK`。

## 前驅工作

　　先去 [cocos2d-x 官網](http://www.cocos2d-x.org/download)下壓縮包，放到一個只有神知道的世界裏面。

　　接下去需要安裝仨東西：

### Android SDK

　　[這東西](http://developer.android.com/sdk/index.html#download)真尼瑪大啊！我家的小水管真吃不起。

　　然後把 **adt-bundle-...zip** 這個包壓縮到任意木有中文和空格的路徑下面。

### NDK

　　[這小夥伴](http://developer.android.com/tools/sdk/ndk/index.html#download)也不小啊。都是 500M 的主兒啊（٩(ŏ﹏ŏ、)۶

　　也解壓到一個地方不用管它。

### Ant

　　據說這貨是阿帕奇出的？總之下載地址在[這裏](http://ant.apache.org/bindownload.cgi)。

## 安裝

　　哦對了你還得有個 Python 路徑，這裏就不累述了。接下去在命令行裏面執行 Cocos2d 的 `setup.py` 文件即可：

```shell
/> py setup.py
```

　　接下去終端會停在下面一行：

```shell
Please enter the path of NDK_ROOT (or press Enter to skip):
```

　　在後面輸入你放好的 NDK 目錄即可。

　　如果下面又出現了：

```shell
Please enter the path of ANDROID_SDK_ROOT (or press Enter to skip):
```

　　你只需在裏面輸入你剛放好的 Android SDK 的目錄即可。（注意是要剛纔的 SDK 壓縮包解壓出來的 sdk 路徑）

　　再如果下面還出現：

```shell
Please enter the path of ANT_ROOT (or press Enter to skip):
```

　　那麼再把 Ant 的路徑搞上去就好了。（又得注意這裏得是 Ant 的 bin 目錄）

　　最後確保終端（或者說命令行）裏面出現如下字樣：

```shell
Please restart the terminal or restart computer to make added system variables take effect
```

　　然後你把終端關了再開一個就好了。至此，大致就安裝完畢了。

## 新建一個 Demo 項目

　　隨意跑到一個目錄下面執行下面的命令：

```shell
/> cocos new FirstGame -p in.xcoder.firstgame -l cpp -d FirstGame
```

> 大致意思就是說創建一個新的項目路徑，叫 `FirstGame`，其包名叫 `in.xcoder.firstgame`，然後語言是 `cpp`，最後 `-d` 是路徑。

　　命令詳情幫助可以看 `cocos --help`。

### 編譯 Demo

　　讀標題，是 Win 篇。所以我們跑到項目路徑下面的 `proj.win32` 目錄下面用 M$ VS 打開 `FirstGame.sln` 就可以打開剛創建的模板項目了。

　　無論如何先編譯看看吧！～

　　如何？跑起來了吧？

### 打包 Demo

　　這裏就講講如何打包安卓的版本吧：

#### Debug 版本

　　跑到你的項目目錄下面（即有 `.cocos-project.json` 文件的目錄），然後執行下面的命令：

```shell
/> cocos run -p android
```

　　等工具編譯打包完成就 OK 了。（記得要查安卓手機並且調試模式哦～）

#### Release 版本

　　如果要上傳到 Google Play 之類的地方，需要有簽名。所以發佈 Release 版本之前，你先得搞好自己的簽名。

##### Keytool

　　在終端跑到你的項目路徑下面，然後執行：

```shell
/> keytool -genkey -v -keystore FirstGame.keystore -alias FirstGame -keyalg RSA -keysize 2048 -validaty 10000
```

　　照着命令行給的提示完成創建密鑰即可。

##### 編譯

　　生成之後啊就直接執行編譯命令了：

```shell
/> cocos run -p android -m release
```

　　在裏面呢最後會讓你輸入 `.keystore` 文件的路徑。

　　我們輸入相對路徑，由於我們剛纔把這個文件搞在項目根目錄，所以我們只需要輸入 `../FirstGame.keystore` 即可。接下去他會讓你輸入密碼、別名和別名信息的密碼。你都正確輸入一遍他就會安安分分跑在你的手機裏面了。

#### 仨版本的文件路徑

　　上面都弄好之後，你的仨版本 `*.apk` 文件也就生成了。很多人可能很困惑，爲什麼是仨版本。因爲其中 Release 版本還分帶簽名和沒簽名版本。

　　總之那個路徑在 `publish/android` 下面，裏面有仨 `*.apk` 文件，你拿出來發佈就可以了。

## 小結

　　其實也沒什麼結不結的，這些東西你們自己去看看官方文檔就好了。總之就這樣了吧，以上。
