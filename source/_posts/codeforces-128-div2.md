title: CodeForces Round 128 DIV2
date: 2012-07-4 11:38:59
tags: [ 老博客備份歸檔, CodeForces ]
category: 老博客備份歸檔
---

　　這次玩脫了。好不容易四題都做出來，卻因爲小細節掛了兩題。

## Description

### Two Problems

　　題意就是說，CF有兩題，每題初始分A和B，然後每題在每分鐘會扣DA和DB分。給你比賽總時間T，問你某個人可不可能拿到X分。（注意可能做出一題、兩題或者一題也沒做出）

　　(0 ≤ x ≤ 600; 1 ≤ t, a, b, da, db ≤ 300且保證及時比賽時間到了各題的分數也不會小於0)

### Game on Paper

　　有一個 N*N 方格紙。在上面的方格里一格格塗黑。每一步塗一格一共塗 m 次，給定 xi 和 yi。問最少塗幾步方格紙裏會出現一個 3*3 的正方形。

　　(1 ≤ n ≤ 1000, 1 ≤ m ≤ min(n· n, 105))

### Photographer

　　照相內存卡里有d容量。其中高質量照片佔a容量、低質量佔b容量。然後有n個顧客，每個顧客需要xi張高質量照片和yi張低質量照片。攝影師如果給一個人拍照了，就應該滿足他所有要求（即給xi張高質量照片和yi張低質量照片）。問攝影師最多能給幾個人拍照。

　　(1 ≤ n ≤ 105, 1 ≤ d ≤ 109, 1 ≤ a ≤ b ≤ 104, 0 ≤ xi, yi ≤ 105)

### Hit Ball

　　封閉房間裏，從房間的一頭最底下的中間以某個方向踢球（一定是網對面踢），問踢到另一頭的牆上的時候，x、z各是多少。

　　(各座標以及向量都是小於等於100的正整數)

![Hit Ball](hit-ball.png)

### Transportation

　　還沒看。

## Analysis

### Two Problems

　　這題只要注意幾個trick就行了：可以做出0題、1題或者2題。直接兩個for枚舉各題在幾分鐘內做出來，然後做一下0題、1題的特殊判斷就好了。

### Game on Paper

　　在每次塗的時候，以當前塗的點位中心，設它爲九宮格的其中一個位置（一共九種位置），對於每種位置，都判斷其對應的九宮格是不是 3*3 的黑色就好了。(我做的時候在設位置的時候 x - 1, y - 1 手賤敲成了 x - 1, y - 2，lock 之後才發現。悲劇)

### Photographer

　　貪心。對於每個人將其所需的總容量算出來再進行遞增排序。最後求的時候推薦累減的方式判斷，因爲我累加然後用 int 最後爆範圍了。

### Hit Ball

　　首先拿出空間幾何的線面相交模板。然後來一個 `while`，每次循環的時候判斷當前所在的點與方向適量形成的直線與 (X, 0, Z) 面的交點在不在終點牆壁大小的範圍內。若不是則說明中途撞牆了判斷方向向量：x &lt; 0則線面相交判斷是不是撞左牆，若是則 x 正負值變一下；x > 0 則線面相交判斷是不是撞右牆，若是則 x 正負值變一下。z &lt; 0則判斷是不是以求搶地，若是則 z 正負變一下。最後 z > 0 則判斷是不是撞天花板，若是則z正負值變一下。然後以球撞擊的點爲新的起點，與新的方向向量形成新的直線，繼續下一次循環。因爲房間大小最大是 100 * 100 * 100，而方向向量各方向是 1 到 100 的整數，不是小數，則撞擊次數不會很多，直接 `while` 撞擊也不會超。

## Code

### Two Problems

