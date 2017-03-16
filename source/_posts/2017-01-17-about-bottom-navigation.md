title: "关于 Bottom Navigation"
date:  2017-01-17 12:00:00
tags:
- Android
- Material Design
---
去年三月，Luke Wroblewski——Google 的一位 Product Director——在 Google+ 上发表了好几条 po，宣布 material design guidelines 正式收纳底部导航栏（bottom navigation）。

> [Bottom navigation bars are now part of Google's Material design guidelines.][1]——Luke Wroblewski

那时候我非常气愤。不久之前 HTC 宣布 8X 无法升级到 Windows 10，苦等许久却被泼了这么一瓢冷水，于是我放弃了 Windows Phone，转而入行 Android 应用开发；此时 Google 来这么一出，不禁让我有一种「Google 药丸」「我们中出了一个 iOS 叛徒」的失落感。再加上 Android 的虚拟按键和 material design 里已有的诸如 Snackbar 的组件，N 下巴之美不可避。

（此处应该 [@Luke 今天被开除了吗][2]。）

时隔数月，闲暇时把玩手机上的应用，发现一向觉得设计不错的酷市场居然也在新版换上了底部导航栏，而且高度只有 48dp，并不是 guidelines 里要求的 56dp。稍感遗憾之余，开始琢磨 bottom navigation 是否有存在的意义，以及如何正确地使用这一导航模式。

不得不说 Google 对这个部件的定位还是很准确的。在这之前我（们）经常把 iOS 应用下方的那一栏称为「底部标签栏」，换言之，我（们）把那几个图标视为标签。标签栏和导航栏的区别在哪里？考虑 Chrome 的标签页、Windows 与 macOS 桌面应用程序的 Tab 控件，可以看出标签的作用主要在于区分与切换同级视图，多用于查看内容，而不是指引功能；而导航栏，顾名思义，是指引用户使用功能的入口。

Windows Phone 8 的设计准则里也有类似的组件，之前叫 App Tabs 和 Paranoma，后来改名为 Pivot 和 Hub。对于前者，文档里是这样描述的：

> The App Tab style… Each tab could basically be a different way to filter and view data that you want presented to the user… Use an App Tab style for apps that have a central data type that you present different filtered views around.

而对于后者：

> Microsoft provides a UI control called the Paranoma control that can be used as the central application hub. This control allows the user to navigate into all of the areas of functionality in the app.

区别很明显，标签用于展示数据，是同组数据的不同展示方式，例如电商应用的订单界面，「全部订单」、「未发货」、「未收货」等等就是使用标签很好的例子；而导航用于分隔并进入功能。因此，Material design 的 bottom navigation 之所以称为 navigation 而不是 bottom tabs bar，其目的就是用来放置应用中最重要的几个**顶级功能**的入口。

> Bottom navigation should be used for:
> · Three to five top-level destinations of similar importance
> · Destinations requiring direct access from anywhere in the app
> —— [Material design guidelines][3]

单从这一点，就能判断很多应用到底有没有正确地使用 bottom navigation。这里原本打算截图 + Sketch 画图细说，但时间有限，只能稍微列举几个了。没学过设计，随心之言，不当之初请指正交流。

## 微信

可以用 bottom navigation，但用得不妥。「聊天」可以算作一个功能入口，但「联系人」不能算，「发现」和「我」是导航的导航，更不能算作同级。

可以放上「聊天」和「朋友圈」，还差一个，我觉得可以放一个「订阅」，按时间线展示订阅号、公众号的更新内容。其他入口可以像 Telegram 一样收进 navigation drawer 里，「联系人」也可以用「新建聊天」的 floating action button 来代替。

很多国内应用右下角都要带个「我」的入口，里面裹着各种各样乱七八糟的功能和数据入口。都在被微信带着跑。

## 网易云音乐

适合用 bottom navigation，然而并没有用，导致顶部的导航十分密集，左右滑动浏览非常麻烦。

作为一个音乐播放器 + 社交的产品，有「发现音乐」、「我的音乐库」和「社交动态」三个主要功能，挺适合放在 bottom navigation 里的。

乱七八糟的次要功能收在 navigation drawer 里，这一点网易云音乐做得很好。

## Instagram

不论是 Android 还是 iOS，Instagram 都在导航栏里放了一个发布的 action，功能与功能导航混在一起，很反人类。Instagram 给人的感觉不像一个客户端应用，而更像一个浏览器，比如：

![Imgur][image-1]

Instagram 在功能上是个挺纯粹的产品（看图和发图），所以没必要一定要用导航栏。硬要用的话，除了「时间线」和「发现」，我也只能想到放上「Instagram Direct」了。果然是每个应用都会有一套内建 IM 机制啊。

[1]: https://plus.google.com/+LukeWroblewski/posts/KywisYsPYV1
[2]: http://www.weibo.com/chihaneh
[3]: https://material.io/guidelines/components/bottom-navigation.html#bottom-navigation-usage

[image-1]: http://i.imgur.com/ICkIDka.png
