# Nginx常用操作

## 一些重要摘要

提前了解这些，可以避免踩坑：

[Nginx 新手起步][2]中给出的一个数据比较重要： 

	一般的情况下，10000 个非活跃的 HTTP Keep-Alive 连接在 Nginx 中仅消耗 2.5MB 的内存

[if 是邪恶的][3]，慎用if，如果非要用，最好只在if中使用下面的命令：

	在 location 区块里 if 指令下唯一 100% 安全的指令应该只有:
	
	return …; rewrite … last;

## 参考

1. [API网关Kong（一）：Nginx、OpenResty和Kong的基本概念与使用方法][1]
2. [Nginx 新手起步][2]
3. [if 是邪恶的][3]

[1]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/09/29/nginx-openresty-kong.html "API网关Kong（一）：Nginx、OpenResty和Kong的基本概念与使用方法"
[2]: https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/nginx_brief.html "Nginx 新手起步"
[3]: https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/if_is_evil.html "if 是邪恶的"

