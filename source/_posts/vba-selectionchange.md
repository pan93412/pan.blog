---
title: VBA 筆記：選取事件
tags:
  - vba
  - 事件
  - 選取
categories:
  - Developments
date: 2020-10-18 23:40:43
---

有天我突然想寫一個 Excel 按鈕，但是又不想要做一個 Form，而想到最好的辦法，就是按 Excel 中的格子。那該怎麼做，才可以做出這種偽按鈕呢？
<!-- more -->
## 原理思考

### Excel 中，按一下儲存格代表？

就是選取「單一」儲存格。就跟你拖曳選取一堆儲存格、選取整整一行或一列所代表的含意是相同的。

### 那怎麼知道使用者選取了什麼？

用 VBA 的事件 (Event)。只要使用者做了什麼動作（觸發），Excel 都會通知你發生新事件，告訴你使用者做了什麼。

首先，Excel 遇到新事件時，會先看看你有沒有在指定的地方放下動作（Sub，子程式），如果沒放就等同忽略，而有放就會觸發（Trigger）這個動作。所以我們只要知道「選取儲存格後該在哪裡接收事件」，就完成了。

## 正文

### 建立接收事件的子程式

VBA 中有一個跟儲存格變更有關的事件，叫做 `Worksheet_SelectionChange`，長得像這樣：

`Private Sub Worksheet\_SelectionChange(ByVal Target As Range)`

別急著複製。VBA 其實有提供一個非常簡單的事件選取工具。首先開啟 VBA 編輯器，找到你想要監聽的工作表，然後按一下上方的 \[(一般)\]，之後選擇 \[Worksheet\]。

![按一下 [專案] 中的其中一個工作表之後，在彈出來的畫面中按一下 [(一般)]，之後按下 [Worksheet]。](image-5.png)

選擇 Worksheet

然後選擇 \[SelectionChange\]，搞定！

![接著，找到旁邊的下拉框，選擇 [SelectionChange]。](image-6.png)

選擇 SelectionChange

### 了解子程式的結構

首先，我們來看看產生的程式碼。

```vb
Private Sub Worksheet_SelectionChange(ByVal Target As Range)

End Sub
```

你會發現到裡面出現了非常多奇怪的東西。什麼是 `Private` 和 `ByVal`？`Target` 是什麼？`As Range` 又是什麼東西？

首先，每個工作表都有自己獨立的事件，也就是你在 A 工作表選取東西，不干 B 工作表的事。如果用 `Public`，就代表 A 工作表的事件也會影響其他工作表。這不應該發生。而 `Private` 就可以限定這個事件是 A 工作表 only 的。

而 `ByVal` 則需要一點資料結構的知識。VBA 傳遞參數有兩種形式，一個是 `ByVal`，建立物件的副本，另一個則是 `ByRef`，傳遞指向原物件的參考。有興趣可以看看〈[VBA 中 ByVal 和 ByRef 的基础用法和区别](https://www.lanrenexcel.com/vba-byval-byref-basic/)〉。

`Target` 則是你選取的儲存格。`As Range` 代表這是一個儲存格範圍 (Range) 類型的物件。舉凡 `.Range()`、`.Cells()` 所回傳的物件都是 `Range`，只是平常不會去特別宣告而已。

### 使用上的注意事項

首先，選取任何儲存格都會觸發這個事件，所以記得判斷儲存格的內容。

此外，如果使用者拖曳選取，那我們就無法正確判斷該觸發的按紐。除了用 `foreach` 掃描所有使用者選取的儲存格外，你也可以用 `.Cells(1,1)` 強制指定我們要的儲存格是最左上角那個。

## 實作：開關按鈕

![三個儲存格，從左自右是「啟動渦輪加速系統」（位於 B3）、「ON」（位於 E3）、「OFF」（位於 F3）。](image-9.png)

我希望上圖範例中，按一下 ON 就讓 ON 變粗，按 OFF 就讓 OFF 變粗，那該怎麼做呢？首先，我們先在 Excel 製作如圖中的按鈕，然後開啟 VBA 編輯器並建立選取事件。

### 怎麼知道使用者按了哪顆按鈕？

我們可以從 `.Address` 取得。就像這樣：

```vb
Target.Cells(1, 1).Address
```

`.Address` 會回傳如 `$A$1` 的絕對座標。而我們可以發現範例中「ON」和「OFF」分別位於 E3 和 F3，所以我們就可以寫出如這樣的偽程式碼：

```plain
如果 使用者按的儲存格是 $E$3 就
  把 $E$3 變粗
  把 $F$3 還原
反之，如果 使用者按的儲存格是 $F$3 就
  把 $F$3 變粗
  把 $E$3 還原
```

那我們要怎麼寫出好看的「如果」敘述句，跟怎麼實作把字型變粗呢？

### 善用 Switch Case

像上面那個偽程式碼，我們就可以寫成這樣：

```vb
Select Case [使用者按的儲存格]
    Case "$E$3"
      '<按下 ON 時的動作>
    Case "$F$3"
      '<按下 OFF 時的動作>
End Select
```

### 得知使用者按的儲存格

首先〈了解子程式的結構〉這裡我們就提過 `Target` 是使用者選擇的儲存格，而為了要確保我們判斷的儲存格只有一個，防止莫名其妙的 bug，我們用 `.Cells(1,1)` 限定要判斷的儲存格是最左上角那個。

而跟〈怎麼知道使用者按了哪顆按鈕？〉介紹到的 `.Address` 結合，就長這樣：

```vb
Target.Cells(1, 1).Address
```

放進去〈善用 Switch Case〉的程式碼吧！

```vb
Select Case Target.Cells(1, 1).Address
    Case "$E$3"
      '<按下 ON 時的動作>
    Case "$F$3"
      '<按下 OFF 時的動作>
End Select
```

### 讓按鈕文字變粗

VBA 要讓字型變粗，會用到的屬性就是 `.Font.Bold`。把屬性設成 `True` 就變粗，`False` 就還原。以下程式碼可以讓 E3 儲存格的字型變粗。

```vb
Range("E3").Font.Bold = True
```

### 成品

```vb
Private Sub Worksheet_SelectionChange(ByVal Target As Range)
   Select Case Target.Cells(1, 1).Address
        Case "$E$3"
            Range("E3").Font.Bold = True
            Range("F3").Font.Bold = False
        Case "$F$3"
            Range("F3").Font.Bold = True
            Range("E3").Font.Bold = False
    End Select
End Sub
```

![三個儲存格，從左自右是「啟動渦輪加速系統」（位於 B3）、「ON」（位於 E3）、「OFF」（位於 F3）。而「OFF」按下去會變粗。](image-10.png)
