<!-- toc -->
# git 跨 repo 操作

如果使用了一个开源的项目，并且基于这个项目开发了自己的分支，同时又希望把改动反馈到社区，就会遇到在跨 repo 操作的问题。

## 将 github 项目导入 gitlab

使用开源项目，通常需要把开源项目导入到自己公司的 git ，操作方法见 [将github项目导入gitlab][2]。

这里的示例情况如下，origin 是公司私有的 repo，upstream 是开源社区的 repo：

```sh
$ git remote -v
origin      http://gitlab.xxxx.cn/infrastructure/ingress-nginx.git (fetch)
origin      http://gitlab.xxxx.cn/infrastructure/ingress-nginx.git (push)
upstream    https://github.com/kubernetes/ingress-nginx.git (fetch)
upstream    https://github.com/kubernetes/ingress-nginx.git (push)
```

## 添加另一个远程仓库

如果要把改动贡献给开源社区，需要在 github 上 fork 原始项目，通过这个 fork 的 repo 提交 pr。

将 fork 的 repo，作为远程仓库加入：

```sh
git remote add lijiaocn https://github.com/lijiaocn/ingress-nginx.git 
```

加入后，一共有三个远程仓库，分别是 github 上的 fork 分支（lijiaocn）、原始的项目（upstream）、公司私有分支（origin）：

```sh
$ git remote -v
lijiaocn    https://github.com/lijiaocn/ingress-nginx.git (fetch)
lijiaocn    https://github.com/lijiaocn/ingress-nginx.git (push)
upstream    https://github.com/kubernetes/ingress-nginx.git (fetch)
upstream    https://github.com/kubernetes/ingress-nginx.git (push)
origin      http://gitlab.xxxx.cn/infrastructure/ingress-nginx.git (fetch)
origin      http://gitlab.xxxx.cn/infrastructure/ingress-nginx.git (push)
```

添加之后，还要把新的远程仓库的内容拉取到本地：

```sh
$ git fetch lijiaocn
```

然后才能够看到新远程仓库 lijiaocn 中的分支：

```sh
$ git branch -r
  lijiaocn/gh-pages
  lijiaocn/master
  upstream/HEAD -> upstream/master
  upstream/gh-pages
  upstream/master
```

## 创建跟踪另一个远程仓库的分支

```sh
$ git branch lijiaocn_master lijiaocn/master
$ git branch -u lijiaocn/master lijiaocn_master （不是必须）
$ git branch -vv
  lijiaocn_master 846ff0036 [lijiaocn/master: ahead 167] Merge pull request #4560 from Shopify/basic-auth-map
* master          846ff0036 [origin/master: ahead 165] Merge pull request #4560 from Shopify/basic-auth-map
```

## 将远程仓库 A 中的更新 rebase 到远程仓库 B 中

这里将远程仓库 upstream 中的更新同步到 lijiaocn_master 分支中：

```sh
$ git remote -v
lijiaocn    https://github.com/lijiaocn/ingress-nginx.git (fetch)
lijiaocn    https://github.com/lijiaocn/ingress-nginx.git (push)
upstream    https://github.com/kubernetes/ingress-nginx.git (fetch)
upstream    https://github.com/kubernetes/ingress-nginx.git (push)
```

采用 rebase 的方式，合并过程如下：

```sh
$ git fetch upstream    # 将 upstream 的更新同步到本地
$ git rebase upstream/master lijiaocn_master
```

## 将另一个分支中的特定 commit 提交到当前分支

```sh
$ git checkout lijiaocn_master
$ git cherry-pick 800e5fe9dc852fb0 （800..是另一个分支中的 commit）
```

## 将本地分支推送到另一个远程仓库的 master 中

```sh
$ git checkout lijiaocn_master
...进行了一些改动 ...
$ git commit -s -m "提交..."
$ git push -u lijiaocn HEAD:master     # 推送到远程仓库 lijiaocn 的 master 分支中
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/04/01/git.html#%E8%B7%A8%E6%89%98%E7%AE%A1%E5%B9%B3%E5%8F%B0%E5%8D%8F%E4%BD%9C%E5%B0%86github%E9%A1%B9%E7%9B%AE%E5%AF%BC%E5%85%A5gitlab "将github项目导入gitlab"
