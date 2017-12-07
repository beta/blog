title: "阅读：十二月 6 日"
date: 2017-12-06 21:07:32
tags:
- 阅读
- Python
- Django
---

## Understanding Python's "yield" Keyword

（来自 [Pycoders Weekly Issue #295](https://mailchi.mp/pycoders/pycoders-weekly-issue-263-source-209781)）

[http://stackabuse.com/understanding-pythons-yield-keyword/](http://stackabuse.com/understanding-pythons-yield-keyword/)

一篇关于 Python 的 `yield` 关键字（以及 generator）很好的入门文章。

 - Generator 是一种 collection，但相比于 list，generator 并不会将所有元素一次性存入内存，而是按需创建每一个元素。因此 `len(generator)` 会报错，而且 generator 只能被遍历一次。内置函数 `next` 返回 generator 中的下一个元素，当 generator 遍历至末尾时，`next` 抛出 `StopIteration` 异常。
 - 创建 generator 的 list comprehension：
   <script src="https://gist.github.com/beta/d21dc6ed4eaa4c49d8cb1b5a5eadb210.js"></script>
 - 在函数中使用 `yield` 关键字代替 `return`，可以让函数返回 generator。对于 generator 函数，`next` 使其停在下一个 `yield` 处并返回被 yield 的值。
 - 如果程序中包含巨大的列表，并且不需要反复遍历，那么适合使用 `yield` 生成 generator 来降低内存使用量。

- - -

## Django 2.0 released

（来自 [Pycoders Weekly Issue #295](https://mailchi.mp/pycoders/pycoders-weekly-issue-263-source-209781)）

[https://www.djangoproject.com/weblog/2017/dec/02/django-20-released/](https://www.djangoproject.com/weblog/2017/dec/02/django-20-released/) 

Django 发布 2.0 版本了。新特性有：

 - 简化了 URL 路由语法，例如
   `url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),`
   可以写成
   `path('articles/<int:year>/', views.year_archive),`。
   `django.conf.urls.url()` 函数变成了 `django.urls.re_path()` 。
 - `contrib.admin` 对移动设备更友好了。
 - 全新的 `Window` 表达式。

还有一些向下不兼容的改动：

 - 不再支持 Python 2.7，只支持 Python 3.4、3.5 和 3.6。
 - 既然不再支持 Python 2，那么在很多地方也就不再支持 bytestring 了。
 - [数据库 backend API 有了很多变动](https://docs.djangoproject.com/en/2.0/releases/2.0/#database-backend-api) 。
 - 不再支持 Oracle 11.2。
 - 将 MySQL 默认的 transaction isolation level 改为 `READ COMMITTED`（之前是 `REPEATABLE READ`）。
 - 将 `AbstractUser.last_name` 长度上限扩大到了 150（之前是 30）。
 - 在对 `QuerySet` 进行切片之后，不允许再调用 `reverse()` 和 `last()`。
 - 自带的 form fields 的可选参数不能再作为 positional 参数传递。
 - `call_command` 开始验证命令的选项了。
 - `Index` 不再接受 positional 参数了。
 - 启用了 SQLite 的 foreign key constraints。

- - -

## User experience design for APIs

（来自 [碼天狗週刊 Issue 108](https://weekly.codetengu.com/issues/108)）

[https://blog.keras.io/user-experience-design-for-apis.html](https://blog.keras.io/user-experience-design-for-apis.html)

在设计和开发面向用户的软件时通常要考虑 user experience（UX），而隐藏在软件用户界面背后的 API 也需要为其用户（开发者）优化使用体验。作者介绍了他在设计 API 时的三条规则：

 1. 设计整体的 workflow，而不是 atomic methods（使 workflow 贴合领域需求、为新用户提供入门指南）。
 2. 减轻用户理解 API 的认知负担（命名一致、减少概念、带例子的文档）。
 3. 为用户提供有帮助的反馈（FAQ、详细的错误信息、建立提问社区）。

对于第 2 条，我最近在工作中感受比较深。公司用 Java 做了一个测试平台，在编写测试用例时必须掌握的概念有 test group、test case 和 test suite，不同的 test suite 又有不同的专用名称，再辅以我司一贯的滥用缩写的风格，痛不欲生。