```cpp
#include <iostream>
using namespace std;

int main()
{
    int x, t, a, b, da, db;
    while(~scanf("%d%d%d%d%d%d", &x, &t, &a, &b, &da, &db))
    {
        bool flag = false;
        for(int i = 0; i < t; i++)
        {
            if(x == a - i * da)
            {
                flag = true;
                break;
            }

            for(int j = 0; j < t; j++)
            {
                if(x == b - j * db)
                {
                    flag = true;
                    break;
                }

                if(x == a - i * da + b - j * db)
                {
                    flag = true;
                    break;
                }
            }
            if(flag) break;
        }
        if(0 == x) flag = true;

        printf("%s\n", flag ? "YES" : "NO");
    }

    return 0;
}
```

### Game on Paper

```cpp
#include <iostream>
using namespace std;

bool mat[1015][1015];

bool check2(int sx, int sy)
{
    for(int i = 0; i < 3; i++)
    {
        for(int j = 0; j < 3; j++)
        {
            //if(i == 1 && j == 1) continue;

            if(!mat[sx + i][sy + j]) return false;
        }
    }

    return true;
}

bool check(int x, int y)
{
    if(check2(x, y)) return true;
    if(check2(x, y - 1)) return true;
    if(check2(x, y - 2)) return true;
    if(check2(x - 1, y)) return true;
    if(check2(x - 1, y - 1)) return true;
    if(check2(x - 1, y - 2)) return true;
    if(check2(x - 2, y)) return true;
    if(check2(x - 2, y - 1)) return true;
    if(check2(x - 2, y - 2)) return true;

    return false;
}

int main()
{
    int n, m;
    int x, y;
    while(~scanf("%d%d", &n, &m))
    {
        int ans = -1;
        memset(mat, 0, sizeof(mat));
        for(int i = 0; i < m; i++)
        {
            scanf("%d%d", &x, &y);
            mat[x + 1][y + 1] = true;

            if(ans == -1)
            {
                if(check(x + 1, y + 1))
                {
                    ans = i + 1;
                }
            }
        }

        printf("%d\n", ans);
    }

    return 0;
}
```

### Photographer

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

struct client
{
    int id;
    int memo;
};

bool cmp(client a, client b)
{
    return a.memo < b.memo;
}

client c[100005];

int main()
{
    int n;
    __int64 d;
    int a, b;
    int x, y;

    while(~scanf("%d%I64d", &n, &d))
    {
        scanf("%d%d", &a, &b);
        for(int i = 0; i < n; i++)
        {
            scanf("%d%d", &x, &y);
            c[i].id = i + 1;
            c[i].memo = a * x + b * y;
        }

        sort(c, c + n, cmp);
        __int64 sum = 0L;
        int maxi = -1;
        for(int i = 0; i < n; i++)
        {
            if(sum + c[i].memo <= d) sum += c[i].memo, maxi = i;
            else break;
        }

        printf("%d\n", maxi + 1);
        for(int i = 0; i <= maxi; i++)
        {
            printf("%d%c", c[i].id, i == maxi ? '\n' : ' ');
        }
    }

    return 0;
}
```

### Hit Ball

```cpp
#include <iostream>
#include <math.h>
using namespace std;
#define eps 1e-8
#define zero(x) (((x)>0?(x):-(x))<eps)
struct point3{double x,y,z;};
struct line3{point3 a,b;};
struct plane3{point3 a,b,c;};

point3 xmult(point3 u,point3 v){
    point3 ret;
    ret.x=u.y*v.z-v.y*u.z;
    ret.y=u.z*v.x-u.x*v.z;
    ret.z=u.x*v.y-u.y*v.x;
    return ret;
}

point3 subt(point3 u,point3 v){
    point3 ret;
    ret.x=u.x-v.x;
    ret.y=u.y-v.y;
    ret.z=u.z-v.z;
    return ret;
}

point3 pvec(plane3 s){
    return xmult(subt(s.a,s.b),subt(s.b,s.c));
}

