title: HGE做格鬥遊戲的熱點圖片碰撞檢測法
date: 2011-10-18 15:22:21
tags: [ 老博客備份歸檔, C++, HGE ]
category: 老博客備份歸檔
---

　　碰撞檢測始終是做2D遊戲中的一個熱點話題，我本人並沒有做過這類遊戲，所以一切只是理論而已，不過正打算做這麼個小遊戲練練手。

　　前幾天在HGE的羣裏看到有人突然問到如何判斷鼠標有沒有點到人（點到紋理的透明區域不算），從而引申出了碰撞檢測問題。

　　他的問題相對好實現，只要算出紋理所按的點是不是透明即可。

　　接下來我得做下碰撞檢測的筆記：

　　碰撞檢測最常用一個方法就是關節設置（當然我並沒有做過），關節設置的話因爲只是判斷多邊形的重疊狀況，算法的複雜度低、效率高，雖然做工有點粗，但總體效果還是性價比比較高的一種方法。當然，這樣的方法需要對每一幀的紋理都設置一個關節，對於人工的代價就稍微大了一些了，並且還要寫個關節編輯器啊神馬的，於是乎代碼量又增加了。我這次是和同寢室木有一點基礎的童鞋一起練手的，所以並沒有打算引進這個方法。

　　於是我就用了另一種稍微“非主流”一些的方法了——逐像素判斷。

　　但是逐像素判斷還是有問題的——如果你的一個“效果”因爲“溫度過高”而不需要顯示，直接隱藏，但又算傷害，這時紋理的逐像素就失去了意義。於是又有了個“臃腫”的辦法，爲需要“額外附加像素”的紋理另做一張圖片，這張圖片上有兩種區域——熱點區和非熱點區。我們把需要“當做空氣”的那些區域一律用某一種極其不常用的顏色覆蓋，如 `ff00ff` 這種變態的粉色，然後其它區域的顏色就隨你怎麼搞了。我們載入的時候兩張紋理一起載入，顯示的時候顯示正常的紋理，而在碰撞檢測的時候用“熱點圖片”來進行逐像素檢測。

　　與上面的關節設置法比較的話，人工的工作量我個人認爲是大大地減少了，至於對於機器的執行能力來說，把時間複雜度提到了 `O(mn)` ，平方級的複雜度了，即紋理相交區域的寬和高。

　　我們來看一下這種碰撞檢測的大體流程吧：

1. 獲得兩個精靈的矩形，並得到相交矩形。若無相交則直接返回 `false`。
2. 根據相交矩形，我們可以得到精靈1、2的紋理中需要檢測的初始座標。
3. 將精靈1、精靈2的熱點圖片的相交區域的那一部分像素拷貝出來備用。（因爲有可能兩個紋理句柄是一樣的，不好同時 `lock`）
4. 開始對於拷貝出來的像素信息逐一判斷對應像素點是否都“不是空氣”，若都“不是空氣”則可以判斷爲碰撞。

　　當然以上的流程我們還可以優化一下，省去拷貝的那一段時間。我們可以直接 `hge->Texture_Lock()` 來進行得到兩個紋理的像素信息的首指針，如果兩個紋理其實只是一個紋理的話，則只需 `hge->Texture_Lock()` 一次，而另一個指針也只想 `hge->Texture_Lock` 即可，然後直接開始判斷。

　　下面獻上我這個函數的實現以及測試代碼和素材：

