title: 讓Node.js和C++一起搞基 —— 2
date: 2014-04-03 22:37:15
tags: [ Node.js, C++ ]
category: NodeJS
---

　　好，今天讓我們更深入地搞基吧！
  
## 溫故而知新，可以爲溼矣

　　首先請大家記住這個 V8 的在線手冊——[http://izs.me/v8-docs/main.html](http://izs.me/v8-docs/main.html)。

　　還記得上次的 `building.gyp` 文件嗎？

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "addon.cc" ]
    }
  ]
}
```

　　就像這樣，舉一反三，如果多幾個 `*.cc` 文件的話就是這樣的：

```json
"sources": [ "addon.cc", "myexample.cc" ]
```

　　上次我們把倆步驟分開了，實際上配置和編譯可以放在一起的：

```sh
$ node-gyp configure build
```

　　複習完了嗎？沒？！

![啪](mama.jpg)

　　好的，那我們繼續吧。
  
## 表番

### 函數參數

　　現在我們終於要講參數了呢。

　　讓我們設想有這樣一個函數 `add(a, b)` 代表把 `a` 和 `b` 相加返回結果，所以先把函數外框寫好：

```cpp
#include <node.h>
using namespace v8;

Handle<Value> Add(const Arguments& args)
{
    HandleScope scope;

    //... 又來！
}
```

#### Arguments

　　這個就是函數的參數了。我們不妨先看看 v8 的[官方手冊參考](http://izs.me/v8-docs/classv8_1_1Arguments.html)。
  
+ `int Length() const`
+ `Local<Value> operator[](int i) const`

　　其它的我們咱不關心，這兩個可重要了！一個代表傳入函數的參數個數，另一箇中括號就是通過下標索引來訪問第 `n` 個參數的。

　　所以如上的需求，我們大致就可以理解爲 `args.Length()` 爲 `2`，`args[0]` 代表 `a` 以及 `args[1]` 代表 `b` 了。並且我們要判斷這兩個數的類型必須得是 `Number`。

　　注意到沒，中括號的索引操作符返回結果是一個 `Local<Value>` 也就是 `Node.js` 的所有類型基類。所以傳進來的參數類型不定的，我們必須得自己判斷是什麼參數。這就關係到了這個 `Value` 類型的一些[函數](http://izs.me/v8-docs/classv8_1_1Value.html)了。

+ `IsArray()`
+ `IsBoolean()`
+ `IsDate()`
+ `IsFunction()`
+ `IsInt32()`
+ `IsNativeError()`
+ `IsNull()`
+ `IsNumber()`
+ `IsRegExp()`
+ `IsString()`
+ ...

　　我就不一一列舉了，剩下的自己看文檔。｡:.ﾟヽ(*´∀`)ﾉﾟ.:｡

#### ThrowException

