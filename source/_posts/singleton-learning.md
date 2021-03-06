title: C++單件模式：Singleton學習筆記
date: 2010-11-13 15:44:15
tags: [ 老博客備份歸檔, C++, 設計模式 ]
category: 老博客備份歸檔
---

　　單件模式（Singleton）是一種用於確保整個應用程序中只有一個類實例且這個實例所佔資源在整個應用程序中是共享時的程序設計方法（根據實際情況，可能需要幾個類實例）。在某些情況下，這種程序設計方法是很有用處的。

　　在小小地學習了一下C++的單件模式之後，突然聯想到PHP的ThinkPHP的MVC框架，覺得這就是一個單件模式的很好實例吧？

　　我上次在PUDN上載了一個HGE的RPG Demo，裏面就用了Singleton模式還有ObjectFactory模式寫的。沒看懂，於是問了谷歌。

> Singleton可以說是《Design Pattern》中最簡單也最實用的一個設計模式。那麼，什麼是Singleton？
顧名思義，Singleton就是確保一個類只有唯一的一個實例。Singleton主要用於對象的創建，這意味着，如果某個類採用了Singleton模式，則在這個類被創建後，它將有且僅有一個實例可供訪問。很多時候我們都會需要Singleton模式，最常見的比如我們希望整個應用程序中只有一個連接數據庫的Connection實例；又比如要求一個應用程序中只存在某個用戶數據結構的唯一實例。我們都可以通過應用Singleton模式達到目的。
>
> 一眼看去，Singleton似乎有些像全局對象。但是實際上，並不能用全局對象代替Singleton模式，這是因爲：其一，大量使用全局對象會使得程序質量降低，而且有些編程語言例如C#，根本就不支持全局變量。其二，全局對象的方法並不能阻止人們將一個類實例化多次：除了類的全局實例外，開發人員仍然可以通過類的構造函數創建類的多個局部實例。而Singleton模式則通過從根本上控制類的創建，將”保證只有一個實例”這個任務交給了類本身，開發人員不可能再有其它途徑得到類的多個實例。這一點是全局對象方法與Singleton模式的根本區別。
>
> <p style="text-align: right;">——摘自百度百科（我不是有意在谷歌找百度的）</p>

　　我在看了這個RPG Demo的Pattern裏的Singleton之後，仿照着自己寫了一個最簡單的Singleton模板實例。

　　***思想就是，在Singleton中建立一個靜態對象，然後以後就用 `Singleton::Instance()` 來調用這個靜態對象。***

　　***而作爲模板就是可以 `class A : public Singleton` 來讓A繼承Singleton的屬性，那麼我們就可以直接用 
`A::Instance()` 來訪問A這個靜態對象了。這個就是這段Singleton代碼的主要思想了。***

　　先是建立了一個空工程，往裏面放了：
  
　　Singleton.h

```cpp
#pragma once
#ifndef SINGLETON_H
#define SINGLETON_H

template<class T>
class Singleton
{
public:
    static T& Instance();

protected:
    Singleton(){}
    virtual ~Singleton(){}

/**
 * 防止拷貝複製
 */
private:
    Singleton(const Singleton &);
    Singleton & operator = (const Singleton &);
};

template<class T>
T& Singleton::Instance()
{
    /** 建立一個靜態對象 */
    static T instance;
    return instance;
}

#endif
```

　　TestSingleton.h

```cpp
#pragma once

#ifndef TESTSINGLETON_H
#define TESTSINGLETON_H

#include "Singleton.h"
#include <iostream>
using namespace std;

class TestSingleton;

/** 從Singleton繼承本類 */
class TestSingleton : public Singleton
{
private:
    int count;

public:
    TestSingleton(void);
    virtual ~TestSingleton(void);
    void AddCount(){ count++; }
    void CoutCount(){ cout << count << endl; }
};

#endif
```

　　TestSingleton.cpp
  
```cpp
#include "TestSingleton.h"

TestSingleton::TestSingleton(void)
: count(0)
{
}

TestSingleton::~TestSingleton(void)
{
}
```

　　main.cpp

```cpp
#include "TestSingleton.h"
using namespace std;

int main()
{
    for(int i = 0; i < 100; i++)
    {
        /** 單件對象count值加1 */
        TestSingleton::Instance().AddCount();
        /** 輸出此單件對象的count值 */
        TestSingleton::Instance().CoutCount();
    }

    return 0;
}
```

