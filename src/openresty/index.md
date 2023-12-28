<!-- toc -->
# OpenResty 学习笔记

这里是对 OpenResty 相关笔记的重新整理，原笔记不再维护。

[OpenResty][1] 是什么？见 [Nginx、OpenResty 和 Kong入门][2]。

## 学习资料

[OpenResty 最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/) 是一份很好的学习资料。

OpenResty 在 nginx 的基础上改造加强，提供的开发语言是 lua，用 lua-nginx-module 解释 lua 脚本，从下面的文档中几乎可以找到所有用法：

* lua 语言用法：[Lua编程速查手册（常用操作)][8]
* nginx 指令汇总：[nginx directives][11]
* nginx 变量汇总：[nginx variables][12]
* openresty 提供的 lua 能力：[openresty/lua-nginx-module][10]
* openresty 集成的模块和 lua 库：[openresty components][13]

以前的笔记：

* [《Web开发平台OpenResty（一）：学习资料、基本组成与使用方法》][3]
* [《Web开发平台OpenResty（二）：组成、工作过程与原理》][4]
* [《Web开发平台OpenResty（三）：火焰图性能分析》][5]
* [《Web开发平台OpenResty（四）：项目开发中常用的操作》][6]
* [《Web开发平台OpenResty（五）：OpenResty项目自身的编译》][7]

## 参考

* [李佶澳的博客][9]

[1]: http://openresty.org/en/ "OpenResty"
[2]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/09/29/nginx-openresty-kong.html "Nginx、OpenResty和Kong入门"
[3]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/10/25/openresty-study-01-intro.html
[4]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/10/25/openresty-study-02-process.html
[5]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/11/02/openresty-study-03-frame-md.html
[6]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/11/09/openresty-study-04-development.html
[7]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/12/17/openresty-study-05-compile.html
[8]: https://www.lijiaocn.com/prog/lua/ "Lua编程速查手册（常用操作)"
[9]: https://www.lijiaocn.com "李佶澳的博客"
[10]: https://github.com/openresty/lua-nginx-module "lua-nginx-module"
[11]: http://nginx.org/en/docs/dirindex.html "nginx directives"
[12]: http://nginx.org/en/docs/varindex.html "nginx variables"
[13]: http://openresty.org/en/components.html "OpenResty Components"
