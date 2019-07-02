---
description: '>-本书的目的就是解释什么是绿色线程。 我们会用一个代码实现示例，虽小但能工作 —— 用实现的绿色线程去执行代码。'
---

# 绿色线程是个啥教程

## cfsamson/book-green-threads-explained  [![](http://llever.com/translate.svg)](https://github.com/chinanf-boy/chinese-translate-list)

「 绿色线程是个啥教程 」

[中文](./readme.zh.md) \| [english](https://github.com/cfsamson/book-green-threads-explained)

### 校对 🀄️

| 翻译的原文 | 与日期 | 最新更新 | 更多 |
| :--- | :--- | :--- | :--- |
| [commit](https://github.com/cfsamson/book-green-threads-explained/tree/4599d94cf2b9ad3ed5cad45134e67b3a63ccf129) | ⏰ 2019-06-26 | ![](https://img.shields.io/github/last-commit/cfsamson/book-green-threads-explained.svg) | [中文翻译](https://github.com/chinanf-boy/chinese-translate-list) |

* [x] README.md
* [ ] [green-threads.md](src/green-threads.zh.md)
* [ ] [background-information.md](src/background-information.zh.md)
* [ ] [an-example-we-can-build-upon.md](src/an-example-we-can-build-upon.zh.md)
* [ ] [the-stack.md](src/the-stack.zh.md)
* [ ] [simple-but-unsafe-green-thread-implementation.md](src/simple-but-unsafe-green-thread-implementation.zh.md)
* [ ] [final-200-lines-of-code.md](src/final-200-lines-of-code.zh.md)
* [ ] [supporting-windows.md](src/supporting-windows.zh.md)

#### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

### 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

## 介绍

{% hint style="info" %}
`我们在这里用到的所有代码都[在 github 存储库](https://github.com/cfsamson/example-greenthreads)。它会有两个分支，`main`只包含代码，和`commented`分支的带有注释的代码，就是解释我们所做工作的注释。`
{% endhint %}

绿色线程，用户线程，goroutine\(go 协程\) 或纤维\(fibers\)，它们有许多名称，但为了简单起见，我将从现在开始，把它们统称为绿色线程。

在本文中，我想通过实现一个非常简单的示例来探索它们是如何工作的，我们用 200 行 Rust 代码，创建自己的绿色线程。我们将在整个过程中解释所有内容，因此我们主要关注的是，通过使用简单但有效的示例来理解它们，并了解它们的工作原理。

{% hint style="info" %}
`我们不会使用任何外部库或帮助程序，并将从头开始做所有事情，因此我们可以确保我们真正了解，发生的事情。`
{% endhint %}

### 这篇文章的目标群众？

这篇文章中，我们正在偷看兔子洞\(我们不知道洞里面，还有什么在等着我们\)，所以如果这听起来很可怕，那么这篇文章可能不适合你。回去吧，过得幸福。

如果你是一个好奇的人，想要了解事情的运作方式，请继续阅读。也许你已经听说过 Go 及其 goroutines，或 Ruby 或 Julia 中的等价物，你知道如何使用它们，但想知道它们是如何工作的 - 那么请继续阅读。

此外，如果：

* 您是 Rust 的新手，想要了解有关其功能（features）的更多信息。
* 您已经参与了 Rust 社区中，有关 async / await，Pin-API 以及我们需要生成器的原因的讨论。在这种情况下，我尝试将所有部分中和在一起。
* 如果您想学习 Rust 中，内联\(inline\)汇编的基础知识。
* 如果你只是好奇。

好的，请与我一起，让我们试图弄清楚需要了解它们的一切。

您不需要是 Rust 程序员，才能理解本文，但强烈建议您首先阅读一些基本语法。如果你想长时间跟踪或克隆存储库并使用代码，你应该得到 Rust 并学习到了基础知识。

{% hint style="info" %}
`[装载 Rust 宝藏，都放在了这里](https://www.rust-lang.org/tools/install)`
{% endhint %}

### 跟上

我在这里提供的所有代码都在一个文件中，并且没有依赖关系，这意味着您可以轻松地启动自己的项目，并按照您的要求进行操作（我建议你这样做）。你甚至可以在[Rust playground](https://play.rust-lang.org)运行大部分代码。只要记得编译器要为`nightly`版本。

### 便携性和问题

目前我遇到了一个问题：`asm!`宏在发布模式下无法编译。它似乎与我在内联宏中使用的`"=m"`约束有关。

**编辑 2019-06-21：** 我决定解决这个问题，并更改内联汇编，以在发布版本上编译和运行。

我已经在 OSX，Linux 和 Windows 上测试了代码。

### 免责声明 <a id="docs-internal-guid-12e6c217-7fff-3de7-4bee-4532b47ef574"></a>

我不打算在这里做一个完美的实现。我正在偷偷摸摸地深入了解本质，并想把这些内容，'铸'成文章，但现在扩展成了一本小书。其实这不是展示 Rust 最强优势 —— 它的安全保证的最佳方式，但它确实显示了 Rust 的一个有趣的用法，代码大多非常干净，易于跟踪。

但是，如果您发现我可以使代码更安全，而不会使其显着更复杂的地方，我欢迎您在[the repo](https://github.com/cfsamson/example-greenthreads)创建一个问题，甚至最好是一个拉动请求。

### 赞誉

[Quentin Carbonneaux](https://github.com/mpu)早在 2013 年，写了一篇[好的博文](https://c9x.me/articles/gthreads/intro.html)，作为我主要代码示例的灵感。谢谢[nickelpro](https://github.com/nickelpro)有关 Windows 支持的帮助和反馈。

### 编辑

2019-06-18：实现适当 Windows 支持的新章节

2019-06-21：相当大的改变和清理。据反馈，Valgrind 报告了一些代码问题并且崩溃了。现在已经修复，目前还没有未解决的问题。此外，代码现在，`debug`和`release`（模式）可在所有平台上构建与运行，都没有任何问题。感谢大家报告他们发现的问题。

2019-06-26： Windows 支持附录，把`XMM`字段视为 64 位，但它们是 128 位，这是我的疏忽。纠正这一点为该章增加了一些有趣的材料，但不好的一点是，也带来了一些复杂性。但是，它现在已有了纠正和说明。

