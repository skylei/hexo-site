title: Untrusted - 遊戲題解
date: 2014-06-12 11:08:34
tags: [ Javascript, untrusted, 遊戲 ]
---

　　[Trusted](http://alexnisnevich.github.io/untrusted/) 是一個代碼解謎遊戲，用 Javascript 來過關的。

　　昨天凌晨花了仨小時通關了這個遊戲，在這裏就粗粗做一下題解吧，好幾題都是 Hack 過去的。（不要臉，( ﾟДﾟ)σ

## Ceil Block A

　　這有點像教學關吧，總之先拿到那臺電腦你就能操作了。拿到電腦後你就能修改地圖內部黑色底色的代碼了。

　　這個時候你只需要把中間設置牆的代碼去掉就可以了，或者註釋掉：

```javascript
//for(y = 3; y <= map.getHeight() - 10; y++) {
//    map.placeObject(5, y, 'block');
//    map.placeObject(map.getWidth() - 5, y, 'block');
//}
//
//for(x = 5; x <= map.getWidth() - 5; x++) {
//    map.placeObject(x, 3, 'block');
//    map.placeObject(x, map.getHeight() - 10, 'block');
//}
```

　　然後 `<ctrl-5>` 重新執行——噠噠～牆就消失了，趕緊到藍色的出口處吧。

## The Long Way Out

　　代碼大致是給你創建了一個迷宮，並且出口處四面用圍牆圍起來。

　　我用了一個比較 Hack 的方法，在第一個黑色區域的最上方把 `maze.create` 重定向到自己的一個空函數，這樣下面調用創建迷宮的函數就不會被執行，這個時候再執行的話迷宮就不見了：

```javascript
maze.create = function() {};
```

　　迷宮不見了還不靠譜，因爲還有一個出口四周有牆——那就自己再建一個出口唄，在第二個黑色區域寫上建立一個新出口的代碼即可：

```javascript
map.placeObject(0, 0, "exit"); 
```

> 勇敢的少年啊，快去創造奇蹟！

## Validation Engaged

　　這題的要求是在還存在着一定量『壁』的情況下你能到達出口，也就是說純粹地刪除它加『壁』的代碼是不行的，那我們做點改動就 OK 了。把『壁』往外移動，直到把人和出口都是在『壁』內。

> 那一天，人類終於回想起曾經一度被他們所支配的恐怖，還有囚禁於鳥籠中的那份屈辱。

```javascript
for(y = 0; y <= map.getHeight() - 3; y++) {
    map.placeObject(5, y, 'block');
    map.placeObject(map.getWidth() - 5, y, 'block');
}

for(x = 0; x <= map.getWidth() - 5; x++) {
    map.placeObject(x, 3, 'block');
    map.placeObject(x, map.getHeight() - 3, 'block');
}
```

## Multiplicity

　　嘛嘛，這是第二關的簡化版——直接再搞一個出口就 OK 了。

```javascript
map.placeObject(20, 10, 'exit');
```

## Minesweeper

　　這是一個雷區，你不碰雷就好。從代碼裏面看出來有個 `map.setSquareColor` 函數可以設置某個格子的顏色。那好辦，我們在設置一個地雷後把它用別的顏色標記出來就好了，然後重新執行只要你不是色盲都能安全通過。

```javascript
map.setSquareColor(x, y, "#ff7800");
```

## Drones 101

　　這題大概就是說有個癡漢會跟你靠近，然後把你先奸後殺。

　　但是癡漢很笨，在他的必經之路用牆堵住他就不會繼續動了。

```javascript
map.placeObject(30, 12, 'block');
map.placeObject(31, 11, 'block');
```

## Colors

　　這個是那個賣相不錯的電話機的教學關卡。所以大致的意思是設置了打電話的回調函數即可。ε٩(๑> ₃ <)۶з

　　分析代碼可知，要通過那幾個長得跟菊花一樣的帶色兒的牆你就要跟那個菊花顏色一樣。所以電話機的回調函數大致是讓你自己變色就好了。

　　按照順序所見，如果人是綠色的通過之後要變成紅色，然後再變成黃色再綠色。於是寫以下的變色過程就可以了：

```javascript
var player = map.getPlayer();

var color = player.getColor();
switch(color) {
    case "#0f0":
        player.setColor("#f00");
        break;
    case "#f00":
        player.setColor("#ff0");
        break;
    case "#ff0":
        player.setColor("#0f0");
        break;
}
```

　　重新執行撿起電話機，然後通過綠菊花之後按 `Q` 使用電話機讓自己變色兒就好了。

> “哎呀，天！他是惦記弟弟了。……可我還不知道呢！那麼這是他老人家的狗？很高興。……你把它帶去吧。……這條小狗怪不錯的。……挺伶俐。……一口就把這傢伙的手指咬破了！哈哈哈哈！……咦，你幹嗎發抖？嗚嗚，……嗚嗚。……它生氣了，小壞蛋，……好一條小狗……”

## Into the Woods

　　森林裏面有樹和牆，我也懶得想或者寫代碼了。（明明是自己想不出來#ﾟÅﾟ）⊂彡☆))ﾟДﾟ)･∵

　　總之我是儘可能向出口靠近，然後到死路了趕緊打電話讓森林重新生成一遍，如此循環往復直到出口。

## Fording the River

　　23333333333333！做這題的時候差點沒把自己瀏覽器卡死。

　　大致的意思是河的上面有一條船，你直接遇水會死，要上船。但是船貌似不跟你走啊 QAQ。

　　而且設定寫着只能有一條 `raft`。

　　咱就來個偷天換日，自己造諾亞方舟鋪滿整條河（因爲懶得計算）。

　　首先定義諾亞方舟的類型：

```javascript
map.defineObject("noah", {
    'type': 'dynamic',
    'symbol': 'a',
    'color': '#420',
    'transport': true,
    'behavior': function(me) {
    }
});
```

　　然後呢把它鋪滿整條大河吧：

```javascript
for(var x = 0; x < map.getWidth(); x++) {
    for(var y = 5; y < 15; y++) {
        map.placeObject(x, y, 'noah');
    }
}
```

> 一條大河，兩岸寬，風吹稻花香兩岸。（喂喂喂，小心卡死_(┐「ε:)_

## Ambush

　　後來我去 `Untrusted` 的 repo 去看題解，發現他們都是去驅使這羣癡漢幹嘛幹嘛。我感覺我的最簡單暴力了——直接廢了他們。

　　其實呢只要把碰撞函數重寫一遍，這堆癡漢馬上就變得人畜無害，你走過去人家還行禮呢233333333333

　　仔細看一下我們要完成的部分在 `behavior` 裏面，所以在這裏面用 `this` 是妥妥生效的。

```javascript
this.onCollision = function() {};
```

> 看我碎蛋大粉拳！（忽然覺得下身一陣疼痛  |Д`)ノ⌒●～*

## Robot

　　你走一步機器人走一步，也是教學關卡。

　　機器人能往下走就往下走，能往右走就往右走就拿到鑰匙了，最後你再追上機器人把鑰匙搶過來就好了。因爲機器人是可以穿過紫翔色的那扇門的。

```javascript
if(me.canMove("down")) me.move("down");
else me.move("right");
```

> 站住，保護費。你不裝 X 我們還是好朋友。

## Robot Nav

　　我居然無聊到自己把路線數出來了。

```javascript
var road = "ddddrrrrrrrrrrrrrrrrrrrrrrrrrrrrrruurrrrrrrrrrrrrrrrrddddddd";
this.cur = this.cur === undefined ? 0 : (this.cur + 1);

if(this.cur >= road.length) return;

if(road[this.cur] === "d") me.move("down");
if(road[this.cur] === "r") me.move("right");
if(road[this.cur] === "u") me.move("up");
```

## Robot Maze

　　好吧作者早就想到了有人會無聊地去數。

　　嘛嘛，就如作者所願寫個最基礎的 DFS 了事吧。

```javascript
var direct = {
    "d": "down",
    "u": "up",
    "l": "left",
    "r": "right"
};

// dfs...
if(undefined === this.dfs) {
    this.ans = "";
    this.step = 0;

    var vis = [];
    for(var i = 0; i < 100; i++) {
        vis.push([]);
        for(var j = 0; j < 100; j++) vis[i].push(false);
    }

    var dir = [
        [ 0, -1, "u", "#f00" ],
        [ 0, 1, "d", "#0f0" ],
        [ -1, 0, "l", "#00f" ],
        [ 1, 0, "r", "#fff" ]
    ];

    this.dfs = function(x, y) {
        if(x === map.getWidth() - 2 && y === 8) {
            return true;
        }
        vis[y][x] = true;

        for(var i = 0; i < 4; i++) {
            var newx = x + dir[i][0];
            var newy = y + dir[i][1];

            if(newx < 0 || newy < 0 ||
                newx >= map.getWidth() ||
                newy >= map.getHeight() ||
                vis[newy][newx] ||
                map.getObjectTypeAt(newx, newy) === "block"
                ) continue;

            var oldans = this.ans;
            this.ans += dir[i][2];

            if(!this.dfs(newx, newy)) {
                this.ans = oldans;
            } else {
                map.setSquareColor(x, y, dir[i][3]);

                return true;
            }
        }

        return false;
    };

    this.dfs(1, 1);
    this.ans += "dd";
}

if(this.step >= this.ans.length) return;
me.move(direct[this.ans[this.step++]]);
```

> 紅魔館的地下室一樣呢。反正是機器人多走幾步路沒事，沒必要用 BFS 求最優解2333333333

## Crisps Contest

　　剛纔那仨 2B 機器人引領你拿到了仨顏色的鑰匙在這邊派上用場了。

　　鑽紅菊花你需要有紅鑰匙，並且用了之後會少掉。其它顏色也一樣。最終你要拿到 `A` 所代表的 `theAlgorithm` 走到出口。

> 等等！啊咧？綠鑰匙的通過判定有個地方可以修改？就是你通過綠菊花的時候需要有綠鑰匙並且你可以選擇你丟棄的東西。丟什麼好呢？電腦？不行不行，過關還靠它呢。電話機？以後肯定要用到。其它顏色鑰匙？那你肯定會被鎖在某個地方出不來。那就只有丟棄 `theAlgorithm` 了——反正只要拿到 `theAlgorithm` 之後不再通過綠菊花就沒事了。

　　於是隻要把綠菊花的通過判斷函數裏面可修改的區域改成 `theAlgorithm` 就好了。

　　最後走的順序大概是：

> 進左上角的門拿到<span style="color: yellow;">黃藥屎</span>和<span style="color: blue;">藍藥屎</span>出來。然後右上角把<span style="color: red;">**紅**</span>和<span style="color: blue;">**藍**</span>拿出來。然後向下直搗黃龍，左黃菊花進拿到 `theAlgorithm` 藍菊花通過拿到<span style="color: yellow;">黃藥屎</span>然後再黃菊花出。
>
> 大功告成！走向勝利的出口吧！
>
> **自古紅藍出 CP！**

## Exceptional Crossing

　　又是過河啊，這次你只能是死了，因爲你的編輯區域只有在 `player.killedBy()` 裏面。

> 《訂製死神》：這個時候讓死神笑就可以了。

　　讓我們一起來玩壞它吧！在裏面填上 `) = (0` 就好了。什麼什麼看不懂？你填進去看一下整句話就知道了：

```javascript
player.killedBy() = (0);
```

　　然後死神就會被你玩壞了。你走過去的時候這句話執行出錯了2333333

## Lasers

　　有很多隱藏線，你人必須要跟隱藏線的顏色一致才能通過，然後目前所有線都用白色給畫出來。

　　目測作者的意思是讓你把硬編碼的白色改成隱藏線的顏色，這樣就能把線的顏色給標記出來，然後再給電話機寫個函數就是讓你自己的人變色。

　　不過我還是用了個 Hack 的方法——

　　第一條線他要畫就畫，咱不碰它就好了，只不過在第一條線畫完的後面我們把這個畫線函數給 Hack 掉：

```javascript
// using canvas to draw the line
var ctx = map.getCanvasContext();
ctx.beginPath();
ctx.strokeStyle = 'white';
ctx.lineWidth = 5;
ctx.moveTo(x1, y1);
ctx.lineTo(x2, y2);
ctx.stroke();

createLaser = abc;
```

　　接下去是在第二片區域寫下自己的畫線函數吧，這題最下方檢測了線的數量不能少於 25 條。麼事，爺高興畫 100 條都麼問題，因爲我都把它縮在左上角了 2333333

```javascript
function abc() {
    for(var i = 0; i < 25; i++) {
        map.createLine([1, 1], [2, 2], function(player) {
            //... Ahahaha
        });

        var ctx = map.getCanvasContext();
        ctx.beginPath();
        ctx.strokeStyle = 'red';
        ctx.lineWidth = 5;
        ctx.moveTo(1, 1);
        ctx.lineTo(2, 2);
        ctx.stroke();
    }
}
```

## Pointers

　　有好多傳送門，每次執行隨機生成傳送位置，有些傳送門會把你傳到二小姐的地下室然後被吃掉。

　　我也懶得多動腦筋或者畫線什麼的，直接對兩個都是傳送門的 CP 標記一樣的隨機顏色就好了，最後跟着顏色走到出口去（有個坑就是有時候這個地圖本身就是死局，所以得多試幾次重新執行 இдஇ

```javascript
var dict = "0123456789ABCDEF";
if(t1.getType() === "teleporter" && t2.getType() === "teleporter") {
    var color = "#" + dict[Math.ceil(Math.random() * 15)] +
        dict[Math.ceil(Math.random() * 15)] +
        dict[Math.ceil(Math.random() * 15)];

    map.setSquareColor(t1.getX(), t1.getY(), color);
    map.setSquareColor(t2.getX(), t2.getY(), color);
}
```

## Super Dr. Eval Bros

　　好吧本意是讓你設置一個 `timer` 然後一直跳啊跳的。

　　不過呢，定一個新方塊給自己搭一座橋就是了：

```javascript
map.defineObject("❤", {
    impassable: function() {
        return true;
    },
    symbol: "❤"
});
map.placeObject(20, 12, "❤");
map.placeObject(21, 12, "❤");
map.placeObject(22, 12, "❤");
map.placeObject(23, 12, "❤");
map.placeObject(24, 12, "❤");
map.placeObject(25, 12, "❤");
map.placeObject(26, 12, "❤");
map.placeObject(27, 12, "❤");
map.placeObject(28, 12, "❤");
map.placeObject(29, 12, "❤");
```

> 你只要打個電話橋就會出現的。

## Document Object Madness

　　好神奇！好奇葩！我鍵盤 `hjkl` 亂按一通就過了。

## Boss Fight

　　打 Boss 了。

　　好吧我承認我 Cheat 了——原諒我用了 `console.log`。

> 因爲當我打開控制檯的時候下面的語句出現在我的眼裏：
>
> > ***If you can read this, you are cheating!***
> >
> > ***But really, you don't need this console to play the game. Walk around using arrow keys (or Vim keys), and pick up the computer (⌘). Then the fun begins!***

　　嘛嘛，無論如何，過關了就好。

　　這題呢是要讓所有的 `boss` 給毀滅掉即可—— 當所有的 `boss` 毀滅之後會爆出任務道具 `theAlgorithm` 然後就能通關了。

　　後來我發現可以讓子彈消滅 `boss`。但是我當時沒這麼做。

　　我先弄了堵牆把子彈擋住先：

```javascript
map.defineObject("保命的", {
    impassable: function() {
        return true;
    },
    symbol: "❤",
    onCollision: function() {
    }
});

for(var i = 0; i < map.getWidth(); i++) {
    map.placeObject(i, 9, "保命的");
}
```

　　這下你就能撿到電話機了——然後給電話機寫回調函數。

　　怎麼說呢，當你每用一次電話機，我就把當前存在於屏幕的 `boss` 和 `bullet` 給分開羅列，然後把 `boss` 的 `_destroy`（警察叔叔，就是這個函數是我 `console.log` 出來的）給嫁接到 `bullet` 的 `_destroy` 去。

　　這樣會出現什麼樣的結果呢？——當子彈碰到牆的時候就會銷燬，這個時候會觸發 `_destroy` 函數，但是這個時候的 `_destroy` 函數已經會變成了 `boss` 的了，也就是說這個時候子彈不會被銷燬反而是某一個 `boss` 的 `_destroy` 函數被調用然後被銷毀了。

　　再怎麼說這都是 Hack 的辦法，所以並不會觸發 `boss` 的 `onDestroy` 函數也就是說即使所有 `boss` 都沒了也不會出現 `theAlgorithm` 這玩意兒。

> 自己動手豐衣足食！

　　敵人不給我們我們就自己造唄！反正通關判定是——`boss` 數量爲 `0` 且你有 `theAlgorithm` 這個道具。

　　所以說當所有 `boss` 都被銷燬之後，我們自己去 `map.replaceObject` 一個 `theAlgorithm` 道具即可。

```javascript
map.getPlayer().setPhoneCallback(function() {
    var bosses = [];
    var bullets = [];
    var objects = map.getDynamicObjects();
    for(var i = 0; i < objects.length; i++) {
        if(objects[i].getType() == "boss") {
            bosses.push(objects[i]);
        } else {
            bullets.push(objects[i]);
        }
    }
    for(var i = 0; i < Math.min(bosses.length, bullets.length); i++) {
        bullets[i]._destroy = bosses[i]._destroy;
    }

    if(bosses.length === 0) {
        map.placeObject(map.getPlayer().getX(), map.getPlayer().getY() + 1,
            'theAlgorithm');
    }
});
```

　　以上代碼寫完後就開始打 `boss` 吧！趕緊去拿到電話機，然後你會發現打一個電話 `boss` 就少一堆，那感覺倍爽兒！

## End of the Line

　　馬上要通關了。這裏是個坑，開始我還以爲這裏就是真·通關了 QAQ。

　　隨後看看後面還是有關卡啊。但是我突然發現 `<ctrl+0>` 跳出來的 menu 左邊多出了文件夾！然後進去隨意翻看了。

　　最後發現原來是要修改 `scripts/objects.js` 文件→＿→。

　　好吧，分析通關驗證來看，這一關的 `map.finalLevel` 爲 `true`。所以我們只需要把 `scripts/objects.js` 文件裏面的：

```javascript
if(!game.map.finalLevel) {
    game._moveToNextLevel();
}
```

給改成如果是 `finalLevel` 就跑到下一關去就可以了：

```javascript
if(game.map.finalLevel) {
    game._moveToNextLevel();
}
```

## Credits

　　由於事先文章結構沒有寫好，就接這關的坑位來小結吧 0. 0。（反正人家只是序幕章了

　　好的，其實也什麼總結的，但是總覺得得有這麼個小結纔對。

　　找工作啊找工作——有想要我的請聯繫我 2333333333

　　聯繫資料在 [CV](http://xcoder.in/curriculumvitae/) 裏面。
