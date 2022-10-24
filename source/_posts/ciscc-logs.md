---
title: CISCC 抱怨系統的開發日誌與心得
tags: []
categories: Developments
date: 2021-01-20 21:01:23
updated: 2021-01-20 21:01:23
robots: noindex
---

CISCC 是個用在 CISC（中學資訊討論群）的抱怨 (Complain) 系統。在我 MacBook Air 歸還前，我決定把我的開發紀錄都放在我的 Blog 上。

<!-- more -->

## 開發動機

![CISC, #行政人員討論區 (2021/1/2)](截圖-2021-01-03-上午1.38.54.webp)

CISC, #行政人員討論區 (2021/1/2)

其實會想開發 CISCC，主要是因為我在 CISC 偶爾會想靠北某些事情，但是我自己靠北不想要暴露自己的身份，也不想要辦一個假的 Discord 帳戶搞這種事情，再加上我最近都沒有可以上傳到學習歷程的專案，所以就著手開始撰寫這個專案。

## 結構圖

![用來描述 2020/1/3 時，ciscc 架構多亂的方塊圖 (2020/1/3)](image.webp)

用來描述 2020/1/3 時，ciscc 架構多亂的方塊圖 (2020/1/3)

![但實際產出的 CISCC 架構沒有這麼亂啦 (2020/1/20)](ciscc-arch.webp)

但實際產出的 CISCC 架構沒有這麼亂啦 (2020/1/20)

我預計寫三個元件：

- `ciscc-discord`: ciscc 跟 Discord 相關的地方
- `ciscc-api`: ciscc 的 API，與 `ciscc-discord` 和 `ciscc-frontend` 串接的橋樑
- `ciscc-frontend`: ciscc 的網頁前端，也就是發文的地方啦！
- `ciscc-docker`: 用 Docker 將 ciscc 這三個元件容器化並整合。

而截至 2020/1/3，完成度最高的是 `ciscc-discord`，其次是 `ciscc-api`，`ciscc-frontend` 尚未動工。2020/1/8 時已全數完工，並穩定上線。

## GitHub 貢獻紀錄

{% image 截圖-2021-01-03-上午1.44.48.webp width:1024px %}

GitHub 上 pan93412 在 2021/1 的貢獻紀錄 (2021/1/3)

{% image image-6.webp width:1024px %}

pan93412/ciscc-frontend 的截圖，截至當時仍有 1 個問題尚未解決，如圖。(2020/1/20)

{% image image-3.webp width:1024px %}

pan93412/ciscc-discord 的截圖，截至當時仍有 1 個問題尚未解決，如圖。(2020/1/20)

{% image image-1.webp width:1024px %}

pan93412/ciscc-api 的截圖，截至當時仍有 1 個問題尚未解決，如圖。(2020/1/20)

{% image image-2.webp width:1024px %}

pan93412/ciscc-docker 的截圖。(2020/1/20)

## 目前使用的 Tech Stack

- `ciscc-discord`: `discord.js` w/ `TypeScript`
- `ciscc-api`: `Nest.js` w/ `TypeScript`
- `ciscc-frontend`: Native JavaScript w/ `Webpack` w/ `TypeScript`

## Notes / Progress Report

### 2021/1/14 ~ 2021/1/16

- [睽違 9 天後，ciscc-discord 終於更新了！](https://github.com/pan93412/ciscc-discord/compare/952ece53564e0bef9c78e5d5586533b27f753a28%5E...master)😆 不過這次不是我的功勞，是 [ChAoSUnItY](https://github.com/pan93412/ciscc-discord/commits?author=ChAoSUnItY) 大大的！他幫我修正了幾個陳年 bug，就比如說可以利用 ciscc 平台亂標人，還有時區錯誤的問題。我也順便把幾個註釋上的問題更新了。
- 但是我還沒有更新 VPS 上的版本，抽空更新。

### 2021/1/8

- 這個時候我已經拿到 MacBook Pro 了。可以繼續開發了！
- [ciscc-api 把先前幾個 bugs，例如 CORS 問題、預先定義頻道 ID、IP 不正確問題、Rate Limit 太短等問題都修正後，彙整成一個 version 後釋出。也更新了 VPS 的上線版本。](https://github.com/pan93412/ciscc-api/compare/c825b4854da49ed4cb78333b777d4044efb596ab%5E...master)
- [ciscc-frontend 也是，但修正幅度比較不大。主要就是發文時鎖定「送出」按鈕，以及使錯誤訊息更友善。](https://github.com/pan93412/ciscc-frontend/compare/e23f86382dbd7937044ebeb2174127be7532a23d...master)

{% image 截圖-2021-01-20-下午8.38.26.webp width:1024px %}

ciscc-api: 截至 2020/1/8 的新增提交 (2020/1/20)

{% image 截圖-2021-01-20-下午8.40.01.webp width:1024px %}

ciscc-frontend: 截至 2020/1/8 的新增提交 (2020/1/20)

### 2020/1/5

{% image 截圖-2021-01-20-下午8.26.05.webp width:1024px %}

Discord 1/5 剛上線時的首幾個訊息 (2020/1/20)

- CISCC 移轉到我的 VPS 了！超棒的，運作幾乎正常。但 CSRF 還是有點問題，Safari 偶爾會卡住不讓發文。
- ciscc-api 已經設定好 Rate Limit，部分解決短時間大量垃圾訊息問題。此外也解決了 CSRF 的問題，CSRF 是跨站保護，雖原意是防止未認可網站存取內部 API，但同時，如果未正確設定 CSRF，會導致正常的瀏覽器無法使用這個 API。我的做法是直接放行所有網站，未來考慮利用設定檔的形式限定可存取這個 API 的網站。

### 2020/1/4

- ciscc-frontend、ciscc-api 和 ciscc-docker 完工，準備明天上線。

### 2020/1/3

- ciscc-discord 的部分，已經能正常發文，訊息也很正確！理論上已經可以穩定使用了。
- ciscc-api 的部分，也是差不多完成了。但是我之後可能需要搭配 token 機制，防止被濫用，當作群發垃圾訊息的地方。傳統的做法是 CSRF Token，但是 ciscc-frontend 是純前端，不能直接注入 CSRF Token，如果是透過 API 拿 CSRF，就完全失去 CSRF 的意義，變得多此一舉又沒有防禦效果。這部分需要再多加思考。

### 2020/1/2

- ciscc-discord 完工。

{% image 截圖-2021-01-02-上午1.57.03.webp width:1024px %}

ciscc-discord 命令清單文件的擷圖 (2020/1/2)

{% image 截圖-2021-01-02-上午2.37.21.webp width:1024px %}

ciscc-discord 可以正常設定預設發言平台了 (2020/1/2)

{% image 截圖-2021-01-02-上午2.37.24.webp width:1024px %}

ciscc-discord 設定預設發言平台後的終端機畫面 (2020/1/2)

{% image 截圖-2021-01-02-下午8.51.11.webp width:1024px %}

ciscc-api 可以用 POST 傳送訊息了 (2020/1/2)

{% image 截圖-2021-01-02-下午8.57.18.webp width:1024px %}

ciscc-api 加入了 Rate Limit (2020/1/2)
