title: 連連看核心算法小分享——1
date: 2011-05-16 12:30:12
tags: [ 老博客備份歸檔, C++, HGE ]
category: 老博客備份歸檔
---

　　**注：這篇文章我到現在也沒有填第二篇的坑。數據沒了重新從 Capture 裏面取出來，看看捨不得，於是把這篇文章也拿回來了。權當紀唸吧，以及當時和 `Kalxd` 的對話。**

## 正文

　　剛忙完邀請賽，蹭了塊銅。剛纔在逛別人博客的時候看別人的文章，突然心血來潮想記一些東西。

　　連連看是我學HGE做的第一個小遊戲，素材用的是QQ的。時間大概是去年國慶吧。好吧，廢話不多說，就講講連連看怎麼找到能消的兩塊吧。

　　首先來回顧一下消方塊的規則，一共有三種可能性：

1. 直線消除（包括水平或者垂直）
2. 一個拐角消除
3. 兩個拐角消除

　　嗯，接下去我們就針對每種可能性開始寫代碼。

　　首先講講一些定義：

　　座標結構體，這個結構體包含了x、y的值以及一些座標中常用的函數。

```cpp
/**
 * @brief 地圖座標結構體
 *
 * 地圖座標結構體，包含x軸值、y軸值
 * 以及一些操作函數。
 */
struct CoorType {
    int x;                                          ///< x軸
    int y;                                          ///< y軸

    /**
     * 重載構造函數
     * 將x、y值各初始化爲-1
     */
    CoorType()
    {
        x = -1, y = -1;
    }

    /**
     * 構造函數重載
     * 將x、y各賦值爲b、a
     * @param a 將要賦值的y軸數值
     * @param b 將要賦值的x軸數值
     */
    CoorType(int a, int b)
    {
        y = a, x = b;
    }

    /**
     * 設置座標
     * 將x、y各賦值爲b、a
     * @param a 將要賦值的y軸數值
     * @param b 將要賦值的x軸數值
     */
    void Set(int a, int b)
    {
        y = a, x = b;
    }

    /**
     * 運算符"+="重載
     * 將此座標與另一座標相加
     * @param &a 另一座標
     * @return 返回結果座標值
     */
    CoorType & operator += (const CoorType &a)
    {
        y += a.y, x += a.x;
        return *this;
    }

    /**
     * 重載運算符"!="
     * 判斷與另一座標是否表示同一個值
     * @param &a 另一座標
     * @return 返回布爾類型表示是否相等
     */
    bool operator != (const CoorType &a)
    {
        if(y != a.y || x != a.x) return false;
        else return true;
    }

    /**
     * 判斷此座標是否合法
     * 若出界則不合法
     * @return 返回布爾類型表示是否合法
     */
    bool isIll()
    {
        if(y >= 0 && x >= 0 && y < MAP_HEIGHT && x < MAP_WIDTH) return true;
        else return false;
    }
};
```

　　然後是關於地圖數組的定義：

```cpp
int Map[MAP_HEIGHT][MAP_WIDTH];
```

　　接着是路徑結構體：

```cpp
/**
 * @brief 路線結構體
 *
 * 合法路線結構體
 * 儲存最多四個點（起點終點和兩個轉折點）
 */
struct PointPath {
    bool bExist;                                    ///< 是否有路徑
    int Num;                                        ///< 駐點個數
    CoorType Points[4];                             ///< 各駐點
};
```

　　接着可以正式開始了。首先我們來想一下，哪些條件各符合上面三種情況的哪一種。對於一條直線的，顯然是x相等或者y相等；對於有一個轉折點的話，我們只需要判斷起點橫向畫線（或者縱向），然後終點縱向畫線（或者橫向），然後從起點到交點以及從交點到終點各可行不；對於兩個轉折點，其中一個轉折點的x或者y跟起點的x或者y相等，另一個轉折點跟終點的x或者y相等。於是這兩個轉折點就根據這樣的性質進行枚舉。因爲連連看的地圖比較小，所以這種O(n^2)的時間複雜度不礙事。

　　爲了方便，我們寫一個 `Abled(CoorType, CoorType, bool, bool);` 函數來進行判斷兩個點（當然兩點是在同一直線上的）是否有同路（即中間沒有東西擋着）。我們先放着這個Abled不管，先實現尋路過程吧。

　　我是用一個CMapSearch類來實現的，聲明如下：