　　這個是我們等下要用到的一個函數。具體在 [v8 文檔](http://izs.me/v8-docs/namespacev8.html#a2469af0ac719d39f77f20cf68dd9200e)中可以找到。

　　顧名思義，就是拋出錯誤啦。執行這個語句之後，相當於在 `Node.js` 本地文件中執行了一條 `throw()` 語句一樣。比如說：

```cpp
ThrowException(Exception::TypeError(String::New("Wrong number of arguments")));
```

　　就相當於執行了一條 `Node.js` 的：

```javascript
throw new TypeError("Wrong number of arguments");
```

#### Undefined()

　　這個函數呢也在[文檔](http://izs.me/v8-docs/namespacev8.html#ad39cfade81e77137fc11ff3a24284340)裏面。

　　具體就是一個空值，因爲有些函數並不需要返回什麼具體的值，或者說沒有返回值，這個時候就需要用 `Undefined()` 來代替了。

#### 動手吧騷年！

　　在理解了以上的幾個要點之後，我相信你們很快就能寫出 `a + b` 的邏輯了，我就把 `Node.js` 官方手冊的代碼抄過來給你們過一遍就算完事了：

```cpp
#include <node.h>
using namespace v8;

Handle<Value> Add(const Arguments& args)
{
    HandleScope scope;
    
    // 代表了可以傳入 2 個以上的參數，但實際上我們只用前兩個
    if(args.Length() < 2)
    {
        // 拋出錯誤
        ThrowException(Exception::TypeError(String::New("Wrong number of arguments")));
        
        // 返回空值
        return scope.Close(Undefined());
    }
    
    // 若前兩個參數其中一個不是數字的話
    if(!args[0]->IsNumber() || !args[1]->IsNumber())
    {
        // 拋出錯誤並返回空值
        ThrowException(Exception::TypeError(String::New("Wrong arguments")));
        return scope.Close(Undefined());
    }
    
    // 具體參考 v8 文檔
    //     http://izs.me/v8-docs/classv8_1_1Value.html#a6eac2b07dced58f1761bbfd53bf0e366)
    // 的 `NumberValue` 函數
    Local<Number> num = Number::New(args[0]->NumberValue() + args[1]->NumberValue());
    
    return scope.Close(num);
}
```

　　函數大功告成！

　　最後把尾部的導出函數給寫好就 OK 了。

```cpp
void Init(Handle<Object> exports)
{
    exports->Set(String::NewSymbol("add"),
        FunctionTemplate::New(Add)->GetFunction());
}

NODE_MODULE(addon, Init)
```

　　等你編譯好之後，我們就能這樣用了：

```javascript
var addon = require('./build/Release/addon');
console.log(addon.add(1, 1) + "b");
```

　　你會看到一個 `2b` ！✧*｡٩(ˊᗜˋ*)و✧*｡
  
### 回調函數

　　上一章我們只講了個 `Hello world`，這一章阿婆主就良心發現一下，再來個回調函數的寫法。

　　慣例我們先寫好框架：

```cpp
#include <node.h>
using namespace v8;

Handle<Value> RunCallback(const Arguments& args)
{
  HandleScope scope;

  // ... 噼裏啪啦噼裏啪啦

  return scope.Close(Undefined());
}
```

　　然後我們決定它的用法是這樣的：

```javascript
func(function(msg) {
    console.log(msg);
});
```

　　即它會給回調函數傳入一個參數，我們設想它是一個字符串，然後我們可以 `console.log()` 出來看。

#### 首先你要有一個字符串系列

　　廢話不多說，先給它一個字符串餵飽了再說吧。_(√ ζ ε:)_

　　不過我們得讓這個字符串是通用類型的，因爲 `Node.js` 代碼是弱類型的。

```cpp
Local<Value>::New(String::New("hello world"));
```

　　什麼？你問我什麼是 `Local<Value>`？

　　那我稍稍講一下吧，參考自[這裏](http://cnodejs.org/topic/4f16442ccae1f4aa270010c5)和[V8參考文檔](http://izs.me/v8-docs/classv8_1_1Local.html)。

　　如文檔所示，`Local<T>` 實際上繼承自 `Handle<T>`，我記得[上一章](/2014/04/02/nodejs-cpp-addons-1/#Handle<Value>)已經講過 `Handle<T>` 這個東西了。

　　然後下面就是講 Local 了。

> Handle 有兩種類型， Local Handle 和 Persistent Handle ，類型分別是 `Local<T> : Handle<T>` 和 `Persistent<T> : Handle<T>` ，前者和 `Handle<T>` 沒有區別生存週期都在 scope 內。而後者的生命週期脫離 scope ，你需要手動調用 `Persistent::Dispose` 結束其生命週期。也就是說 Local Handle 相當於在 C++`在棧上分配對象而 Persistent Handle 相當於 C++ 在堆上分配對象。

#### 然後你要有個參數表系列

　　終端命令行調用 C/C++ 之後怎麼取命令行參數？

```cpp
#include <stdio.h>

void main(int argc, char* argv[])
{
    // ...
}
```

　　對了，這裏的 `argc` 就是命令行參數個數，`argv[]` 就是各個參數了。那麼調用 `Node.js` 的回調函數，`v8` 也採用了類似的[方法](http://izs.me/v8-docs/classv8_1_1Function.html#ac61877494d2d8bb81fcef96003ec4059)：

```cpp
V8EXPORT Local<Value> v8::Function::Call(Handle<Object>recv,
    int argc,
    Handle<Value> argv[]
);
```

> ~~QAQ 卡在了 `Handle<Object> recv` 了！！！明天繼續寫。~~

　　好吧，新的一天開始了我感覺我充滿了力量。(∩^o^)⊃━☆ﾟ.*･｡
  
　　經過我多方面求證（[SegmentFault](http://segmentfault.com/q/1010000000456217)和[StackOverflow](http://stackoverflow.com/questions/22842908/what-does-the-first-argument-of-functioncall-in-v8-engine-mean/22848601?noredirect=1#22848601)以及一個扣扣羣），終於解決了上面這個函數仨參數的意思。

　　後面兩個參數就不多說了，一個是參數個數，另一個就是一個參數的數組了。至於第一個參數 `Handle<Object> recv`，StackOverflow 仁兄的解釋是這樣的：

> It is the same as apply in JS. In JS, you do
>
> ```javascript
var context = ...;
cb.apply(context, [ ...args...]);
```
> The object passed as the first argument becomes this within the function scope. More documentation on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply). If you don't know JS well, you can read more about JS's this here: http://unschooled.org/2012/03/understanding-javascript-this/
>
> <p style="text-align: right;">—— 摘自 [StackOverflow](http://stackoverflow.com/questions/22842908/what-does-the-first-argument-of-functioncall-in-v8-engine-mean/22848601?noredirect=1#22848601)</p>

　　總之其作用就是指定了被調用函數的 `this` 指針。這個 `Call` 的用法就跟 JavaScript 中的 `bind()`、`call()`、`apply()` 類似。

　　所以我們要做的事情就是先把參數表建好，然後傳入這個 `Call` 函數供其執行。

　　第一步，顯示轉換函數，因爲本來是 `Object` 類型：

```cpp
Local<Function> cb = Local<Function>::Cast(args[0]);
```

　　第二步，建立參數表（數組）：

```cpp
Local<Value> argv[argc] = { Local<Value>::New(String::New("hello world")) };
```

#### 最後調用函數系列

　　調用 `cb` ，把參數傳進去：

```cpp
cb->Call(Context::GetCurrent()->Global(), 1, argv);
```

　　這裏第一個參數 `Context::GetCurrent()->Global()` 所代表的意思就是獲取全局上下文作爲函數的 `this`；第二個參數就是參數表中的個數（畢竟雖然 `Node.js` 的數組是有長度屬性的，但是 `C++` 裏面數組的長度實際上系統是不知道的，還得你自己傳進一個數來說明數組長度）；最後一個參數就是剛纔我們建立好的參數表了。
  
#### 終章之結束文件系列

　　相信這一步大家已經輕車熟路了吧，就是把函數寫好，然後放進導出函數裏面，最後申明一下。

　　我就直接放出代碼吧，或者直接跑去 `Node.js` 的[文檔](http://nodejs.org/api/addons.html#addons_callbacks)看也行。
  
```cpp
#include <node.h>
using namespace v8;

Handle<Value> RunCallback(const Arguments& args)
{
    HandleScope scope;
    Local<Function> cb = Local<Function>::Cast(args[0]);
    const unsigned argc = 1;
    Local<Value> argv[argc] = { Local<Value>::New(String::New("hello world")) };
    cb->Call(Context::GetCurrent()->Global(), argc, argv);
    
    return scope.Close(Undefined());
}

void Init(Handle<Object> exports, Handle<Object> module)
{
    module->Set(String::NewSymbol("exports"),
        FunctionTemplate::New(RunCallback)->GetFunction());
}

NODE_MODULE(addon, Init)
```

　　Well done! 最後剩下的步驟就自己去吧。至於 `Js` 裏面這麼調用這個函數，我在[之前](#回調函數)已經提到過了。
  
## 番外

　　嘛嘛，我感覺我的學習筆記寫得越來越奔放了求破～
  
　　今天就先寫到這裏吧，寫學習筆記的過程中我又漲姿勢了，比如說那個 `Call` 函數的參數意義。
  
　　如果你們覺得本系列學習筆記對你們還有幫助的話，就來和我一起搞基吧麼麼噠～Σ>―(〃°ω°〃)♡→