---
title: VBA 實作：數字輸入器
tags: []
id: '258'
categories:
  - - uncategorized
---

有天，你心血來潮想要做一個 Excel 上的數字輸入器，但是不想要用突兀的 Form 表單。那該怎麼做呢？
<!-- more -->
## 思考規格

首先，沒有 Form 要怎麼做出類似按鈕的效果呢？我們先來想想一般使用者直覺會覺得哪個地方最像按鈕，而無庸置疑，就是儲存格。

然後，使用者會希望數字怎麼擺放與呈現呢？我們可以參考計算機。而一台計算機的數字鍵盤會有什麼呢？除了數字鍵盤及顯示面板外，就是「Backspace」和「清除」鍵。

接著，我們思考一下使用者會希望按下按鈕會發生什麼。最為直覺的是在按下的按鈕加上底色，同時數字也會呈現在顯示面板上。但為求簡單，所以我決定讓按下的按鈕變成斜體。

最後，我們就可以做出像這樣的東西：

![](https://blog.pan93.tk/wp-content/uploads/2020/10/image-4.png)

數字輸入器的介面參考

## 製作按鍵的回饋感

### 思考

首先，VBA 有什麼方法能夠知道我按了哪個儲存格呢？VBA 有個非常好用的功能，叫做「事件」。只要使用者做了什麼動作（觸發），Excel 都會通知你發生新事件，告訴你使用者做了什麼。

但是使用者又不是只會按儲存格，他們也可能會打字，會上下移動工作表，甚至 VBA 的事件精細到連打開試算表都要通知你一聲。我們要怎麼篩選這些事件呢？而且，事件到底是通知到哪裡呢？

首先，Excel 遇到新事件時，會先看看你有沒有在指定的地方放下動作（Sub，子程式），如果沒放就等同忽略，而有放就會觸發這個動作。所以我們只要知道「按下儲存格後該在哪裡接收事件」，就完成了。

### 事件的接收（監聽）

Excel 中，你按儲存格實際上是選取儲存格。而 VBA 中就有一個跟儲存格變更有關的事件，叫做 Worksheet\_SelectionChange，長得像這樣：

Private Sub Worksheet\_SelectionChange(ByVal Target As Range)

別急著複製。VBA 其實有提供一個非常簡單的事件選取工具。首先開啟 VBA 編輯器，找到你想要監聽的工作表，然後按一下上方的 \[(一般)\]，之後選擇 \[Worksheet\]。

![](https://blog.pan93.tk/wp-content/uploads/2020/10/image-5.png)

選擇 Worksheet

然後選擇 \[SelectionChange\]，搞定！

![](https://blog.pan93.tk/wp-content/uploads/2020/10/image-6.png)

選擇 SelectionChange

### 了解事件結構

首先，我們來看看產生的程式碼。

Private Sub Worksheet\_SelectionChange(ByVal Target As Range)

End Sub

你會發現到裡面出現了非常多奇怪的東西。什麼是 `Private` 和 `ByVal`？`Target` 是什麼？`As Range` 又是什麼東西？

首先，每個工作表都有自己獨立的事件，也就是你在 A 工作表選取東西，不干 B 工作表的事。如果用 `Public`，就代表 A 工作表的事件也會影響其他工作表。這不應該發生。而 `Private` 就可以限定這個事件是 A 工作表 only 的。

而 `ByVal` 則需要一點資料結構的知識。VBA 傳遞參數有兩種形式，一個是 `ByVal`，建立物件的副本，另一個則是 `ByRef`，傳遞指向原物件的參考。有興趣可以看看〈[VBA 中 ByVal 和 ByRef 的基础用法和区别](https://www.lanrenexcel.com/vba-byval-byref-basic/)〉。

`Target` 則是你選取的儲存格。`As Range` 代表這是一個儲存格範圍 (Range) 類型的物件。舉凡 `.Range()`、`.Cells()` 所回傳的物件都是 `Range`，只是平常不會去特別宣告而已。

### 建立按鍵回饋感

![](https://blog.pan93.tk/wp-content/uploads/2020/10/image-3.png)