title: 「NBUT 2014 校賽·網絡同步賽」題解 (未完成...)
date: 2014-05-05 15:08:39
tags: [ ACM, 算法 ]
category: ACM
---

　　這次比賽由 [Hungar](http://blog.163.com/surgy_han/)，[8Mao](http://www.cnblogs.com/Wine93/) 以及我負責的。明明都讀研了，還詐屍回來出題——歸結起來大概是因爲各種面試不順吧，想來虐虐學弟妹們怒刷存在感。結果網絡賽還是被虐得死去活來。（果然我是蒟蒻 (◓Д◒)✄╰⋃╯

　　好了廢話不多說，還是直接上題解吧。

## Minecraft Server Bug

　　題意大概就是說一排巖漿和水，你要拿一桶水和巖漿，並且水的下標小於巖漿。

　　爲了更便於理解，我們從後往前做。首先將序列讀進來之後從後往前遍歷——若是巖漿，那麼巖漿數加一，如果是水，那麼這桶水能選擇後面巖漿的任意一桶，也就是說答案加上當前的巖漿數即可。

> 注意用 `__int64`。

```cpp
#include <iostream>
using namespace std;

int main()
{
    int n;
    char ch[1000005];
    while(~scanf("%d\n", &n))
    {
        char tmp;
        for(int i = 0; i < n; i++)
        {
            scanf("%c%c", ch + i, &tmp);
        }
        
        __int64 ans = 0;
        int cnt = 0;
        for(int i = n - 1; i >= 0; i--)
        {
            if(ch[i] == 'L') cnt++;
            else ans += (__int64)cnt;
        }
        
        printf("%I64d\n", ans);
    }
    
    return 0;
}
```

## Beautiful Walls

　　一堵牆，每單位高度不定。你需要選擇其中任意連續的牆，使得你選擇的牆每單位的高度都是唯一的——問有多少種選法。

　　先求出總的種數，然後求不滿足的數量，最後用總數減去不滿足數即爲答案。

```cpp
#include <iostream>
#include <cstring>
using namespace std;
#define lint long long
#define N 100005
int p[N], A[N];
lint solution(lint n)
{
    memset(p, -1, sizeof(p));
    lint ans = n * (n + 1) / 2, Max = 0;
    for(lint i=0; i < n; ++i)
    {
        if(~p[A[i]])
        {
            if(Max < p[A[i]]) Max = p[A[i]];
            p[A[i]] = i + 1;
        }
        else
        {
            p[A[i]] = i + 1;
        }
        ans -= Max;
    }
    return ans;
}

int main()
{
    int n, x;
    while(~scanf("%d", &n))
    {
        for(int i = 0; i < n; ++i)
        {
            scanf("%d", A + i);
        }
        printf("%lld\n", solution(n));
    }
    return 0;
}
```