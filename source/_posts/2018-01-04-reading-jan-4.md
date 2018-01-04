title: "阅读：一月 4 日"
date: 2018-01-04 14:51:30
tags:
- 阅读
- Go
- Android
- 软件工程
---

## The ultimate guide to writing a Go tool

[https://arslan.io/2017/09/14/the-ultimate-guide-to-writing-a-go-tool/](https://arslan.io/2017/09/14/the-ultimate-guide-to-writing-a-go-tool/)

Go 的 struct tag 为结构体成员添加了额外的键值对，作为额外的描述或选项。作者写了一个名叫 [gomodifytags](https://github.com/fatih/gomodifytags) 的工具，可以自动地为结构体添加/删除各种格式的 struct tag（例如序列化时的 JSON 键名）。具体做法是用 `go/parser` 解析代码得到 AST，用 `go/ast` 操作 AST 找到 struct tag 的位置，然后用作者写的 [structtag](https://github.com/fatih/structtag) 库来修改 struct tag。除了直接修改代码，gomodifytags 还能产生编辑器可用的 JSON 输出。

- - -

## Task Stack

[https://blog.stylingandroid.com/task-stack/](https://blog.stylingandroid.com/task-stack/)

Activity 的回退操作很难处理。在 Lollipop 之前，Android Developer 文档里是把 back 和 up 区分处理的，前者是相对于用户的操作历史，而后者则是依据 app 的逻辑结构。在 Lollipop 之后，这两种操作的界限好像越来越模糊了。

这篇文章比较详细地介绍了如何用 task stack 来构建合理的回退操作，从而让用户在按下返回键时不会觉得困惑。文章针对的场景是：

 1. 如果从同一 app 的 activity A 启动 activity B，那么返回到 activity A。
 2. 如果从通知直接进入 activity B，而 activity B 的 parent activity 是 A，那么返回到 activity A。
 3. 如果后台 task list 中已经包含 activity B，而又从通知进入 activity B，那么返回到 activity A 而不是后台的 B。

对于这三种场景，都需要在 manifest 中为 activity B 添加 `parentActivityName` 和 `meta-data`：

```xml
...
<activity
  android:name=".SecondActivity"
  android:parentActivityName=".FirstActivity">
  <meta-data
    android:name="android.support.PARENT_ACTIVITY"
    android:value=".FirstActivity" />
  ...
</activity>
```

三种场景的解决方案分别是：

 1. 直接调用 `onBackPressed`。
 2. 在通知创建 content intent 时，用 `addParentStack` 添加 activity A 为 parent，用 `addNextIntent` 添加 activity B 为 next intent。
 3. 在 `addNextIntent` 时为 intent 添加 `FLAG_ACTIVITY_SINGLE_TOP` 这个 flag。

`addParentStack` 从字面上来看，参数似乎应该是一系列要添加的 parent activity；然而其实应该填入目标 activity 的 class，manifest 中设置的 parent 会被自动读入。

- - -

## The Software Engineer’s Essential Time Estimation Guide

[https://hackernoon.com/a-software-engineers-essential-time-estimation-guide-d7328238c510](https://hackernoon.com/a-software-engineers-essential-time-estimation-guide-d7328238c510)

估算项目工时是软件工程师的日常工作。由于对自身进度的乐观、沟通成本以及突发情况等因素，大多数人都无法准确地估算项目需要的时间。

作者认为要想准确的估算时间，就要先制定好计划，并且控制好每一个步骤的粒度，既不能模糊不清又不能过于详细。在计划的基础上估计每一步的时间，然后再为一些额外的事项分配时间，例如会议、调试、code review 等等。

最重要的是，在项目完成之后要回过头来检查估算的时间与实际时间是否一致，分析估算不准确的原因，才能在之后更加准确地进行估算。