point3 intersection(line3 l,plane3 s){
    point3 ret=pvec(s);
    double t=(ret.x*(s.a.x-l.a.x)+ret.y*(s.a.y-l.a.y)+ret.z*(s.a.z-l.a.z))/
        (ret.x*(l.b.x-l.a.x)+ret.y*(l.b.y-l.a.y)+ret.z*(l.b.z-l.a.z));
    ret.x=l.a.x+(l.b.x-l.a.x)*t;
    ret.y=l.a.y+(l.b.y-l.a.y)*t;
    ret.z=l.a.z+(l.b.z-l.a.z)*t;
    return ret;
}

int main()
{
    double a, b, m;
    double vx, vy, vz;

    while(~scanf("%lf%lf%lf%lf%lf%lf", &a, &b, &m, &vx, &vy, &vz))
    {
        /** 5 walls */
        plane3 door, lw, rw, top, ground;
        door.a.x = 0, door.a.y = 0, door.a.z = 0;
        door.b.x = 1, door.b.y = 0, door.b.z = 0;
        door.c.x = 0, door.c.y = 0, door.c.z = 1;
        lw.a.x = 0, lw.a.y = 0, lw.a.z = 0;
        lw.b.x = 0, lw.b.y = 1, lw.b.z = 0;
        lw.c.x = 0, lw.c.y = 0, lw.c.z = 1;
        rw.a.x = a, rw.a.y = 0, rw.a.z = 0;
        rw.b.x = a, rw.b.y = 1, rw.b.z = 0;
        rw.c.x = a, rw.c.y = 0, rw.c.z = 1;
        ground.a.x = 0, ground.a.y = 0, ground.a.z = 0;
        ground.b.x = a, ground.b.y = 0, ground.b.z = 0;
        ground.c.x = a / 2, ground.c.y = m, ground.c.z = 0;
        top.a.x = 0, top.a.y = 0, top.a.z = b;
        top.b.x = a, top.b.y = 0, top.b.z = b;
        top.c.x = a / 2, top.c.y = m, top.c.z = b;

        line3 l;

        l.a.x = a / 2;
        l.a.y = m;
        l.a.z = 0;

        l.b.x = (a / 2) + vx;
        l.b.y = m + vy;
        l.b.z = vz;

        point3 v;
        v.x = vx, v.y = vy, v.z = vz;

        point3 myans;
        while(true)
        {
            point3 ans, ans1, ans2, ans3, ans4;
            memset(&ans1, 0, sizeof(point3));
            memset(&ans2, 0, sizeof(point3));
            memset(&ans3, 0, sizeof(point3));
            memset(&ans4, 0, sizeof(point3));

            ans = intersection(l, door);
            if(ans.x >= 0 && ans.z >= 0 && ans.x <= a && ans.z <= b)
            {
                myans = ans;
                break;
            }

            point3 totans;

            if(v.x < 0)
            {
                ans1 = intersection(l, lw);
                if(ans1.z >= 0 && ans1.z <= b)
                {
                    v.x = -v.x;
                    totans = ans1;
                }
            }
            else
            if(v.x > 0)
            {
                ans2 = intersection(l, rw);
                if(ans2.z >= 0 && ans2.z <= b)
                {
                    v.x = -v.x;
                    totans = ans2;
                }
            }

            if(v.z > 0)
            {
                ans3 = intersection(l, top);
                if(ans3.x >= 0 && ans3.x <= a)
                {
                    v.z = -v.z;
                    totans = ans3;
                }
            }
            else
            if(v.z < 0)
            {
                ans4 = intersection(l, ground);
                if(ans4.x >= 0 && ans4.x <= a)
                {
                    v.z = -v.z;
                    totans = ans4;
                }
            }

            l.a = totans;
            l.b = totans;
            l.b.x += v.x;
            l.b.y += v.y;
            l.b.z += v.z;
        }

        printf("%.10lf %.10lf\n", myans.x, myans.z);
    }

    return 0;
}
```
