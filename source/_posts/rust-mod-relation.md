---
title: Rust 的 crate/super/self 關係
date: 2021-10-13 11:06:21
updated: 2021-10-13 11:12:00
tags:
  - Rust
  - crate
  - super
  - self
  - programming
categories:
  - Developments
---

![`rust_hello_world` 的架構圖，與 crate、super、self 的關係](crate-relation.png)

假設 `rust_hello_world` 的目錄架構長這樣：（範例源自於我手邊的某個 production 專案）

- rust_hello_world
  - Cargo.toml
  - src
    - cli
      - opt.rs
    - lib.rs
    - cli.rs
    - logger.rs

「crate」概括來說就是一個專案，用 `Cargo.toml` 區分。每個 `mod` 都是一個層級，`super` 就是上個層級，`self` 就是本層級。上面的圖已經把 crate / super / self 的對應關係寫得很清楚了，以下寫範例：

- `cli::opt::Opt` 想要讀取 `LoggingLevel`，路徑可以這樣走：
  - `super::super::logger::LoggerLevel`
  - `crate::logger::LoggerLevel`
- `cli::opt::Opt` 想要讀取 `PROG_NAME`，路徑可以這樣走：
  - `super::PROG_NAME`
  - `crate::cli::PROG_NAME`

事實上 Rust 的模組關係也沒這麼複雜。把上面的例子變成目錄：

- [crate] /
  - [mod] cli/
    - [mod] opt/
      - [struct] Opt.txt
    - [const &str] PROG_NAME.txt
  - [[pub] mod] logger/
    - [[pub] enum] LoggingLevel.txt

`super` 等價於 `..`，self 等價於 `.`，而 crate 則類似 `/`。
