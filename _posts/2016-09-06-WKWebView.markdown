---
layout: post
title:  "WKWebView实践分享"
date:   2016-09-06 17:46:23 +0800
description: ezbuy内部iOS小组分享
categories: Home
---

ezbuy内部iOS小组分享

---

		自从公司的`ezbuy` App最低支持版本提升到`iOS8`以后, 使用更多的`iOS8`以后才特有的新特性就被提上了议程, 比如`WebKit`.
	作为公司最没有节操, 最没有底线的程序员之一, 这项任务不可避免的就落到了我的身上.

		既然要使用`Webkit`, 那么首先我们就得明白为什么要使用它, 它相对于`UIWebView`来说, 有什么优势, 同时, 还得知道它的缺陷,
	以及这些缺陷是否会对公司现有业务造成影响.

	首先我们来说说它的优势:

