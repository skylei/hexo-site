title: ～公衆檔所～項目解析
date: 2014-03-27 01:34:04
category: Document
tags: [ Node.js, Project Parse ]
---

　　所謂“公衆檔所”，其實就是一個公共的臨時網盤了。這個東西是一個老物了，在我剛接觸 `Expressjs` 的時候寫的。當時還隨便搞了一下 `backbone.js`，但是沒有深入，勿笑。關於深入構架 `Expressjs` 方面也沒做，只是粗粗寫了下最基礎的路由，所以整個文件結構也不是很規範。但是應該能比較適合剛學 `Node.js` 以及剛接觸 `Expressjs` 的人吧。

　　Repo地址在[我的Github](https://github.com/XadillaX/public-file-house)上。Demo地址在 [http://dang.kacaka.ca/](http://dang.kacaka.ca/)，由於個人電腦的不穩定性，所以不保證你們隨時可以訪問，保不定哪天就失效了，所以最好的辦法還是自己 `clone` 下來啪啪啪。

　　它所需要的東西大致就是 `Expressjs` + `Redis` + `Backbone` 了。不過都是最最基礎的代碼。

## 部署

　　把部署寫在最前面是爲了能讓你們自己電腦上有一個能跑的環境啦。公衆檔所在我自己這邊的環境裏面是由三臺電腦組成的。

+ 網關“服務器”。這是我這邊環境一致對外的機器。實際上是一片樹莓派，裝了 `nginx`，然後對內部做反向代理。
+ 本體“服務器”。跑了 **公衆檔所** 本體。
+ 數據庫“服務器”。我們用的數據庫實際上不是嚴格意義上的數據庫，只是 `redis` 罷了，也沒做與其它數據庫的持久化，只是用了他內部自帶的持久化。

　　**如果你們裝一臺機子上，那麼就是：**

　　將 `repo` 給 clone 到自己的機子上。

{% code shell %}
$ git clone https://github.com/XadillaX/public-file-house
{% endcode %}

　　裝好 **redis**，並根據需要修改 `redis.conf` 文件。
  
　　執行 `redis.sh` 文件開啓數據庫。如果你自己本身已經開啓數據庫或者用其它方法開啓了，請忽略上面數據庫相關步驟。
  
　　然後打開 `commonConst.js` 文件進行編輯，把相關的一些信息改成自己所需要的。

　　哦對了，還有一個“潔癖相關”的步驟。我以前年輕不懂事，把 `node_modules` 文件夾也給加到版本庫中了，而且也在裏面居然自己加了兩個沒有弄到 `nmp` 去的模塊（**而且這兩個模塊本來就不應該放在這個文件夾下，但是不要在意這些細節，反正我現在肯定不會做這麼傻的事了**）。
  
　　至於爲什麼不要這麼做，就跟 `node_modules` 文件夾的意義相關了。而且裏面有可能有一些在我本機編譯好的模塊，所以最好還是清理下自己重新裝一遍爲佳。

　　具體呢大致就是把 `node_modules` 文件夾裏面的 `alphaRandomer.js` 文件和 `smpEncoder.js` 文件拷貝出來備份到任意文件夾，然後刪除整個 `node_module` 文件夾。接下去跑到項目根目錄執行：
  
{% code shell %}
$ npm install
{% endcode %}
  
　　把三方模塊重新裝好之後，把剛纔拷出去的倆文件放回這個目錄下。（**但是以後你們自己寫別的項目的話千萬別學我這個壞樣子啊，以前年輕不懂事 QAQ**）
  
　　最後跑起來就行啦：

{% code shell %}
$ node pfh.js
{% endcode %}

## 解析

接下去就是要剖析這小破東西了。

### 基礎文件

#### pfh.js

　　這個文件其實是 `Expressjs` 自動生成的，以前不是很懂他，所以也沒怎麼動，基本上是保持原封不動的。

#### router.js

　　這個是路由定義的文件。比較醜陋的一種方法，把需要定義的所有路由都寫進兩個 `json` 對象中，一個 `POST` 和一個 `GET`。

　　看過 `Expressjs` 文檔的人或者教程的人都知道，最基礎的路由註冊寫法其實就是：

{% code javascript %}
app.get(KEY, FUNCTION);
{% endcode %}

　　或者：

{% code javascript %}
app.post(KEY, FUNCTION);
{% endcode %}

　　所以我下面有一個函數：

{% code javascript %}
exports.setRouter = function(app) {
    for(var key in this.getRouter) {
        app.get(key, this.getRouter[key]);
    }

    for(var key in this.postRouter) {
        app.post(key, this.postRouter[key]);
    }
};
{% endcode %}

　　其大致意思就是把之前我們定義好的兩個路由對象裏的內容一一給註冊到系統的路由當中去。這個是我最初最簡陋的思想，不過後來我把它稍稍完善了一下寫到[別的地方](https://github.com/XadillaX/exframess/blob/master/config/router.js#L17)去了。
  
### 模型

#### model/fileModel.js

　　這個就是模型層了，主要就是 `redis` 的一些操作了。在這裏我用的是 [`redis`](https://github.com/mranney/node_redis) 這個模塊，具體的用法大家可以看它 `repo` 的 `README.md` 文件。
  
　　大致就三個函數：
  
1. `fileModel.prototype.keyExists`: 判斷某個提取碼存在與否。
2. `fileModel.prototype.get`: 獲取某個驗證碼的文件信息。
3. `fileModel.prototype.addFile`: 添加一個文件信息。

> 不過有個壞樣子大家不要學，`Node.js` 大家都約定俗成的回調函數參數一般都是 `callback(err, data, blahblah...)` 的，第一個參數都是錯誤，如果沒錯誤都是 `null` 或者是 `undefined` 的。但是以前也沒這種意識，所以回調函數的參數也都是比較亂的。

### 控制器

#### action/index.js

　　這是一些基礎控制器。
  
##### exports.index

　　純粹的首頁顯示。

##### exports.download

　　文件下載控制器。由代碼可知，首先獲取 `token` 和 `code`。 `token` 是驗證 **URL** 的有效性而 `code` 即提取碼了。

　　期間我們驗證了下 `token`：

{% code javascript %}
if(!functions.verifyBlahblah(token)) {
    resp.redirect(baseConfig.webroot);
}
{% endcode %}

　　而這個 `verifyBlahblah` 函數就在[這個文件](https://github.com/XadillaX/public-file-house/blob/master/plugin/functions.js#L21)裏面。

{% code javascript %}
exports.verifyBlahblah = function(blahblah) {
    var array = blahblah.split("^");
    var time = array[array.length - 1];
    array.pop();

    var encoder = require("smpEncoder");

    try {
        var text = encoder.norBack(array, time.toString());
        text = encoder.decode(text);
    } catch(e) {
        return false;
    }

    var now = text.substr(0, 10);
    var token = text.substr(10);

    if(parseInt(Date.now() / 1000) - parseInt(now) > 300) return false;
    if(token !== require("../commonConst").token) return false;

    return true;
};
{% endcode %}

　　大體意思就是把其打散到數組裏面，其中時間戳是最後一位。然後解密。最後驗證解密後的 `token` 是否等於系統的 `token` 以及時間戳有沒有過期。

　　大家通過截取 `Chrome` 或者 `Firefox` 的請求信息，不難發現有這麼個地址：

{% code %}
Request URL:http://localhost/download?file=662ZE&token=65^97^74^68^106^125^88^115^65^96^66^105^127^114^87^123^123^114^84^124^114^125^120^121^99^116^100^118^116^98^124^120^109^98^120^100^80^119^120^87^119^105^116^8^1395904110
Request Method:GET
Status Code:200 OK
{% endcode %}

　　而這一坨 `65^97^74^68^106^125^88^115^65^96^66^105^127^114^87^123^123^114^84...^1395904110` 便是所謂的 `token` 了。而且本來就是個demo，這個 `token` 也就是隨便做做樣子罷了。

　　接下去通過驗證之後，便可以從數據庫中讀取文件信息了。如果有文件，那麼通過 `resp.download` 函數呈現給用戶。

{% code javascript %}
var fileModel = new FileModel();
fileModel.get(code, function(status, error, obj) {
    if(error) resp.redirect(baseConfig.webroot);
    else {
        if(obj === null) {
            resp.redirect(baseConfig.webroot + "/get/" + code + "/not-exist");
        } else {
            resp.download(baseConfig.uploadDir + code, require("urlencode")(obj.filename));
        }
    }
});
{% endcode %}

##### exports.getToken

　　這個函數就是生產一個有效的 `token` 用的。在前端是通過 **ajax** 來獲取的。

{% code javascript %}
var encoder = require("smpEncoder");
var token = baseConfig.token;
var now = parseInt(Date.now() / 1000);
var result = encoder.encode(now + token);
result = encoder.norGo(result, now.toString());
var resultString = "";
for(var i = 0; i < result.length; i++) resultString += (result[i] + "^");
{% endcode %}

　　大體呢就是根據目前的時間戳和系統 `token` 一起加密生產一個有效的 `token`。

##### exports.send2fetion

　　通過自己的飛信給自己發送提取碼以備忘。

　　這裏的話用了一個 `fetion-sender` 的模塊。`Repo` 在[這裏](https://github.com/XadillaX/fetion-sender)。

#### action/upload.js

　　這個文件裏面其實就一個 `exports.upload` 函數，另一個是生成提取碼用的。

##### function genAlphaKey(time, callback)

　　生成提取碼。我們假設最多嘗試10次，若嘗試10次還沒有生成唯一的驗證碼就輸出錯誤讓用戶重試。所以就有了：

{% code javascript %}
function genAlphaKey(time, callback) {
    var keyLength = config.uploadLen;
    var filename = alphaRandomer.rand(keyLength);
    var fileModel = new FileModel();

    fileModel.keyExists(filename, function(status, result) {
        if(!status) {
            if(time < maxTryTime) {
                genAlphaKey(time + 1, callback);
            }
            else {
                callback(false, result, "");
            }

            return;
        } else {
            if(result) genAlphaKey(time, callback);
            else {
                callback(true, "", filename);
            }
        }
    });
}
{% endcode %}

　　不斷地生成定長的提取碼，然後通過模型的 `keyExists` 函數來確定這個提取碼是否存在，如果存在了就遞歸調用重新生成，否則就直接回調。
  
##### exports.upload

　　上傳文件的頁面了。

{% code javascript %}
if(req.files.files.length !== 1) {
    result.status = false;
    result.msg = "請用正確的姿勢餵我文件。";
    resp.send(200, result);
    return;
}

var fileInfo = req.files.files[0];
if(fileInfo.size > config.maxUploadSize) {
    result.status = false;
    result.msg = "文件太大啦，公衆檔所一次只能吃10M的文件哦。";
    resp.send(200, result);
    return;
}
{% endcode %}

　　前面一堆話大致就是做下有效性判斷而已。然後調用函數來生成有效的提取碼：

{% code javascript %}
genAlphaKey(1, function(status, msg, filename) {
    ...
});
{% endcode %}

　　如果生成成功的話就往數據庫中添加文件信息：

{% code javascript %}
var fileModel = new FileModel();
fileModel.addFile(filename,
    fileInfo.name,
    fileInfo.headers["content-type"],
    function(status, msg) {
        ...
    }
);
{% endcode %}

　　如果添加也成功了的話，那麼把剛上傳到臨時文件夾的文件給移動到上傳文件儲存目錄中，以便以後可以被下載：

{% code javascript %}
fs.rename(fileInfo.path, uploadDir + filename, function(err) {
    ...
});
{% endcode %}

　　如果移動也成功了的話，那麼返回一個成功的json信息：

{% code javascript %}
result.status = true;
result.code = filename;
resp.send(200, result);
{% endcode %}

### 視圖

　　這裏視圖就一個 `index.ejs` 。然後通過 `backbone.js` 來調用不同的頁內模板和邏輯來實現的類似於 **SPA ~~(Solus Par Agula)~~ (Single Page Application)** 的效果。

####  views/index/index.ejs

　　像類似於下面的這種就是 `backbone.js` 的模板概唸了：

{% code html %}
<script type="text/template" id="faq-template">
...
</script>
{% endcode %}

　　到時候就可以通過 `backbone.js` 中的函數來填充到頁面實體當中去。

#### public/js/index.js

　　在擁有了所有的前端js依賴之後，這個文件就是這個 `SPA` 的入口了。

　　邏輯很簡單：

{% code javascript %}
var workspace = null;
$(function() {
    workspace = new Workspace();
    Backbone.history.start({ pushState: true, hashChange: false });
});
{% endcode %}

　　新建一個 `Workscpace`，然後對 `backbone` 進行一點配置。

> To indicate that you'd like to use HTML5 pushState support in your application, use Backbone.history.start({pushState: true}). If you'd like to use pushState, but have browsers that don't support it natively use full page refreshes instead, you can add {hashChange: false} to the options.
>
> <p style="text-align: right">——摘自 [backbonejs.org](http://backbonejs.org/#History-start)</p>

　　然後這個 `Workspace` 即這個 `SPA` 的本體了。

#### public/backbone/router/workspace.js

　　這裏定義了幾個路由，即什麼路由要用哪個類去處理。這樣才能在 `URL` 當中各種跳轉。其實無非就是把待渲染元素渲染成頁內模板，然後把頁面的各種事件響應邏輯改掉即可。對於 `Backbone` 我其實只用過兩次，現在也忘不大多了，怕誤人子弟，所以一些具體的函數啊用法啊還是去參考下官網比較好來着。

#### public/backbone/view/*.js

　　就是各路由所對應的視圖了。

##### uploadView.js

　　比如說 `uploadView.js` 文件當中，執行渲染函數：

{% code javascript %}
    ...,
    
    render      : function() {
        $(this.el).html(Mustache.to_html(
            this.template
        ));

        $("#uploadfile").fileupload({
            url         : "../../upload.pfh",
            dataType    : "json",
            done        : this.uploaded,
            progressall : this.processUpload,
            start       : this.startUpload
        });

        $(".template").show("normal");

        return this;
    },
    
    ...
{% endcode %}

　　就是用頁內模板來渲染：

{% code javascript %}
$(this.el).html(Mustache.to_html(
    this.template
));
{% endcode %}

　　而這個 `this.el` 是在 `Workspace` 中定義的：

{% code javascript %}
    ...,
    
    upload      : function() {
        var uploadView = new UploadView({ el: "#main-template-container" });
        uploadView.render();
    },
    
    ...
{% endcode %}

　　如你所見，就是這個 `#main-template-container` 了。
  
　　這個渲染完畢之後，然後把 `#uploadfile` 給變成上傳按鈕（用了 **jquery.fileupload.js**）。再然後把渲染好的頁面給 `show` 出來。

　　然後這個　`uploadView.js` 中還定義了兩個[響應事件](http://backbonejs.org/#View-delegateEvents)：

{% code javascript %}
    ...,
    
    events      : {
        "click .upbutton"   : "upload",
        "click #uploadpage-to-download" : "goDownload"
    },
    
    ...
{% endcode %}

　　即在按下 `.upbutton` 的時候會執行 `upload` 函數，在按下“去下載”的按鈕時會執行 `goDownload` 函數。

{% code javascript %}
    ...,
    
    upload      : function() {
        $("#uploadfile").click();
    },
    
    ...
{% endcode %}

　　執行上傳函數的時候，實際上是自動觸動了 `#uploadfile` 按鈕的 `click` 事件。這個時候就會按照之前定義好的 `$("#uploadfile").fileupload(...)` 去處理了。

##### getView.js

　　這個是獲取文件的視圖。

　　渲染時會獲取 `code` 。這個 `code` 同樣是 `Workspace` 傳入的：

{% code javascript %}
    ...,
    
    get         : function(code, err) {
        var getView = new GetView({ el: "#main-template-container" });
        getView.setCode(code);
        if(err !== undefined) getView.setError(err);
        getView.render();
    },
    
    ...
{% endcode %}

　　上面關於 `get` 的路由是 `get/:code` 之類的，所以這個 `code` 會作爲一個路由參數傳給 `get` 函數。
  
　　有了這個 `code` 之後就可以把頁面渲染出來了。這就是爲什麼我們地址輸入 `http://localhost/get/XXXXX` 的時候輸入框裏面就有提取碼了。把這個渲染出來之後，我們對“二維碼”的兩張圖片做下響應：鼠標移動上去會顯示出來。再然後我們要獲取二維碼了（`this.genQRCode()`）：

{% code javascript %}
    ...,
    
    genQRCode   : function() {
        var code = this.code;

        var dpage = "http://dang.kacaka.ca/get/" + code;
        dpage = UrlEncode(dpage);
        var img = '<img style="width: 150; height: 150;" src="https://chart.googleapis.com/chart?cht=qr&chs=150x150&choe=UTF-8&chld=L|4&chl=' + dpage + '" />';

        $("#download-page-qr").attr("data-content", img);

        var opage = "http://dang.kacaka.ca/download?";
        this.getToken(function(token) {
            if(undefined === token) {
                $("#download-origin-qr").attr("data-content", '<div style="text-align: center;">二維碼生成失敗。</div>');
                return;
            }

            opage += "token=" + token;
            opage += "&file=" + code;
            opage = UrlEncode(opage);
            var img = '<div style="text-align: center;"><img style="width: 150; height: 150;" src="https://chart.googleapis.com/chart?cht=qr&chs=150x150&choe=UTF-8&chld=L|4&chl=' + opage + '" /><br /><small>該二維碼有效期五分鐘。</small></div>';
            $("#download-origin-qr").attr("data-content", img);
        });

        if("" === this.code) {
            $("h2 small").css("display", "none");
        } else $("h2 small").css("display", "inline-block");
    }
{% endcode %}

　　無非就是調用谷歌的 API 然後生成圖片地址放上去罷了。一個地址就是當前頁面地址，另一個就是加上 `token` 之後的直接下載地址。

　　如你所見，獲取token是通過ajax往服務器請求的：

{% code javascript %}
    ...,
    
    getToken    : function(callback) {
        $.get("../../blahblah", {}, function(e) {
            callback(e.token);
        });
    },
    
    ...
{% endcode %}

　　然後事件的話：

{% code javascript %}
    ...,
    
    events      : {
        "click #downloadpage-to-upload" : "toUpload",
        "click #download-btn"   : "toDownload",
        "keydown #download-code": "toDownloadKeydown",

        "keyup #download-code"  : "navCode"
    },
    
    ...
{% endcode %}

　　按了“去上傳”按鈕會跑去上傳。如果按下“下載”按鈕就下載文件了。然後輸入框裏面彈起鍵盤的話，會導致輸入框文字變化，這個時候就要更新二維碼以及URL了。

{% code javascript %}
    ...,
    
    navCode     : function() {
        var code = $("#download-code").val();
        workspace.navigate("get/" + code);
        this.code = code;

        if(code === "") {
            $("h2 small").css("display", "none");
        } else {
            this.genQRCode();
        }
    },
    
    ...
{% endcode %}

　　每當輸入框變化之後，地址欄就要變成新的 `get/:code` (`workspace.navigate("get/" + code)`) 了，然後重新獲取一遍二維碼。

　　下載按鈕的邏輯代碼如下：

{% code javascript %}
    ...,
    
    toDownload  : function() {
        var code = $("#download-code").val();
        workspace.navigate("get/" + code);
        this.getToken(function(token) {
            if(token === undefined) {
                alert("獲取驗證信息失敗，請稍後重試。");
            } else {
                var url = "../../download?file=" + code + "&token=" + token;
                window.location.href = url;
            }
        });
    },
    
    ...
{% endcode %}

　　反正就是根據 `code` 來生成地址，然後從獲取token的地址中把token拿出來拼接成下載地址之後再訪問（`window.location.href = url`）就好了。

##### uploadedView.js

　　這個視圖是上傳成功視圖。功能很簡單，就是現實下提取碼，然後飛信能發送一下，以及能複製驗證碼罷了。

{% code javascript %}
    ...,
    
    render      : function() {
        if(undefined === this.code) {
            workspace.navigate("upload", { trigger: true, replace: true });
            return;
        }

        $(this.el).html(Mustache.to_html(
            this.template,
            { code: this.code }
        ));

        var phoneinfo = store.get("fetion-info");
        if(undefined !== phoneinfo) {
            $("#phonenumber").val(phoneinfo.phonenumber);
            $("#password").val(phoneinfo.password);
        }

        $(".template").show("normal", function() {
            $('#copy-code-btn-parent').zclip({
                path:'../../ZeroClipboard.swf',
                copy:function() {
                    return $("#code-input").val();
                },
                afterCopy: function() {
                    alert("提取碼已經成功複製到剪切板了。");
                }
            });
        });

        return this;
    },
    
    ...
{% endcode %}

　　通過判斷有沒有 `code` 來判斷是否上傳成功。這個 `code` 的來源是 `uploadView.js` 中的 `uploaded (done: this.uploaded)` 函數：

{% code javascript %}
    ...,
    
    uploaded    : function(e, data) {
        var result = data.result;
        if(!result.status) {
            $("#progress").css("display", "none");
            $("#progress .progress-bar").html("已上傳 0%");
            $("#progress .progress-bar").attr("aria-valuenow", "0");
            $("#progress .progress-bar").css("width", "0%");

            $("#upload-div #feed-doc").removeClass("alert-info");
            $("#upload-div #feed-doc").addClass("alert-danger");
            $("#upload-div #feed-doc").html(result.msg);
            $("#upload-div #feed-doc").css("display", "block");
            return;
            return;
        } else {
            store.set("code", result.code);
            workspace.navigate("uploaded", { trigger: true, replace: true });

            return;
        }
    },
    
    ...
{% endcode %}

　　`e` 和 `data` 這兩個參數哪來？首先這個 `uploaded` 函數是在之前渲染的時候定義成 `jquery.fileupload` 的上傳結束回調函數的，所以這兩個參數自然是 `jquery.fileupload` 傳過來的。詳見[這裏](https://github.com/blueimp/jQuery-File-Upload/wiki/Options#done)。
  
　　總之就是上次成功之後，這個upload函數會獲取一個 `code`，然後它就會拿這個 `code` 存到 `store` 中。這個 `store.js` 是一個 `localStorage` 的封裝。它的代碼和文檔在[這裏](https://github.com/cloudhead/store.js)。

　　存好之後讓 `Workspace` 給導航到 `uploaded` 視圖中。

　　而這個 `uploaded` 視圖的初始化函數裏面有這樣的代碼：

{% code javascript %}
    ...,
    
    initialize  : function() {
        this.code = store.get("code");
        var self = this;

        $("#sending").click(self.sending);
        $("#cancel-sending").click(self.cancelSending);
        $("#phonenumber, #password").keydown(function(e) { if(e.keyCode === 13) self.sending(); });
    },
    
    ...
{% endcode %}

　　就是初始化的時候，從 `localStorage` 中把 `code` 給取出來。

## 結束

　　代碼量少，用到的東西也是基礎；不過以前的代碼由於不瞭解 `Node.js` 啊 `Expressjs` 啊等等的，所以導致代碼雜亂無章、髒亂無比，所以一定程度上阻礙了可讀性的存在。

　　希望本文能給各位看官稍稍理清思路。我也不必寫得面面俱到，只是在某個程度上點題一下而已。更多的大家自己看代碼即可了。不過希望還不要把大家給誤導了就好，畢竟這代碼我自己現在看覺得好丟臉啊 QAQ。大家就去其糟粕取其精華吧。（喂喂喂，我去年買了個表，哪有什麼精華啊！