```cpp
/**
 * @brief   Test the collision by the "hot" texture
 * @author  XadillaX
 * @email   admin@xcoder.in
 * @date    2011/10/18
 * @http://xcoder.in
 *
 * @param spr1 The first sprite to test the collision
 * @param x1 "x" of top-left corner of sprite 1
 * @param y1 "y" of top-left corner of sprite 1
 * @param spr2 The second sprite to test the collision
 * @param x2 "x" of top-left corner of sprite 2
 * @param y2 "y" of top-left corner of sprite 2
 * @param hot1 The "hot" texture for sprite 1. It will be the default texture of spr1 if it equal to 0
 * @param hot2 The "hot" texture for sprite 2. It will be the default texture of spr2 if it equal to 0
 * @param airColor The color which considered of "air"
 *
 * @return if they are collided, return true
 */
bool IsCollision(hgeSprite* spr1, float x1, float y1, hgeSprite* spr2, float x2, float y2, HTEXTURE hot1 = 0, HTEXTURE hot2 = 0, DWORD airColor = 0xffff00ff)
{
    /** Set the rect */
    hgeRect r1, r2;
    r1.Set(x1, y1, x1 + spr1->GetWidth(), y1 + spr1->GetHeight());
    r2.Set(x2, y2, x2 + spr2->GetWidth(), y2 + spr2->GetHeight());

    /** Test for the intersect of rectangles */
    if(r1.Intersect(&r2))
    {
        int x[] = { x1, x2, x1 + spr1->GetWidth(), x2 + spr2->GetWidth() };
        int y[] = { y1, y2, y1 + spr1->GetHeight(), y2 + spr2->GetHeight() };
        std::sort(x, x + 4);
        std::sort(y, y + 4);
        hgeRect r;

        /** Set the rectangle area where the two rectangles intersected. */
        r.Set(x[1], y[1], x[2], y[2]);

        /** The start point of sprite1 and sprite2. (From the intersected area) */
        int sx1, sy1, sx2, sy2;
        sx1 = x[1] - x1;
        sy1 = y[1] - y1;
        sx2 = x[1] - x2;
        sy2 = y[1] - y2;

        /** Get the "hotspot" of texture */
        HTEXTURE hTex1 = hot1;
        HTEXTURE hTex2 = hot2;
        if(hTex1 == 0) hTex1 = spr1->GetTexture();
        if(hTex2 == 0) hTex2 = spr2->GetTexture();

        float tx1, ty1, tw1, th1, tx2, ty2, tw2, th2;
        int w1 = hge->Texture_GetWidth(hTex1), w2 = hge->Texture_GetWidth(hTex2);
        spr1->GetTextureRect(&tx1, &ty1, &tw1, &th1);
        spr2->GetTextureRect(&tx2, &ty2, &tw2, &th2);

        DWORD* color1 = new DWORD[(x[2] - x[1]) * (y[2] - y[1])];
        DWORD* color2 = new DWORD[(x[2] - x[1]) * (y[2] - y[1])];
        DWORD* color;

        /** Copy the effectivearea of texture 1 */
        color = hge->Texture_Lock(hTex1, true);
        for(int i = 0; i < y[2] - y[1]; i++)
        {
            for(int j = 0; j < x[2] - x[1]; j++)
            { 
                color1[i * (x[2] - x[1]) + j] = color[((int)ty1 + sy1) * w1 + (int)tx1 + sx1 + i * w1 + j];
            } 
        } 
        hge->Texture_Unlock(hTex1);

        /** Copy the effectivearea of texture 2 */
        color = hge->Texture_Lock(hTex2, true);
        for(int i = 0; i < y[2] - y[1]; i++)
        {
            for(int j = 0; j < x[2] - x[1]; j++) 
            { 
                color2[i * (x[2] - x[1]) + j] = color[((int)ty2 + sy2) * w2 + (int)tx2 + sx2 + i * w1 + j]; 
            } 
        } 
        hge->Texture_Unlock(hTex2);

        /** Test for the collision */
        for(int i = 0; i < y[2] - y[1]; i++)
        {
            for(int j = 0; j < x[2] - x[1]; j++)
            {
                if(color1[i * (x[2] - x[1]) + j] != airColor && color2[i * (x[2] - x[1]) + j] != airColor)
                {
                    delete []color1;
                    delete []color2;

                    return true;
                }
            }
        }

        delete []color1;
        delete []color2;
        return false;
    }
    else return false;
}
```

[點擊下載](src.rar)
