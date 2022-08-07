---
title: 文组也能懂的 Rust async 机制（简体版）
date: 2022-08-07 21:12:45
updated: 2022-08-07 21:21:00
tags:
    - Rust
    - async
    - easy
    - simple
categories: Development
---

> 考虑到其他语区的朋友，我使用 [繁化姬](http://zhconvert.org) 将这篇文章全文转换为简体（暂时不包含图档）。简体版是以 [commit 89f7c52](https://github.com/pan93412/pan.blog/tree/89f7c52c8d72915077808d2fe1db5dd12d622bc7) 为底进行转换，若未来有一些更新，亦会随之同步并在此更新。
>
> 希望这篇文章能帮到所有地方的 Asynchronous Rust 初学者 :) 如果未来有机会，也会考虑翻译成英文版本。

## 背景

<https://twitter.com/repsiace/status/1554103778994900992/>

![Rust async 到底是什么啊啊啊啊](20220807130414.webp)

> 修改一下：work stealing, thread-per-core, waker, mpsc, task queue 只有他们懂... 正常人不可能看懂 – [@twicemoemoe, 22-08-02](https://twitter.com/twicemoemoe/status/1554305217134735362)

作为一个文组，其实觉得 async 真的没有想象中的这么困难⋯⋯ 😂 或许搭配一些图片会好懂很多吧。

## TL;DR 不废话版本

- **Sync**（同步）：一件事情做完之后，再做下一件事情。
  - **blocking**（堵塞）：指“等一件事情”的行为。
- **Async**（异步）：一件事情还没完成，可以做其他不冲突的事情。
  - **concurrency**（并发、並行）：程序 **架构** 中，各个任务可以 **独立运行** 的特性。
    - **future**：Rust 中的一个异步任务的表示。
    - **polling**：不停地询问任务，确认事情是否已经完成。
    - **event-driven**：事情完成后，任务自己发通知表明完成。
  - **parallelism**（平行）：同时 **运行** 数个程序的行为。
    - **thread**（线程、執行緒）：系统进程（任务集）的基本单元。thread 通常是交由 CPU 内核运行。
      - **spawn**（生成）：指产生 thread 的行为。
      - **thread pool**（线程池）：将 thread 高效分配给每个任务的地方。
- **Async runtime**: 以 tokio 为例
  - **join** (macro)：并发运行 async 函数，并在全部完成后回传。
  - **select** (macro)：哪个 async 函数快，回传那个 async 函数的结果。
  - **main** (attribute macro)：在 main() 初始化 runtime。
  - **block_on**：在 sync 上运行 async 函数。
  - **spawn**：平行运行 async 函数。
  - **spawn_blocking**：在异步函数里面，为一个高耗时且同步 (blocking) 的函数另辟新线程 (thread)。

## 同步 (Synchronous) 跟异步 (Asynchoronous)

“**同步**”就是整个程序等一件事情完成（**blocking**，堵塞）。“**异步**”则是一件事情还没完成，可以做其他不冲突的事情。

就以早餐来说，“同步”就像是等吐司烤完之后，才去准备花生酱（**每件任务循序渐进地运行**）；“异步”就像是吐司正在烤的时候，就先准备花生酱（**每件任务并发地运行**）。

![同步和异步的图解。](async-sync.webp)

## 并发 (Concurrency) 与平行 (Parallelism)

刚才有提到“吐司正在烤的时候，就先准备花生酱”是个 **并发** 行为。

**并发** 就是指程序架构，任务可以互不相关运行的特性。我们可以称“吐司正在烤”和“准备花生酱”是个独立任务，而我们可以 **并发** 地进行这些任务。

说到并发，那什么是 **平行** 呢？

假如你妈也想帮你一起做早餐，而你负责“烤土司、准备酱料”，而你妈负责“煎蛋、摆盘”。你做自己的任务、而你妈做自己的任务——这就是 **平行**。

![并发搭配平行。](parallelism.webp)

回到电脑的例子上。我们可以把“你”和“你妈”比拟为 **CPU 内核 (core)** ，分配给你和你妈的一大堆独立任务叫做 **线程 (thread)** 。**并发** 就是工作单元自己用异步的方式处理任务；**平行** 就是分配其他工作单元处理任务。

### 進階：线程池 (thread pool)

你电脑的内核是有限的。就以 Apple M1 这颗 CPU 来说，最多也就只有 8 个内核。那要怎么高效的把一大堆线程，都分配到这些 CPU 上呢？

我们通常把这个“分配”叫做调度 (scheduling，排程）。首先，最简单的做法就是自己跟系统开线程（通常我们把这种线程叫做 **OS thread**）：

![OS Thread](os-thread.webp)

在 Rust 里面，使用 OS thread 是非常简单的：

```rs
let handle = std::thread::spawn(|| {
  /* 你的同步作业 */
});
```

这样的好处是不需要在程序里面包一个调度器，但缺点是线程的 **spawn** 生成（把任务分配给家人）和 **destroy** 销毁（告诉家人不用继续运行任务）都得找你的操作系统（管家）操作。那如果我们可以自己调度，是不是就能减少找操作系统（管家）的开销，甚至是做到更多更高效的事情（比如在一个线程里面运行数个并发任务）？

首先，我们需要先跟管家说“我需要 N 个可以分配家人任务的线程，”然后把这些线程都放进去我们的 `thread pool`。接下来程序需要 thread 运行任务，就请 `thread pool` 分配。而我们从 `thread pool` 分配到的线程，就是 **green thread**。

用文字描述太混乱，直接用图解吧：

![Green threads](green-thread.webp)

但假如每个 threads 里面都是会堵住程序的任务 (blocking)，那 Thread Pool 里面的 OS Thread 就得等这些任务完成，最后反而没有达到预期的高效分配。因此，如果 threads 里面是完全 sync 的任务，就 **没有必要用上 thread pool**，让 OS 分配即可。

![Thread pool in sync](thread-pool-sync.webp)

反之，如果 threads 里面的任务可以并发，因为一个 thread 可以并发运行多个任务，thread pool 的安排就会显得比 OS threads 高效许多：

![Thread pool in async](thread-pool-async.webp)

## 进阶阅读：Future 与 Executor

> 对底层比较没兴趣？可以先看 [Async 函数、区块和 await](#async-函数区块和-await)。

基础理论结束，拉回“任务”和 Rust 本身。你或许会很好奇“怎么让 Rust 知道一个任务是否完成？”

这里就要提到 Rust 的 Future 了。Rust 的 **Future** 就是一个可并发任务的抽象表示。而 **Executor** 就是负责轮询 (**polling**) Futures 的程序。把我们之前的“异步”图解用 Executor 的视角描绘出来：

![Executor 内部基本上怎么判断的](./how-executors-work.webp)

用文字表达：

1. Executor 会从 Future Receiver 抓出一个 Future。
2. Executor 运行这个任务的 `poll()`，并观察其回应。
3. 如果是 `Poll::Pending`（处理中），就继续处理下个任务。
4. 如果是 `Poll::Ready(T)`（已完成，回传值是 T），则将回传值交给需求方。

但这个文字流程有个问题：`poll()` 只会运行一次吗？如果会运行数次，那 `poll()` 下次会在什么时候运行？这里就得提到 Rust 的 **waker** 机制了。每一次 `poll()`，Executor 都会给这个任务一个 `context`。里面有一个 `waker`，可以用来提醒 Executor“可以运行 `poll()` 了。”

> 如果对这方面有兴趣的话，可以参考 <https://huangjj27.github.io/async-book/02_execution/03_wakeups.html>。另外 Future 还有很多很多的知识点（`Pin` 等等），碍于篇幅就先搁置。

### 延伸阅读：Polling 等各种方式的优劣

Polling（轮询），用程序写出来是像这样的：

```rs
loop {
  let status = task.poll();

  if let Polling::Ready(value) = status {
    return value;
  }

  continue;
}
```

这样有什么问题？首先异步任务通常要一段时间才会完成，一直 `poll()` 不会加快运行速度。如果真这样写，会浪费很多 CPU 时间在没必要的 `poll()` 上。另外，`loop {}` 是个 *blocking* 同步函数，这样子写，下一个 `Future` 是运行不了的。

那换另一种方式呢？

```rs
let (tx, rx) = std::sync::mpsc::channel();
let mut return_values = vec![];

for task in rx {
  let status = task.poll();

  if let Polling::Ready(value) = status {
    // 保存回传值，不再排程。
    return_values.push(value);
  } else {
    // 继续排程。
    tx.send(task);
  }
}
```

其实这就相当于在 `poll()` 阶段回传 `Poll::Pending` 前调用 `wake_by_ref()`——这个方法解决了 `loop {}` 的问题，但还是没能解决“没必要 `poll()` 的问题。”

倘若如果我们可以等到作业完成，再运行 `wake()` 呢？要这么做，我们就得先知道“工作什么时候才完成？”如果任务是用 callback 或 event 告知任务状态的，那就是在收到 event、或 callback 触发进行调用。

> 延伸阅读：<https://huangjj27.github.io/async-book/02_execution/05_io.html>

## async 函数、区块和 await

上面介绍了很多 Rust 中 `Future` 与 `Executor` 的理论基础，但实务上没有这么麻烦。事实上在 Rust 中，用 async 函数和 block 是非常直觉的。

就以上面的 [早餐例子](#同步-Synchronous-跟异步-Asynchoronous) 来说，我们可以把它先改写成这样：

```rs
async fn make_breakfast() -> Toast {
  let toast = bake_toast().await;
  let butter = prepare_peanut_butter().await;

  toast.apply(butter);
  toast
}
```

你或许会很疑惑他跟下面这个版本有什么差异：

```rs
fn make_breakfast() -> Toast {
  let toast = bake_toast();
  let butter = prepare_peanut_butter();

  toast.apply(butter);
  toast
}
```

首先，虽然整体上“做早餐”还是循序运行的（先烤完吐司、才准备花生酱），但做早餐这件事情因为已经是异步的了，所以你可以在做早餐的时候做其他事情：

![Sync in Async](./sync-in-async.webp)

后者的话就是纯 Sync，明显是比前者低效的：

![纯 Sync](./pure-sync.webp)

对照图片，或许你发现到 `.await` 刚好就是“任务切换点”。`.await` 之后，你可以去做其他事情（而不是空等）。等到烤箱声音响了之后 ([`wake()`](#进阶阅读future-与-executor)) 之后再回来做剩下的事情。所以整体上 async 函数是比较高效的，但我们要怎么让整个任务更高效呢（在 async 里面一次性运行更多任务？）

### 在 async 函数里面并发运行数个任务 (futures)

刚才提到我们希望在一个 async 里面一次性运行数个任务。这里我们可以借助 `tokio` 的 [`join!()`](https://docs.rs/tokio/latest/tokio/macro.join.html) 工具宏，表示“我希望这两个任务同时操作”，就像是把这两个任务融合为一了：

```rs
async fn make_breakfast() -> Toast {
  let (toast, butter) = tokio::join!(
    // 要注意这里不需要 .await，
    // await 的事情 `join!()` 会处理。
    bake_toast(),
    prepare_peanut_butter()
  );

  toast.apply(butter);
  toast
}
```

这样 `make_breakfast` 里面就是高效的模式了：

![Async in async](./async-in-async.webp)

那换一种现实中也常遇到的例子：你希望早餐可以在小孩上学前做完，如果没做完就不要继续做了。所以我们想要设置一个计时器，如果计时到了还没做完就直接取消；反之就继续做：

![Async with timeout](./async-with-timeout.webp)

这种情况就可以用 [`tokio::select!()`](https://docs.rs/tokio/latest/tokio/macro.select.html)——同时等“做早餐”跟“计时器”，回传完成速度最快的任务（分支），而取消剩下没做完的任务（分支）：

```rs
// Option 包含“有”或“没有”两种可能。如果计时器到了，
// 吐司还没完成，那就没有早餐；反之，就有早餐。
async fn make_breakfast_with_timer() -> Option<Toast> {
  tokio::select! {
    // 如果早餐先完成，那就有早餐。
    toast = make_breakfast() => Some(toast),

    // 如果时间先到，那就没早餐。
    _ = timer() => None,
  }
}

/// 一个设置在 30 分钟的计时器
async fn timer() {
  tokio::time::sleep(
    std::time::Duration::from_secs(30 /* min */ * 60 /* sec */)
  ).await
}

async fn make_breakfast() -> Toast {
  let (toast, butter) = tokio::join!(
    bake_toast(),
    prepare_peanut_butter()
  );

  toast.apply(butter);
  toast
}
```

如果你对这方面很有兴趣，可以看看 <https://huangjj27.github.io/async-book/06_multiple_futures/01_chapter.html>。

### 延伸阅读：await 只能在 async function 里面运行

要注意的，是 `.await` 只能在 async block 或 async function 里面使用。也就是说，你不能在同步函数（包括 `main()`）里面调用异步函数：

```rs
fn main() {
  // 会编译错误！
  let file_content = make_breakfast().await;

  // 还是不行 😄
  let file_content = async {
    make_breakfast().await
  }.await; /* async block 也需要 await */
}
```

既然每个调用者都必须是 async 的，**那是谁调用第一个 async 函数呢？** [这就得提到 Async Runtime 了。](#rust-的-async-runtime)

### 进阶阅读：从 Future 看 async 和 await

但这里面的 async 和 await 分别代表什么意义呢？`async fn` 其实展开来看，就是一个回传 Future 的函数：

```rs
struct ReadFileFuture { ... }

impl Future for ReadFileFuture {
  type Output = String;

  fn poll(...) { ... }
}

fn read_file(path: &Path) -> ReadFileFuture {}
```

而 await 的大致意思就是“没完成就说整个函数没完成；完成就继续”：

```rs
// 把这个函数的 context 转交给 read_to_string
let content_status = tokio::fs::read_to_string(path).poll(cx);

let content = match content_status {
  // 如果这个 feature 没完成，就剩下的 async 也就无法继续。
  Poll::Pending => return Poll::Pending,

  // 反之，把值拿回来
  Poll::Ready(c) => c,
}
```

实际上这部分还有许多地方需要考虑：包括要怎么在下次调用 `poll()` 的时候，知道现在要继续运行哪个 `Future`。更详细的资讯可以参考 <https://huangjj27.github.io/async-book/02_execution/02_future.html>。

## Rust 的 async runtime

实务上你不需要自己写一个 executor，**而是使用现成的 async runtime（运行阶段、运行时）**。一个 **async runtime** 除了 executor 之外，还有提供很多功能（比如上文提及的 thread pool、工具宏和函数，以及文件读写、channel 等等常用功能的异步对应方法）。

常见的 async runtime 有 `tokio`、`async-std` 和 `smol`，其中又以 `tokio` 和 `async-std` 为大宗。下文会以 `tokio` 作为介绍示例。

### 让 `main()` 变成 async 函数的起源地

我们在[〈延伸阅读：await 只能在 async function 里面运行〉](#延伸阅读await-只能在-async-function-里面运行)里面有提及“所有调用者必须都是 *async function*，”那 `main()` 呢？

还记得上文有提到“Futures 需要一个 *executor* 调度？”那 `main()` 原则上就是配置 runtime，让 runtime 准备 executor 的地方：

```rs
fn main() {
  // 设置多线程的 tokio runtime。
  tokio::runtime::Builder::new_multi_thread()
    // 激活所有功能。
    .enable_all()
    // 建构 runtime。
    .build()
    // 如果 runtime 建构失败就停住整个程序。
    .unwrap()
    // async 函数的起源地——堵塞 (blocking)，
    // 让整个 main() 等待这个 async 函数完成。
    .block_on(async {
        println!("Hello world");
    })

  // 然后程序就可以结束了。
}
```

不过其实不用这么麻烦：[`tokio::main!`](https://docs.rs/tokio/latest/tokio/attr.main.html) 这个 **属性宏 (attribute macro)** 就能帮你写好这方面的初始化了。你只要这样就好：

```rs
#[tokio::main]
async fn main() {
  println!("Hello, World!");
}
```

### 进阶阅读：让一个任务 (Future) 变成一个绿色线程 (Green Thread)——`spawn`

还记得併發 (Concurrent) 跟平行 (Parallelism) 吗？虽然大部分的情况下，在 **单线程**“併發”就已经很足够快了。倘若这个任务耗时很长，你希望开另一条线程“平行”专门处理这个任务，那就可以用 `spawn`：

```rs
let handle = tokio::task::spawn(async {
  /* 现在这里面的东西，都在独立的 thread 里面跑了！ */ 
});
```

[`tokio::task::spawn`](https://docs.rs/tokio/latest/tokio/task/fn.spawn.html) 虽然用起来很像建立 OS thread 的 `std::thread::spawn`，但 **spawn 里面不要放高耗时的同步函数**——除非你乐见整个 runtime 被卡在一件任务上面（或者是直接把 runtime 搞死，直接 panic！）

那要怎么在异步函数里面，开另一个 thread 跑同步函数呢？你可以用接下来会提到的 `tokio::task::spawn_blocking`。

### 在异步函数里面调用高耗时同步函数——`spawn_blocking`

除了开一个 `std::thread::spawn` OS thread 跑这种函数之外，你也可以用
[`tokio::task::spawn_blocking`](https://docs.rs/tokio/latest/tokio/task/fn.spawn_blocking.html) 开一个 **可以 await** 的同步 *blocking* 堵塞函数。

```rs
let _this_returns_42 = tokio::task::spawn_blocking(|| {
  for i in 0..114514 {
    for j in 0..1919810 {}
  }

  42
}).await;
```

这样子跑高耗时的函数之时，照样可以运行其他不用堵塞的任务。同理，你也可以把这个套进去 `join!` 并发完成，可是 **这样建立出的 thread 是取消不了的——不只是单纯的 `select!`，还包含 `.abort()`** 。因此还是尽量选择并善用异步函数。想深入了解 sync 函数与 async 函数桥接的资讯，可以参考 <https://tokio.rs/tokio/topics/bridging>。

## 结语

基本上把这则 Twitter 推文想要知道的大部分知识点都讲了。碍于个人能力不足，或许讲得不够清晰或不甚明确，也十分欢迎各路专家指正 QQ

另外这篇文章花了我 7hr 来写，如果觉得有用的话，欢迎把这篇文章分享给更多对 async 以及 Rust 异步程序有兴趣的人 owo 谢谢 🙏

![7hr!!!](./how-long-i-wasted.webp)  

另外也可以 follow 我的 [GitHub](http://github.com/pan93412) 支持我的 OSS 工作 ouo

## 参考文献

- <https://medium.com/erens-tech-book/%E7%90%86%E8%A7%A3-process-thread-94a40721b492>
- <https://hackmd.io/@sysprog/concurrency/https%3A%2F%2Fhackmd.io%2F%40sysprog%2FS1AMIFt0D#Concurrency-%E4%B8%A6%E8%A1%8C-vs-Parallelism-%E5%B9%B3%E8%A1%8C>
- <https://weihanglo.tw/async-book/02_execution/02_future.html>
- <https://en.wikipedia.org/wiki/Scheduling_(computing)>
- <https://zh.wikipedia.org/wiki/绿色线程>
- <https://doc.rust-lang.org/std/future/trait.Future.html>
- and more, a lot. Thanks to these authors ❤️
