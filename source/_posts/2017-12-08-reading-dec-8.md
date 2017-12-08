title: 阅读：十二月 8 日
date: 2017-12-08 19:54:58
tags:
- 阅读
- Python
- 创业
- Android
- 字体
---

## How — and why — you should use Python Generators

[https://medium.freecodecamp.org/how-and-why-you-should-use-python-generators-f6fb56650888](https://medium.freecodecamp.org/how-and-why-you-should-use-python-generators-f6fb56650888) 

这篇文章是对上次读过的 [_Understanding Python's "yield" Keyword_](https://kyouko.net/post/2017-12-06-reading-dec-6.html#Understanding-Python’s-“yield”-Keyword) 的很好的补充。作者给出了一个新的视角：Python 中的 Iterator 可以用来实现 lazy evaluation，通过给类实现 [Iterator Protocol](https://docs.python.org/3/c-api/iter.html)（`__iter__` 和 `__next__` 方法），就可以在遍历的过程中动态地计算出较大的数据集。`yield` 关键字和 generator 的引入为构建 Iterator 提供了一种更加 Pythonic 的方法，通过保存函数的状态来实现 lazy evaluation。

- - -

## Using Python’s Pathlib Module

（来自 [Pycoders Weekly Issue #294: Release](https://mailchi.mp/pycoders/pycoders-weekly-issue-263-source-209777)）

[http://pbpython.com/pathlib-intro.html](http://pbpython.com/pathlib-intro.html)

Python 3.4 引入的 Pathlib 是一个面向对象的文件系统接口。作者用 Pathlib + Pandas 做了一个管理和排序复杂目录结构下大量文件的工具，并制作了一份 [cheat sheet](https://github.com/chris1610/pbpython/blob/master/extras/Pathlib-Cheatsheet.pdf) 总结了 Pathlib 中常用接口的用法，很清楚地给出了 Windows 和 POSIX 的区别。再也不要手动拼接路径和 URI 啦。

- - -

## Python Hashes and Equality

（来自 [Import Python Newsletter Issue - 152](http://importpython.com/newsletter/no/152/)）

[https://hynek.me/articles/hashes-and-equality/](https://hynek.me/articles/hashes-and-equality/)

本文介绍了 Python 中对象比较和哈希的机制。关于比较：

 - 对象默认根据 identity 判等。自定义比较的机制可以通过重载 `__eq__(self, other)` 来实现。
 - Python 2 中必须重载 `__ne__(self, other)` 才能使用 `!=`。Python 3 会自动实现这个方法。

而关于哈希：

 - 对于可哈希（[hashable](https://docs.python.org/3/glossary.html#term-hashable)）的对象，`hash()` 函数返回一个整数哈希值。
 - 要使类的实例可哈希，需要重载 `__hash__(self)` 和 `__eq__(self, other)`。默认情况下 `object.__hash__(self)` 根据实例的 identity 计算哈希值。

如果需要实现哈希，作者提出了两点容易踩坑的地方：

 - 一个对象的哈希值在对象的生命周期内必须保持不变。
 - 两个可哈希的对象如果相等，那么必须具有相同的哈希值。

- - -

## How I Bootstrapped My Side Project to $6k/Month While Hating Everything It Stands For

[https://www.indiehackers.com/@EO/how-i-bootstrapped-my-side-project-to-6k-month-while-hating-everything-it-stands-for-43c35faa25](https://www.indiehackers.com/@EO/how-i-bootstrapped-my-side-project-to-6k-month-while-hating-everything-it-stands-for-43c35faa25)

很有趣的创业经历分享。作者开发了一个叫 [AppyGEN](https://appygen.net/) 的 SaaS 平台，可以通过套用模板的方式生成*原生* Android 应用，帮助不懂编程的人「借鉴」其他应用的经验、发布到 Google Play 并以此获利。后来作者意识到，这种模式会严重破坏市场生态，而不像其他的创业公司那样「make the world a better place」，但是作者已经为 AppyGEN 投入了太多人力物力，因此只能继续把它作为一项事业做下去。

之后作者通过引入游戏化的激励机制（gamification）以及十分诱人的邀请注册回报，成功地将这个平台做到了月收入 $6k 以上。以下是作者给予创业者的几点建议：

 - 不要把赚钱当成最主要的目的。
 - 不要闷头开发，要多与别人分享。
 - 要做就做能让自己觉得自豪的事业。

- - -

## The Android Developer’s Guide to Better Typography

（来自 [AndroidDev Digest #171](https://www.androiddevdigest.com/digest-171/)）

[https://medium.com/google-design/the-android-developers-guide-to-better-typography-97e11bb0e261](https://medium.com/google-design/the-android-developers-guide-to-better-typography-97e11bb0e261)

Jelly Bean（4.1，API level 16）之后的 Android 版本以及 Google Play Services 12 以上的版本支持 Google Fonts provider，应用可以从 [Google Fonts](https://fonts.google.com/) 下载字体，下载后的字体可以被手机中的所有应用使用，不仅可以缩减 APK 体积，也避免了重复占用用户的存储空间。Android Studio 3.0 增加了 Downloadable Fonts 的功能，可以让开发者方便地使用 Google Fonts 了。文章以一个 [修改过的 Plaid 项目](https://github.com/rsheeter/plaid) 代码为例介绍了 Downloadable Fonts 的用法。

Android 终于有了官方的字体方案，依托于 Google Fonts，应该可以满足西文用户的 UI 字体需要了。但是对于 CJK 文字而言，至今依然没有可用的 web fonts 解决方案，而且考虑到 Google Play Services 在国内的覆盖程度和可用性，看看就好。
