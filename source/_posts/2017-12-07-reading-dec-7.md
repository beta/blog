title: "阅读：十二月 7 日"
date: 2017-12-07 20:02:01
tags:
- Reading
- SSH
- Software Engineering
- Android
---

## Upgrade your SSH keys!

（来自[碼天狗週刊 Issue 108](https://weekly.codetengu.com/issues/108)）

[https://blog.g3rt.nl/upgrade-your-ssh-keys.html](https://blog.g3rt.nl/upgrade-your-ssh-keys.html) 

DSA 和 < 2048 bits 的 RSA SSH keys 都不够安全，作者推荐：

 - 使用 Ed25519（`-t ed25519`）取代 DSA 和 RSA。
 - 给 `ssh-keygen` 加上 `-o` 参数，使用 [RFC4716](https://tools.ietf.org/html/rfc4716) 格式，增强 key 面对暴力破解的能力。
   然而 `-o` 的作用是使用 OpenSSH 的格式，而使用 RFC4716 是 `-m RFC4716`。Ed25519 会默认使用 OpenSSH 格式。
 - 使用 `-a <rounds>` 指定 KDF 的次数。次数越高，使用密码验证的过程就越慢，但是防止暴力破解的能力也会越好。作者在自己的电脑测试了 100 次 KDF 大约需要 1 秒来解密 key，还是可以接受的。

简而言之：`ssh-keygen -o -a 100 -t ed25519`，然后提供一个足够强的密码。

OpenSSH 6.5 以上的版本提供了对 Ed25519 的支持。GitHub 支持 Ed25519，但 Launchpad 和 Gerrit 不行（我司就在用 Gerrit 做 code review，遗憾）。Gnome-keyring 处理 OpenSSH 格式有 bug。PuTTY 升级到 late-2015 之后的版本就可以支持 Ed25519 了。

- - -

## Using a logbook to improve your programming

（来自[碼天狗週刊 Issue 108](https://weekly.codetengu.com/issues/108)）

[https://routley.io/tech/2017/11/23/logbook.html](https://routley.io/tech/2017/11/23/logbook.html) 

作者在大学学习的是工程学，经常使用 logbook 来记录工程项目的过程。Logbook 的内容包括：

1. 思考要解决的问题；
2. 描述解决问题的方法；
3. 描述实施方法的过程；
4. 记录发生的事情，以及如何进行优化。

Logbook 可以用在软件开发中，有助于分解问题、记录上下文并且用于反思和自我提升。作者使用 Vim + Markdown 来写 logbook，甚至还提供了一个快速创建 logbook 并打开 Vim 的 shell 函数，很有仪式感。

- - -

## Android Building Image Filters like Instagram

（来自 [AndroidDev Digest - Digest #170](https://www.androiddevdigest.com/digest-170/)）

[https://www.androidhive.info/2017/11/android-building-image-filters-like-instagram/](https://www.androidhive.info/2017/11/android-building-image-filters-like-instagram/) 

作者介绍了如何在 Android 应用中加入滤镜，并且在 UI 上尽可能地模仿 Instagram。不过，文章只是简单概括了滤镜通常是用 C/C++、JNI 和 OpenCV 来做的，并没有介绍具体如何实现。作者在 Zomato 的 [AndroidPhotoFilters](https://github.com/Zomato/AndroidPhotoFilters) 库的基础上做了一些修改，用户只需要添加一条 Gradle 依赖就能获得滤镜功能和一组预设的滤镜包，所以这篇文章更像是库的 getting started/广告。