```cpp
/**
 * @brief 地圖搜索類
 *
 * 根據指定地圖搜索出各合法路徑。
 */
class CMapSearch
{
private:
    int Map[MAP_HEIGHT][MAP_WIDTH];                                             ///< 地圖數據矩陣
    PointPath dis[MAP_HEIGHT][MAP_WIDTH][MAP_HEIGHT][MAP_WIDTH];                ///< 路徑數組
    STLMap grap;                                                                ///< STL映射
    CoorType dir[4];                                                            ///< 常量座標增量
    PointPath Hint;                                                             ///< 提示時用的合法路徑

    /**
     * @brief 兩點尋徑
     *
     * 對(x1, y1)和(x2, y2)進行尋徑
     * @param x1 第一個座標的x軸
     * @param y1 第一個座標的y軸
     * @param x2 第二個座標的x軸
     * @param y2 第二個座標的y軸
     * @return 返回一個路線結構體的值，若不存在路徑，則結構體的bExist爲假
     * @see Abled
     */
    PointPath DoSearch(int y1, int x1, int y2, int x2);

public:
    /**
     * @brief 構造函數
     *
     * @param _Map[][Map_Width] 地圖矩陣
     */
    CMapSearch(int _Map[][MAP_WIDTH]);

    /**
     * @brief 析構函數
     */
    ~CMapSearch(void);

    /**
     * @brief 載入地圖
     * 從矩陣中載入地圖到對象
     *
     * @param _Map[][Map_Width] 地圖矩陣
     */
    void LoadMap(int _Map[][MAP_WIDTH]);

    /**
     * @brief 搜索地圖
     * 對整個地圖進行搜索每兩個相同方塊之間的路徑
     *
     * @return 如果存在至少一條路徑則返回真，否則爲假，用於是否重列
     * @see CreateSTLMap
     * @see DoSearch
     */
    bool Search();

    /**
     * @brief 創建map映射
     * 創建一個方塊ID的映射，對每個ID創建一條的該ID的方塊在地圖中的各座標的鏈表
     */
    void CreateSTLMap();

    /**
     * @brief 判斷是否有障礙
     * 對於a、b兩座標（在同一直線）進行判斷期間是否有方塊障礙而導致不能連線
     *
     * @param a 座標a（頭座標）
     * @param b 座標b（尾座標）
     * @param head 若包括頭座標則爲true，否則爲false
     * @param tail 若包括尾座標則爲true，否則爲false
     * @return 若有障礙則返回false，否則爲true
     */
    bool Abled(CoorType a, CoorType b, bool head = false, bool tail = false);

    /**
     * @brief 得到路徑
     * 得到兩個座標的連線具體路徑
     *
     * @param y1 第一個座標的y軸
     * @param x1 第一個座標的x軸
     * @param y2 第二個座標的y軸
     * @param x2 第二個座標的x軸
     * @return 返回一個路線結構體，表示該兩個座標直接的路線
     */
    PointPath GetPath(int y1, int x1, int y2, int x2);

    /**
     * @brief 得到提示路徑
     * 得到一條提示的路徑的相應兩個方塊
     *
     * @param &a 接受第一個方塊ID的變量
     * @param &b 接受第二個方塊ID的變量
     */
    void GetRandomHint(int &a, int &b);
};
```

　　然後我這個分享裏所講的算法就是DoSearch和Abled函數了，因爲其它函數就是用於“提示”道具的。在DoSearch中我們先定義兩個臨時變量，一個是返回值（一個PointPath），四個座標變量：

```cpp
PointPath ans;
CoorType a(y1, x1), b(y2, x2), c, d;
```

　　其中a、b表示起點和終點，c、d表示可能用到的兩個轉折點。

　　首先我們先來判斷直線情況吧，這種情況比較簡單：

```cpp
//如果是直線
if(a.x == b.x || a.y == b.y)
{
    if(Abled(a, b, true, true))
    {
        ans.bExist = true;
        ans.Num = 2;
        ans.Points[0] = a, ans.Points[1] = b;
        return ans;
    }
}
```

　　對於這種情況，我們只需直接判斷a、b直接有沒有通路就好，如果有通路我們就將結果記錄到ans中並返回即可。

　　而有一個轉折點、兩個轉折點的情況以及Abled函數將在下一篇文章中小分享一下。
  
## 回憶時間

　　然後下面就是在這篇文章裏面我跟 `Kalxd` 的對話了，想想現在真是滄海桑田啊。

　　CSS 樣式早已經不在了，截圖裏面是一篇白板

![評論回憶](comment.png)
