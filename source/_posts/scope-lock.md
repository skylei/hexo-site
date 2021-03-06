title: 線程安全——Scope Lock模式
date: 2012-09-8 01:31:59
tags: [ 老博客備份歸檔, C++, ScopeLock, 線程安全, 線程死鎖 ]
category: 老博客備份歸檔
---

　　***嘛，本文是建立在M$的Visual Studio基礎上的，linux☭勿噴。***

　　我最先用到 ScopeLock 模式是在自己開發 **XAE引擎** 的時候。在裏面用到挺多的線程函數，那麼如何解決臨界區就成了一個重要的課題。可能大家想，不就一個線程鎖臨界區什麼的麼，一個 `EnterCriticalSection` 和一個 `LeaveCriticalSection` 不就解決了麼？

　　其實不然。在 **M$** 中，最常用的當然就是 `CRITICAL_SECTION` 了，但是如果臨界區上鎖卻木有解鎖呢？這就會發生死鎖現象。對於一個粗心的程序猿來說這樣的錯誤還是有機率發生的。就算你足夠細心，還是有時候會一失足成千古恨。

　　所以就有了這麼一種方法來杜絕這種死鎖的產生—— `ScopeLock`。

　　那麼什麼叫 `ScopeLock`？

　　我們試想一下如果有這麼一個類——在構造的時候，你傳進去一個 `CRITICAL_SECTION` 的引用並且將其 `EnterCriticalSection` 進入到臨界區。當它析構的時候，我們直接 `LeaveCriticalSection` 就好了。

　　也許你會問，這樣的一個類會有什麼用呢？

　　那麼我下面演示一段簡單的 ScopeLock 代碼先吧：

```cpp
struct ScopeLock
{
    CRITICAL_SECTION& m_cs;

    ScopeLock(CRITICAL_SECTION cs) : m_cs(cs)
    {
        EnterCriticalSection(&m_cs);
    }

    ~ScopeLock()
    {
        LeaveCriticalSection(&m_cs);
    }
};

int i;
CRITICAL_SECTION cs;

// ...假設我們已經初始化好了這個臨界區

void ScopeLockTest()
{
    ScopeLock oLock(cs);
    i = 0;
}
```

　　我們可以發現，當我們剛進入 `ScopeLockTest` 函數的時候，聲明瞭一個 `oLock` 對象，這個時候運行 `oLock` 的構造函數，也就是進入了 `cs` 這個臨界區。而當 `ScopeLockTest` 函數運行完畢要退出這個函數的時候，`oLock` 對象的生命週期也就走到了盡頭，對應的，它將會執行析構函數，那麼就自然而然地退出了 `cs` 臨界區。

　　其實無論 `ScopeLockTest` 這個函數怎麼寫，哪怕是中間有一些 `if` 判斷直接 `return` 掉，只要是 `ScopeLockTest` 這個函數執行完畢，`oLock` 就會自動析構，從而達到瞭解鎖過程。那麼不管粗心還是細心的童鞋們都不用爲忘記退出臨界區而煩惱了。

　　而且 `ScopeLock` 模式只是一種思想，並不是對於 **M$** 的臨界區的一種專用性物品。例如在QT裏，我們一樣可以用 `ScopeLock` 來對線程的一些 `MutexLock` 之類的東西進行操作。

　　上面所寫的例子只是思路的一種形成，並不是一個完整的ScopeLock類（結構體），雖然說它現在已經可以用了。你可以在上面完善，加上自己的東西，使其能確確實實在項目中使用。由於代碼的關聯性，我單單發出我的 `ScopeLock` 的話會缺少很多關聯的東西，所以咱就不發了，思路在這裏，相信誰都能寫出自己的一個 `ScopeLock` 吧。