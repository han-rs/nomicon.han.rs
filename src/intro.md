# Rust 死灵书 (The Rustonomicon)

> **译者前言**
>
> 本书翻译大量参考自 [nomicon-zh-Hans](https://github.com/rust-lang-cn/nomicon-zh-Hans), 这里提供双语对照版. 译者补充的翻译内容后续会贡献到上游. 在此先感谢原仓库所有译者的贡献!
>
> 相关工作将于近日展开, 有需要的可先行到 [https://nomicon.purewhite.io/](https://nomicon.purewhite.io/) 查阅.
>
> 本翻译项目更新情况如下:
>
> ![Last updated](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2Fhan-rs%2Freference.han.rs%2Flatest-commit%2Fmaster&query=%24.date&label=Last%20updated)
>
> ![Status](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2Fhan-rs%2Freference.han.rs%2Fbranch-infobar%2Fmaster&query=%24.refComparison.behind&prefix=The%20base%20commit%20is%20&suffix=%20commit(s)%20behind%20the%20official%20one&label=Status)

<div class="warning">

Warning:
This book is incomplete.
Documenting everything and rewriting outdated parts take a while.
See the [issue tracker] to check what's missing/outdated, and if there are any mistakes or ideas that haven't been reported, feel free to open a new issue there.

警告:
本书尚未完成.
记录所有内容并重写过时的部分需要时间.
请查看 [issue tracker] 了解缺失/过时的内容, 若发现任何未报告的错误或想法, 欢迎在那里提交新 Issue.

</div>

[issue tracker]: https://github.com/rust-lang/nomicon/issues

## The Dark Arts of Unsafe Rust | 不安全 Rust 的黑魔法

> THE KNOWLEDGE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF UNLEASHING INDESCRIBABLE HORRORS THAT SHATTER YOUR PSYCHE AND SET YOUR MIND ADRIFT IN THE UNKNOWABLY INFINITE COSMOS.
>
> 这些知识是 "按原样" 提供的, 没有任何形式的明示或暗示的保证, 包括但不限于释放难以描述的恐怖, 粉碎你的心灵, 让你的思想漂流在不可知的无限宇宙中.

The Rustonomicon digs into all the awful details that you need to understand when writing Unsafe Rust programs.

Rust 死灵书 (The Rustonomicon) 挖掘了你在编写不安全 Rust 程序时需要了解的所有可怕的细节.

Should you wish a long and happy career of writing Rust programs, you should turn back now and forget you ever saw this book.
It is not necessary.
However if you intend to write unsafe code — or just want to dig into the guts of the language — this book contains lots of useful information.

如果你希望在编写 Rust 程序的过程中获得长久而快乐的职业生涯, 你应该现在回头, 忘记你曾经看过这本书.
它没有必要.
然而, 如果你打算编写不安全代码, 或者只是想深入了解语言的内涵, 这本书包含了很多有用的信息.

Unlike *[The Rust Programming Language][trpl]*, we will be assuming considerable prior knowledge.
In particular, you should be comfortable with basic systems programming and Rust.
If you don't feel comfortable with these topics, you should consider reading [The Book][trpl] first.
That said, we won't assume you have read it, and we will take care to occasionally give a refresher on the basics where appropriate.
You can skip straight to this book if you want; just know that we won't be explaining everything from the ground up.

与[《Rust 程序设计语言》][trpl]不同的是, 我们将假设你已掌握相当多的前置知识.
特别是, 你应该对基本的系统编程和 Rust 非常熟悉.  如果你对这些主题感到困惑, 你应该考虑先阅读《Rust 程序设计语言》.
总的来说, 我们并不会假定你已经读过《Rust 程序设计语言》, 我们仍然会注意偶尔在适当的时候对基础知识进行复习. 如果你愿意, 你可以直接跳过它就来看这本书: 但你需要知道, 我们不会从头到尾地详细解释一切.

This book exists primarily as a high-level companion to [The Reference][ref].
Where The Reference exists to detail the syntax and semantics of every part of the language, The Rustonomicon exists to describe how to use those pieces together, and the issues that you will have in doing so.

本书主要作为[《Rust 语言参考》][ref]的高级配套读物.
《Rust 语言参考》的存在是为了详细说明语言的每一部分的语法和语义, 而本书的存在是为了描述如何将这些部分结合起来使用, 以及你在这样做时将会遇到的问题.

The Reference will tell you the syntax and semantics of references, destructors, and unwinding, but it won't tell you how combining them can lead to exception-safety issues, or how to deal with those issues.

《Rust 语言参考》会告诉你引用、析构器和 unwind 的语法和语义, 但它不会告诉你如何将它们结合起来导致异常安全问题, 或如何处理这些问题.

It should be noted that we haven't synced The Rustnomicon and The Reference well, so they may have duplicate content.
In general, if the two documents disagree, The Reference should be assumed to be correct (it isn't yet considered normative, it's just better maintained).

需要注意的是, 我们没有很好地同步本书和《Rust 语言参考》的内容, 所以它们可能有重复的内容. 一般来说, 如果这两个文档有分歧, 应该认为《Rust 语言参考》是正确的(它还没有被认为是规范性的, 只是维护得更好).

Topics that are within the scope of this book include: the meaning of (un)safety, unsafe primitives provided by the language and standard library, techniques for creating safe abstractions with those unsafe primitives, subtyping and variance, exception-safety (panic/unwind-safety), working with uninitialized memory, type punning, concurrency, interoperating with other languages (FFI), optimization tricks, how constructs lower to compiler/OS/hardware primitives, how to **not** make the memory model people angry, how you're **going** to make the memory model people angry, and more.

本书范围内的主题包括: (不)安全代表了什么; 语言和标准库提供的不安全原语(primitive); 用不安全原语创建安全抽象的技术细节; 子类型 (subtyping) 和可变性 (variance); 异常安全 (panic/unwind 安全); 操作未初始化的内存; 类型双关(type punning); 并发 (concurrency); 与其他语言的互操作 (FFI); 优化技巧; 如何构建低级编译器/操作系统/硬件原语; 如何不使内存模型程序员生气, 又如何去使内存模型程序员生气, 等等等等.

The Rustonomicon is not a place to exhaustively describe the semantics and guarantees of every single API in the standard library, nor is it a place to exhaustively describe every feature of Rust.

Rust 死灵书不是一个详尽描述标准库中每一个 API 的语义和保证的地方, 也不是一个详尽描述 Rust 的每一个特性的地方.

Unless otherwise noted, Rust code in this book uses the Rust 2024 edition.

除非另有说明, 本书中的 Rust 代码使用 Rust 2024 版.

[trpl]: https://doc.rust-lang.org/book/index.html
[ref]: https://doc.rust-lang.org/reference/index.html
