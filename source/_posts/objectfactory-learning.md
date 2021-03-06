title: C++對象工廠模式：ObjectFactory學習筆記
date: 2010-11-17 15:49:45
tags: [ 老博客備份歸檔, C++, 設計模式 ]
category: 老博客備份歸檔
---

　　對象工廠，顧名思義，就是產生對象的一個“工廠”。根據傳入的一個參數而產生相應的不同種類的對象。

　　用於批量生成同一個父類的不同子類的對象時用到。

　　本學習筆記基於Singleton（單件模式）基礎上進行擴展。

　　看《C++單件模式：Singleton學習筆記》請點擊[鏈接](/2010/11/13/singleton-learning/)。

　　對於工廠模式，網上有很多不同的實現方法。我這裏是一個HGE的RPG Demo中所用的，這段代碼本身寫的非常的好，開始好些語句沒看懂，雖然就這麼幾句話。花了一點時間去研究了其代碼，並自己重新實現了一遍，加上了通俗易懂的註釋。

　　工廠類以模板形式實現，基於Singleton：

```cpp
/**--------------------------------
 * 對象工廠模式(Object Factory)
 *
 * Code by XadillaX
 * http://www.xcoder.in
 * Created at 2010-11-17 1:33
 */
#ifndef OBJECTFACTORY_H
#define OBJECTFACTORY_H
#pragma once
#include <map>
#include <string>
#include "../單件模式/Singleton.h"

template<class T>
class ObjectFactory : public Singleton<ObjectFactory<T>>
{
public:
    typedef T* (*tCreator)();                               ///< 重定義對象生成函數指針
    typedef std::map<std::string, tCreator> tCreatorMap;    ///< 對象生成函數指針map

    /**
     * @brief 註冊新“生產車間”
     * 將生成對象的函數加入對象工廠
     *
     * @param *name 類名稱
     * @param procedure “生產”對象的函數
     * @return 是否成功註冊
     */
    bool Register(char *type, tCreator procedure);

    /**
     * @brief 找到“生產車間”
     * 根據傳入的類名返回相應的新對象的生成函數
     *
     * @param &type 類名
     * @return 相應的新對象的生成函數
     */
    T* Create(const std::string &type);

private:
    /** “生產車間”映射 */
    tCreatorMap _map;
};

template<class T>
bool ObjectFactory<T>::Register(char *type, tCreator procedure)
{
    string tmp(type);
    /** 將新函數加入map中 */
    _map[tmp] = procedure;
    return _map[tmp];
}

template<class T>
T* ObjectFactory<T>::Create(const std::string &type)
{
    /** 在映射中找到相應“生產車間” */
    tCreatorMap::iterator iter = _map.find(type);

    /** 檢測“車間”是否存在 */
    if(iter != _map.end())
    {
        /** 讓返回值爲相應的“生產車間” */
        tCreator r = iter->second;

        /** 返回“生產車間” */
        return r();
    }

    return 0;
}

#endif
```

　　以上就是基於單件模式而實現的工廠模式了。

　　在樣例中，我建立了一個基類Base，然後用A和B來繼承它。

　　在一個for循環中，交替建立了A對象和B對象。這只是一個Demo，看不出有什麼方便的，感覺用一個if來各自生成就好了，就像
  
```cpp
if(type == "A") p = new A();
else p = new B();
```

　　**當然，上面也是一種方法。但是，試想一下，我們將要創建的A、B、C、D、E、F、G類放到一個配置文件中，然後我們從配置文件中讀取這些數據並創建相應的對象，並且這些對象的順序是打亂的，你就要有n個if來判斷了，而且擴展性不高。用一個對象工廠進行封裝的話，儼然形成了一個靜而有序的生產工廠，有秩序地管理着不同的對象車間，不覺得這是一件非常美妙的事情麼？**

　　好了，話不多說，直接上Demo。

```cpp
#include <iostream>
#include "ObjectFactory.h"
using namespace std;
/** 基類 */
class Base;

/** Base類及其子類的對象工廠 */
typedef ObjectFactory<Base> BaseFactory;

class Base
{
public:
    Base(){};
    ~Base(){};
};

class A : public Base
{
public:
    A(){ cout << "An A object created." << endl; };
    ~A(){};
};

class B : public Base
{
public:
    B(){ cout << "A B object Created." << endl; }
    ~B();
};

/** 對象A的“生產車間” */
Base* ACreator()
{
    return new A();
}

/** 對象B的“生產車間” */
Base* BCreator()
{
    return new B();
}

/**
 * @brief 主函數
 */
int main()
{
    /** 將A、B的“生產車間”註冊到對象工廠中 */
    bool AFlag = BaseFactory::Instance().Register("A", ACreator);
    bool BFlag = BaseFactory::Instance().Register("B", BCreator);

    /** 若註冊失敗則退出 */
    if(!AFlag || !BFlag) exit(0);

    Base *p;
    for(int i = 0; i < 10; i++)
    {
        string type = (i % 2) ? string("A") : string("B");

        /** p用相應“生產車間”進行生產 */
        p = BaseFactory::Instance().Create(type);

        delete p;
    }

    return 0;
}
```
