---
title: vChewing 出了 2.4.0 SP2 了！
date: 2022-09-07 01:17:00
updated: 2022-10-24 23:30:20
tags:
    - vChewing
    - 威注音
    - twitter
categories: Recommendations
robots:
---

> src: <https://twitter.com/bystartw/status/1567200310430662656?s=61&t=UVzhpuQaGcgZkGpySz_yVQ>

vChewing 出了 2.4.0 SP2 了！🎉

自己覺得 vChewing 是除了官方注音之外，長得最好看（因為有 IMK 加持），而且功能跟選字功能也很棒的輸入法，真的很推薦想嘗試第三方輸入法的朋友試試看！（連結見引文）

![vChewing 在 2.4.0 SP2 推出時的 commit 提交情況](activity.png)
Figure 1: vChewing 提交很活躍！

## 資安疑慮

另外關於 vChewing (下稱 vC) 的資安或隱私疑慮：

* vC 在 2.3.0 之後引入 Apple 官方的沙盒機制，因此只要你沒有授予授權，輸入法本身是看不到你的資料夾的。
* 雖然我沒有完全閱讀 vC 的 src，但至少它的 Shift 是靠旁敲的方式偵測的，不是記錄所有鍵盤輸入
* vC 只有更新和網站相關有連到個人站台

## 稽核方式

如果有興趣稽核 vChewing 外連的情況，可以自己 clone 回來用 regex 查，或者是直接用這個第三方網站看：
[sourcegraph](https://sourcegraph.com/search?q=context:global+repo:vChewing/%28vChewing-macOS%7CTekkon%7CMegrez%7CHotenka%7Clibvchewing-data%29+%28http%7Chttps%7Cftp%7Cws%7Cwss%29%5C:&patternType=regexp)

** 如果還是信不過（認為作者會混淆連結）的話，也可以自己掛一個抓包軟體偵測 vC 的所有請求啦…… 雖然我是找不到除了更新以外的請求 code：[sourcegraph](https://sourcegraph.com/search?q=context:global+repo:vChewing/%28vChewing-macOS%7CTekkon%7CMegrez%7CHotenka%7Clibvchewing-data%29+%28%28en%7Cde%29%28crypt%7Ccode%29%28ion%7C%29%29%7C%28base64%7Cbase32%7Caes%7Copenssl%7CSymmetricKey%7Cstream%7CSession%7CConnection%7CRequest%7CResponse%29+count:%22all%22&patternType=regexp)
