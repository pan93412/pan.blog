---
title: 什麼時候該引入 Lockfile？
date: 2022-10-18 13:37:00
updated: 2022-10-24 22:19:37
tags:
  - lockfile
  - npm
  - cargo
  - twitter
categories: Developments
robots:
---

> [原文撰於 Twitter](https://twitter.com/bystartw/status/1582244519319597056?s=61&t=E09UfEWbDsXiuks4rs0Y3g)。

## 撰文初衷

![第一次看到沒有 commit package-lock.json 的 repo, posted by @as790726, on 2022/10/17.](twitter-original-post.png)

> 第一次看到沒有 commit package-lock.json 的 repo

## TL;DR

事實上這樣在 library 上沒有什麼問題。Lockfile 的追蹤有個小原則：

- 應用程式建議追蹤 lockfile：不追蹤，下次 npm install 就無法確定具體的依賴版本是什麼。
- 函式庫可以不用追蹤，因為使用者安裝套件時，套件管理器會根據依賴自動選取最適合的版本，而你自己的 lockfile **會被忽略**[^1]。不過建議追蹤，見下文。

[^1]: _The difference is that package-lock.json cannot be published, and it will be ignored if found in any place other than the root project._ <https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json#package-lockjson-vs-npm-shrinkwrapjson>

## 函式庫「該不該」追蹤 lockfile？

假如函式庫有用到諸如 ESLint 的工具，追蹤一下可以避免之後設定開發依賴的麻煩，
所以像 NPM 官方就是推薦 **無論如何都追蹤 lockfile**。

不過也有預設不推薦在 library 情境下追蹤 lock 的例子，比如 Rust 的 Cargo 套件管理器[^2]。不過 Cargo 的開發工具主要都是作為 submodule 安裝在系統中，通常不會跟著 repo 一起追蹤，所以不太適合放在一起比較。

不過要注意：**這時候的 lockfile 就不是追給下游應用程式看的**，主要是為了自己開發方便。

[^2]: <https://doc.rust-lang.org/cargo/faq.html#why-do-binaries-have-cargolock-in-version-control-but-not-libraries>

## 為什麼「應用程式」就該追蹤 lockfile？

`package.json` 通常不是描述固定的版本，而是一個版本區間：舉個例子：你可能在 `package.json` 裡面描述 `vue: "^2.4.0"`，但實際上 NPM 幫你選了 `2.7.13`。這個行為是可以預測的，可以參考 NPM 官方的 Semver 計算機：<https://semver.npmjs.com>。

看起來只有更動 minor version，會發生什麼事情嗎？我之前就有遇過一個 case：某個的 Vuepress 站台沒追蹤 lock，CI 完全靠 npm install 自己選版本。然後朋友發現本機產生出的 blog 樣式，和 CI 產生的 blog 樣式不一樣。後來才發現其實就是本機有 lock 記錄版本；CI 沒有 lock 所以就選了樣式有差異的最新版本。

當然，你也可以 _pin_ 固定的版本（`2.7.13`），可是你 pin 的依賴，可能也沒 pin 它的依賴…… 老實說用 lockfile 還是比較保險 XD

## 結論

所以 app 建議還是追蹤一下。而 library 的話由於 npm install 只認 root 的 package-lock.json，所以理論上追不追蹤都不影響使用者——只影響你自己的 DX 而已 XD
