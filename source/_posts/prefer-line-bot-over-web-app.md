---
title: 為什麼選擇「壟斷的」LINE bot，而不是傳統的 Web apps？
date: 2022-09-13 09:02:00
updated: 2022-10-24 23:17:02
tags:
  - line
  - web
  - bot
  - app
  - technology
  - twitter
categories: The thoughts
robots:
---

> 原文載於 [Twitter](https://twitter.com/bystartw/status/1569492119722786818?s=61&t=UVzhpuQaGcgZkGpySz_yVQ) 和 [Facebook 文章底下的留言](https://www.facebook.com/denny0223/posts/pfbid0X9GmUVBXiBCxYXyqq5qRVjNbTXjrBHzBuPJyogttmQnL2EW2YZzD6NooUycbVTakl)。我當時是以技術面探討問題，不過原討論串還有討論到很多層面的問題，值得到 `Facebook` 原文，看看其他人的想法。

## Context

> 又是一個必須要有 LINE 才能參與的高雄活動
>
> 這樣的機制設計真的沒有問題？？

## TL;DR

個人覺得還真的沒有什麼問題：

1. LINE bot 的認證比 Web App 來得簡單
2. LINE bot 的 UX 不差
3. 開發者仍然有自己開發 UI 的自由度
4. 設計 LINE bot 通常比 Web App 省力
5. 台灣的 LINE 使用者足夠多

## 為何不用其他平台？

Telegram 的話也可以辦到，但相較之下使用量就少很多了；Messenger Bot 也是台灣使用的一個大宗，但之前碰過 FB 的審核機制……麻煩到讓人不是很想碰。

所以 LINE 確實是兼顧大部分使用者，而且成本最低的最優解了。至少這個活動還不是強制性的……對岸沒有 WeChat 幾乎什麼事情都做不了。（甚至包括出門）。

另外上面講的這四個平台（LINE、Telegram、Facebook 跟 WeChat）全部都有不自由的地方，感覺換到哪裡都半斤八兩啦 🌚

## 備註

- Discord 的話也不是很自由，而且使用廣泛度也略輸 Messenger 跟 LINE。Discord 目前更適合當一個社群場合經營，而它的 bot 目前還沒有這兩個來得直接。
- Matrix 的話是有無限可能，但光同步聊天記錄就慢的跟……一樣。而且使用量甚至贏不過 Telegram。
- Slack 和 Mattermost 的專業不是這個，而且在新工作空間都需要註冊一支新的帳號。後者的維護成本甚至大於一個小型的 Web App。
- Web Apps 除了開發的能力要求和成本都比較高之外，還有儲存使用者資料的資安問題——不專業的 Web Developer 可能就會把密碼存成明碼或純粹用 MD5 或 SHA-1 雜湊。
