title: 關於HGE的透明背景處理
date: 2011-9-13 12:12:32
tags: [ 老博客備份歸檔, C++, HGE ]
category: 老博客備份歸檔
---

　　嘛 = = 在做那個項目的動畫預覽器的時候，因爲那引擎封裝得太麻煩了，於是自己基於HGE再移植一遍，發現其中有一個SetTransparentColor函數，即設置透明色。

　　拿出來分享一下吧。

　　其實方法很簡單，`HTEXTURE` 是紋理句柄，當你用 `Texture_Lock` 這個函數鎖定這個紋理的時候，它的返回值就是這個紋理在內存中的首地址。也就是說接下來的 width * height 個地址中就是這個紋理的每一個像素了。既然要設置透明色，只要對於每個像素判斷一下與運算一下就好了。

```cpp
HTEXTURE SetTransColor(HTEXTURE hTex, DWORD dwColor)
{
    /** 注：上面的dwColor代表的是RGB，不是ARGB */
    static HGE* hge = hgeCreate(HGE_VERSION);

    int size = hge->Texture_GetWidth(hTex) * hge->Texture_GetHeight(hTex);
    DWORD* dwTex = hge->Texture_Lock(hTex);
    for(int i = 0; i < size; i++)
    {
        if((dwTex[i] & 0x00FFFFFF) == dwColor)
        {
            dwTex[i] &= 0x00FFFFFF;
        }
    }
    hge->Texture_Unlock(hTex);

    return hTex;
}
```

　　嘛，這樣一來，就透明瞭~
