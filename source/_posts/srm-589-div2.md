title       : TopCoder SRM 589 DIV 2
category    : Programming
date        : 2013-08-31
tags        : [ TopCoder, SRM ]
---

　　好久沒擼 **TC** 了，手都生了。前兩天剛折騰好 **LinuxMint** + **Cinnamon**，順便手賤把 **TC** 環境配好了。

　　隨便進去扯了一套最新的 **SRM** 來搞，全跑完之後才發現原來這場比賽還處於 **System Running** 階段。於是知道了比賽一結束還在 **Running** 的時候你就已經可以自己拉出來做了。小綠名大家不要笑。


<!-- 我是小小分割符 -->

## Summary

　　這次 **DIV 2** 的難度一般，一道簽到題加兩道普通的 **DP**。

　　[Code on GitHub](https://github.com/XadillaX/xadillax-topcoder/tree/master/code/SRM589-DIV2).

## 250pt - Goose Tattarrattat

題意很簡單，就是給你一個字符串，問你最少改變多少字符讓字符串所有字符都一樣。

簽到題，找最多的字符跟總長度一減就OK了。

{% code cpp %}
#define SIZE(x) ((int)(x.size()))
#define LENGTH(x) ((int)(x.length()))
class GooseTattarrattatDiv2 
{
public:
    int getmin(string S);
};

int GooseTattarrattatDiv2::getmin(string S)
{
    int maxsame = 0;
    map<char, int> mp;
    for(int i = 0; i < LENGTH(S); i++)
    {
    	mp[S[i]]++;
    	maxsame = max(maxsame, mp[S[i]]);
    }
    
    return LENGTH(S) - maxsame;
}
{% endcode %}

## 500pt - Gears

有 ***N*** 個齒輪圍成一圈，相鄰兩個齒輪要反方向轉才能正常運轉不卡到其它輪子。你要從中間拿掉幾個齒輪（留空了就不影響其左邊的左邊的齒輪），問最少拿掉幾個使得所有齒輪能正常轉。

我們建兩個二維 ***dp*** 數組，或者一個三維 ***dp*** 數組：

{% code cpp %}
dp[i][0|1][0|1]
{% endcode %}

第一維 `i` 代表當前是第 `i` 個齒輪。第二維若是 `0` 則表示這個齒輪拿走，若是 `1` 代表留下。第三維若是 `0` 則代表第一個齒輪拿走，`1` 代表第一個齒輪留下。整個數組的每個元素就代表該齒輪留下或者拿走且第一個齒輪是留下或者拿走的情況下的最少拿走齒輪數。

所以我們能得到幾個狀態轉移方程：

> ### 第一個齒輪

{% code cpp %}
dp[i][0][0] = 1;
dp[i][0][1] = INF;
dp[i][1][1] = 0;
dp[i][1][0] = INF;
{% endcode %}

> ### 第二個齒輪
> 
> 如果與第一個同向那麼就有了一留一走或者兩個都走的情況。否則就是四種情況都可以。

{% code cpp %}
if(與第一個齒輪同向)
{
    dp[1][0][1] = 1;
    dp[1][1][0] = 1;
    dp[1][0][0] = 2;
    dp[1][1][1] = INF;
}
else 
{
    dp[i][0][0] = 2;
    dp[i][0][1] = 1;
    dp[i][1][0] = 1;
    dp[i][1][1] = 0;
}
{% endcode %}

> ### 之後的所有齒輪
> 
> 若該齒輪與前一個齒輪方向相同 ，那麼該齒輪留下的時候，前一個齒輪必須得走，那麼就是 `dp[i - 1][0][?]`;該齒輪走的時候，前一個齒輪可走可留，就是 `dp[i - 1][0|1][?] + 1` 的稍微小一點那個。
> 
> 若方向不相同 ，那麼就是該齒輪留下的時候，前一個齒輪也可以留下。

{% code cpp %}
if(與前一個齒輪同向)
{
    dp[i][1][0] = dp[i - 1][0][0];
    dp[i][1][1] = dp[i - 1][0][1];

    dp[i][0][0] = min(dp[i - 1][1][0], dp[i - 1][0][0]) + 1;
    dp[i][0][1] = min(dp[i - 1][1][1], dp[i - 1][1][0]) + 1;
}
else 
{
    dp [i][1][0] = min(dp[i - 1][0][0], dp[i - 1][1][0]);
    dp [i][1][1] = min(dp[i - 1][0][1], dp[i - 1][1][1]);

    dp [i][0][0] = min(dp[i - 1][1][0], dp[i - 1][0][0]) + 1;
    dp [i][0][1] = min(dp[i - 1][1][1], dp[i - 1][1][1]) + 1;
}
{% endcode %}

最後若最後一個齒輪與第一個齒輪同向，那麼在 `dp[i - 1][0][0]`、`dp[i - 1][0][1]`、`dp[i - 1][1][0]` 中挑一個。若不同向，那麼多了個 `dp[i - 1][1][1]` 這個選擇。

下面就是代碼了：

{% code cpp %}
class GearsDiv2 
{
public:
    int getmin(string Directions);
};

int GearsDiv2::getmin(string Directions)
{
    int dp[100][2][2];
    for(int i = 0; i < LENGTH(Directions); i++)
    {
        if(i == 0)
        {
            dp[i][0][0] = 1;
            dp[i][0][1] = 10000000;
            dp[i][1][1] = 0;
            dp[i][1][0] = 10000000;
        }
        else
        if(i == 1)
        {
            if(Directions[i] == Directions[i - 1])
            {
                dp[i][0][1] = 1;
                dp[i][1][0] = 1;
                
                dp[i][0][0] = 2;
                dp[i][1][1] = 10000000;
            }
            else
            {
                dp[i][0][0] = 2;
                dp[i][0][1] = 1;
                dp[i][1][0] = 1;
                dp[i][1][1] = 0;
            }
        }
        else
        if(Directions[i] == Directions[i - 1])
        {
            dp[i][1][0] = dp[i - 1][0][0];
            dp[i][1][1] = dp[i - 1][0][1];
            
            dp[i][0][0] = min(dp[i - 1][1][0], dp[i - 1][0][0]) + 1;
            dp[i][0][1] = min(dp[i - 1][1][1], dp[i - 1][0][1]) + 1;
        }
        else
        {
            dp[i][1][0] = min(dp[i - 1][0][0], dp[i - 1][1][0]);
            dp[i][1][1] = min(dp[i - 1][0][1], dp[i - 1][1][1]);
            
            dp[i][0][0] = min(dp[i - 1][1][0], dp[i - 1][0][0]) + 1;
            dp[i][0][1] = min(dp[i - 1][1][1], dp[i - 1][1][1]) + 1;
        }
    }
    
    int ans;
    int mi = LENGTH(Directions) - 1;
    if(Directions[mi] == Directions[0])
    {
        ans = min(dp[mi][0][0], min(dp[mi][1][0], dp[mi][0][1]));
    }
    else
    {
        ans = min(min(dp[mi][0][0], dp[mi][1][1]), min(dp[mi][0][1], dp[mi][1][0]));
    }
    
    return ans;
}
{% endcode %}

## 1000pt - Flipping Bits

給你一個 **01串** 與一個正整數 ***M***。**01串** 有如下三種操作:

+ 隨便反轉一位（0 -> 1, 1 -> 0）。
+ 將開頭 `k * M` 位反轉。k 可以是任何正整數。
+ 將末尾 `k * M` 位反轉。k 可以是任何正整數。

問最少需要幾步將整個字符串變成都是 `1`。

這又是一個 **DP** 的題目。

我們先設有 ***G*** 組，一組 ***M*** 個 `01字符`。那麼就能有

{% code cpp %}
dp1[i][0|1]
dp2[i][0|1]
{% endcode %}

其中 `i` 代表第 `i` 組，第二維如果是 `0` 就代表這一組採用一位位反轉的操作將這組全變成 `1`，如果是 `1` 則將整組全部反轉再採用一位位反轉的操作將這組全變成 `1` 。至於 `dp1` 和 `dp2` 則代表從頭到尾和從尾到頭。

由於只有 `0` 和 `1` 反轉，那麼一組反轉兩次就能還原原狀——這是一個非常重要的性質。

如果某一組採用**整組反轉**的操作，若前一組也是**整組反轉**，那麼就相當於操作次數不變，只是將前一組的反轉範圍延續到這一組；若前一組是**非整組反轉**，那麼就相當於從頭到這一組反轉之後，前面的所有組再反轉回去——相當於是多了兩次操作。於是就有了（先只拿 `dp1` 作爲例子）：

{% code cpp %}
dp1[i][1] = min(
    dp[i - 1][0] + 這一組1的數量 + 2,
    dp[i - 1][1] + 這一組1的數量 
);
{% endcode %}

如果某一組採用**非整組反轉**，那麼操作次數就是前一組的**整組反轉**或者**非整組反轉**的操作次數加上這一組 `0` 的數量：

{% code cpp %}
dp1[i][0] = min(
    dp[i - 1][0] + 這一組0的數量,
    dp[i - 1][0] + 這一組0的數量
);
{% endcode %}

用上面的轉移方程把正反向都求了一遍之後，我們就可以求總答案了，總答案就是我們枚舉中間只有**操作1**的段的首尾，加上該中間段前部分的 ***dp*** 答案和其後部分的 ***dp*** 答案，取出最小值就是了。

{% code cpp %}
class FlippingBitsDiv2 
{
public:
    int getmin(vector <string> S, int M);
    
    string str;
    int group;
    int tn1[2600], tnsum1[2600];
    int tn2[2600], tnsum2[2600];
    
    int dp1[2600][2];
    int dp2[2600][2];
    
    int calcsum(int l, int r)
    {
        if(l > r) return 0;
        int tot = tnsum1[r] - tnsum1[l - 1];
        return tot;
    }
};

int FlippingBitsDiv2::getmin(vector <string> S, int M)
{
    str = "";
    for(int i = 0; i < SIZE(S); i++) str += S[i];
    group = LENGTH(str) / M;
    ZERO(tn1);
    ZERO(tn2);
    
    // init.
    for(int i = 0; i < group; i++)
    {
        int op = i * M;
        int ed = op + M;
        for(int j = op; j < ed; j++)
        {
            if(str[j] == '0') tn1[i]++, tn2[group - i - 1]++;
        }
        
        dp1[i][0] = 100000;
        dp1[i][1] = 100000;
        dp2[i][0] = 100000;
        dp2[i][1] = 100000;
    }
    
    // tnsum
    for(int i = 0; i < group; i++)
    {
        if(i == 0) tnsum1[0] = tn1[0], tnsum2[0] = tn2[0];
        else
        {
            tnsum1[i] = tnsum1[i - 1] + tn1[i];
            tnsum2[i] = tnsum2[i - 1] + tn2[i];
        }
    }
    
    // dp.
    for(int i = 0; i <= group; i++)
    {
        if(i == 0)
        {
            dp1[i][0] = dp1[i][1] = dp2[i][0] = dp2[i][1] = 0;
        }
        else
        if(i == 1)
        {
            // head -> tail
            dp1[i][0] = tn1[i - 1];
            dp1[i][1] = 1 + (M - tn1[i - 1]);
            
            // tail -> head
            dp2[i][0] = tn2[i - 1];
            dp2[i][1] = 1 + (M - tn2[i - 1]);
        }
        else
        {
            // head -> tail
            dp1[i][0] = min(
                dp1[i - 1][0] + tn1[i - 1],
                dp1[i - 1][1] + tn1[i - 1]
            );
            dp1[i][1] = min(
                dp1[i - 1][0] + 2 + (M - tn1[i - 1]),
                dp1[i - 1][1] + (M - tn1[i - 1])
            );
            
            // tail -> head
            dp2[i][0] = min(
                dp2[i - 1][0] + tn2[i - 1],
                dp2[i - 1][1] + tn2[i - 1]
            );
            dp2[i][1] = min(
                dp2[i - 1][0] + 2 + (M - tn2[i - 1]),
                dp2[i - 1][1] + (M - tn2[i - 1])
            );
        }
    }
    
    int minans = 100000000;
    for(int i = 0; i <= group; i++)
    {
        for(int j = 0; j <= group - i; j++)
        {
            int zzl = i;
            int zzr = group - j - 1;
            
            minans = min(minans,
                min(dp1[i][0], dp1[i][1]) +
                min(dp2[j][0], dp2[j][1]) +
                calcsum(zzl, zzr)
                );
        }
    }
    
    return minans;
}
{% endcode %}