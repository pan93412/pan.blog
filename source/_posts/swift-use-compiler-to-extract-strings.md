---
title: 善用 Swift 的字串擷取功能簡化 I18n 流程
date: 2022-08-27 19:47:25
updated: 2022-08-27 19:47:25
tags:
  - Swift
  - Compiler
  - Xcode
  - I18n
  - L10n
categories: Developments
robots:
---

## 撰文原由

在 [威注音 2.1.0 SP1 的更新日誌](https://github.com/vChewing/vChewing-macOS/releases/tag/2.1.0) 下看到了這麼一段話：

> Interface Builder 給每個標籤的命名都是隨機的，手動改起來又低效又容易改壞，對多語言本地化而言簡直是天災。

[事實上在 WWDC21 中](https://www.wwdcnotes.com/notes/wwdc21/10220/)，Apple 就為 Xcode 推出了一項可以解決這項痛點的新功能：「使用編譯器擷取字串」。與早期透過從 Swift 原始碼拉取可翻譯字串，經常漏掉一些字串的方式不同——這個功能會先編譯原始碼，然後從編譯出的檔案中判斷可以翻譯的字串。

## 啟用功能

這項功能在新專案是預設啟用的，但舊專案可以 opt-in：點開專案設定，進入 “Build Settings”，展開所有功能（All），尋找 “Swift” 然後找到 Localization 的 “Use Compiler to Extract Swift Strings”，將其設定為 Yes 即可。

![Opt in “Use Compiler to Extract Swift Strings”](opt-in-function.webp)

## 匯出字串

開啟之後的國際化與本地化方式，跟先前會不太一樣。以往需要手動改 strings 檔案，現在可以先到 “Product” > “Export Localization…” 匯出編譯器擷取出的所有字串：

![“Product” > “Export Localization…”](menu-export-localization.webp)

接下來指定匯出的路徑，然後等待 Swift 完成編譯並擷取字串。若是從舊專案 opt-in，擷取字串的過程中可能會拋出一些錯誤，這個時候就得修正（如果只是警告的話也可以忽略）：

![Exporting…](exporting.webp)

## 翻譯流程

接下來就可以進入選擇好的資料夾，使用 Xcode 點開對應語系的 xcloc 檔案：

![Open the xcloc file with Xcode](open-xcloc.webp)

如果不習慣 Xcode 的本地化工具，也可以打開特色選單 > 「顯示套件內容」，Localized Content 裡面就有通用的 xliff 格式以及一些其他檔案（比如 RTF）。xliff 檔案可以用 Poedit 開啟，也可以上傳到 Crowdin、Transifex、Weblate 等協作翻譯平台：

![Open the xliff file with Xcode](open-xliff-with-poedit.webp)

## 匯入字串

翻譯流程完成之後，可以到 “Product” > “Import Localization…” 匯入翻譯完成的 xcloc 檔案：

![Go to “Product” > “Import Localization…”](menu-import-localization.webp)

匯入時可能會跳出一些 lint 提示（警告或錯誤），這裡可以視情況修正。若是故意為之，亦可直接匯入：

![The lint result of XCLOC import](lint-xcloc.webp)

由於是基於編譯器擷取的結果，因此 strings 檔案也會跟著更新，往後就不需要自己維護 strings 檔案了：

![The diff before importing and after importing](git-staging-diff.webp)

## 結語

我並不是 Swift 開發者，這個功能是之前在對一些 macOS 軟體本地化時研究發現出來的。這個功能除了讓本地化翻譯員可以完整翻譯所有字串，也可以減少開發者維護 strings 檔案的成本。

希望這篇文章能夠幫助到 Swift 的 I18n 工程師以及翻譯者 :)
