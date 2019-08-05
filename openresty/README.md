# OpenResty介绍

内容准备中...移步[Web开发平台OpenResty](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/10/25/openresty-study-01-intro.html)

最近（2018-12-01 22:22:13）在研究一个名为[Kong][1]的api网关，它是在[OpenResty][2]上实现的。
为了搞清楚Kong的运作原理，着实花费了一番功夫：

```
先是学习OpenResty，发现OpenResty是基于Nginx的实现的，于是去学习Nginx。
搞清楚OpenResty是如何将Nginx改造成一个Web应用运行平台后，再回头看Kong的实现。
Kong的开发语言是Lua，又去学习Lua。
```

折腾了一圈之后，总算大概搞清楚[Nginx][4]、[OpenResty][2]、[Lua][6]和[Kong][1]之间的关系，以及它们各自的工作原理。学习过程中随手做的笔记在[系列教程汇总][5]中。

[OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/)是一份很好的学习资料，有一个名为“Kong/Envoy实践互助”的QQ群，是一个比较好的讨论场所，群号是：952503851。

[1]: https://docs.konghq.com/ "kong"
[2]: http://openresty.org/en/ "OpenResty"
[3]: https://www.lijiaocn.com/tags/class.html "lijiaocn.com class"
[4]: http://nginx.org/ "nginx"
[5]: https://www.lijiaocn.com "lijiaocn.com"
[6]: https://www.lijiaocn.com/programming/chapter-lua/ "lua"

