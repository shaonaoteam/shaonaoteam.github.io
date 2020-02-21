---
layout:     post
author:     zjhChester
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Blog
---
# 2020-02-04-使用Gitalk 插件出现的未找到相关的issues进行评论

## 前言

在整合GItPage和Gitalk 的时候，我看人家同样的方式都部署成功了，硬生生弄了几个小时一直都是一个状态xxxx暂无issues，请联系xxx初始化，然后点登录以后就自动跳到首页

![1580796758775](https://zjhchester.github.io/img/Gitalk1.png)

![1580796758788](https://zjhchester.github.io/img/Gitalk2.png)

![1580797021676](https://zjhchester.github.io/img/Gitalk3.png)

这个问题就是在申请github application的时候，里面主页地址和回调函数地址一定要精确到大小写，还有精确到https和http，最好就直接复制主页地址，这个解决了你的application使用者就不会为0了

最后附上一个Github Application<a href="https://github.com/settings/applications/new ">申请地址</a>

![logo](https://zjhChester.github.io/img/apple-touch-icon.png)

