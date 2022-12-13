# 將Sprite sheet分割的方法

Unity的遊戲解包之後，常常會發現他們把所有動作圖片包成一張大圖。  
如果想將每張圖片分割出來，以人工方式擷取不但麻煩又不夠精確。  
這邊就筆記一下找到的方法，希望幫助到有同樣困擾的人。

## 安裝Unity

進入[Unity官網](https://unity.com/download)，下載Unity Hub。

執行並下載安裝其中一個Unity版本。

之後，新建立一個專案，範本選2D，因為要利用其Sprite Editor。

![img](https://i.imgur.com/vZcdG8C.png)

## 匯入圖片

直接將圖片拖曳到專案內，下方應該會看到這個檔案。

![img](https://i.imgur.com/kOJZr1r.png)

右邊會有圖片的設定，需要對以下欄位進行處理

- Sprite Mode: 改為Mutliple
- Mesh Type: 改為Full Rect
- Advanced > Read/Write: 勾選
- Max Size: 要改為匯入檔案的寬高，Unity的圖片應該都是要求方形且寬高為2的次方，我匯入的圖為4096\*4096，所以此處選4096

![](https://i.imgur.com/EgyR2qA.png)

設定完後，點選Apply。

## 安裝套件

點選Sprite Editor，如果你使用2D的範本，應該已經內含。否則會出現錯誤訊息。

![img](https://i.imgur.com/BxnuXkS.png)

選擇功能表的[Window]-[Package Manager]

![img](https://i.imgur.com/mbW9uNw.png)

左上方選擇Unity Registry，右上方搜尋"2D"，選擇右下方的Install

![img](https://i.imgur.com/r8PWQD1.png)

裝完後再點選圖片，選擇Sprite Editor，就會開啟視窗。

![img](https://i.imgur.com/cpndqMH.png)

## 分割圖片

選擇左上角的Slice（如果無法選擇，代表前面匯入圖片時Sprite Mode沒有設定為Multiple），Method改成Smart後按下Slice。

![img](https://i.imgur.com/dWW1zUi.png)

成功的話，所有項目會用白框圈起來。

如果前面的Max Size沒有設定正確，此處的圖片有可能被壓縮，導致分割不正確，所以還是要選對。

![img](https://i.imgur.com/9PRl74U.jpg)

選擇右上角的Apply回到原頁面。點選右邊的箭頭就會展開所有小圖。

![img](https://i.imgur.com/vnViI8C.png)

## 另存圖片

有點麻煩的是，Unity內建沒辦法把這些圖片直接另存。所以參考[這篇](https://stackoverflow.com/questions/55844965/can-you-export-sliced-sprites-png-images-in-unity)的回答。

複製以下程式碼，另存成`EditorSubSprites.cs`。

```csharp=
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class ExportSubSprites : Editor {

    [MenuItem("Assets/Export Sub-Sprites")]
    public static void DoExportSubSprites() {
        var folder = EditorUtility.OpenFolderPanel("Export subsprites into what folder?", "", "");
        foreach (var obj in Selection.objects) {
            var sprite = obj as Sprite;
            if (sprite == null) continue;
            var extracted = ExtractAndName(sprite);
            SaveSubSprite(extracted, folder);
        }

    }

    [MenuItem("Assets/Export Sub-Sprites", true)]
    private static bool CanExportSubSprites()
    {
        return Selection.activeObject is Sprite;
    }

    // Since a sprite may exist anywhere on a tex2d, this will crop out the sprite's claimed region and return a new, cropped, tex2d.
    private static Texture2D ExtractAndName(Sprite sprite) {
        var output = new Texture2D((int)sprite.rect.width, (int)sprite.rect.height);
        var r = sprite.textureRect;
        var pixels = sprite.texture.GetPixels((int)r.x, (int)r.y, (int)r.width, (int)r.height);
        output.SetPixels(pixels);
        output.Apply();
        output.name = sprite.texture.name + " " + sprite.name;
        return output;
    }

    private static void SaveSubSprite(Texture2D tex, string saveToDirectory) {
        if (!System.IO.Directory.Exists(saveToDirectory)) System.IO.Directory.CreateDirectory(saveToDirectory);
        System.IO.File.WriteAllBytes(System.IO.Path.Combine(saveToDirectory, tex.name + ".png"), tex.EncodeToPNG());
    }
}
```

將這個檔案放到`專案目錄/Assets/Editor`（如果無此目錄就新增）

![img](https://i.imgur.com/7PnBF6T.png)

將底下圖片的分割檔全選，按右鍵選擇[Export Sub-Sprites]。

![img](https://i.imgur.com/aGUp2OF.png)

選擇另存目錄，就會發現圖片都被匯出了。

![img](https://i.imgur.com/F6TYRpW.png)
