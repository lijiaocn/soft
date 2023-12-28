<!-- toc -->

# Git 的 rebase 操作（commit 回放）

rebase 命令将另一个分支上的修改同步到当前分支，但是和 merge 不同，rebase 修改的当前分支的 commit 之前的内容。

举例说明：

1. 原始分支 A
2. 基于 A 创建了分支 B
3. 分支 A 作了一次提交 c1，分支 B 作了一次提交 c2
4. 基于 A 对于 B 分支进行 rebase 后，分支 B 中的提交记录变成  c1 c2

执行 rebase 后，B 分支的内容等同于将 B 分支上的修改在 A 分支上重放。

**rebase 会改变当前分支中的 commit id，通常在 push 之前执行 rebase 操作**

## rebase 的命令参数

rebase 的命令参数如下：

```sh
NAME
       git-rebase - Reapply commits on top of another base tip

SYNOPSIS
       git rebase [-i | --interactive] [<options>] [--exec <cmd>] [--onto <newbase>]
               [<upstream> [<branch>]]
       git rebase [-i | --interactive] [<options>] [--exec <cmd>] [--onto <newbase>]
               --root [<branch>]
       git rebase --continue | --skip | --abort | --quit | --edit-todo | --show-current-patch
```

如果不指定 branch，默认对当前分支进行 rebase，如果指定了 branch，先切换到指定分支，再 rebase。

## 基于 tag 进行 rebase

`--onto` 指定 rebase 回放操作的开始位置，譬如 A 分支上有 b1 b2 c1 三个 commit，`--onto b2` 指定在 A 分支的 b2 位置回放当前分支的 commit。我们可以用这个参数进行基于 tag 进行回放。

如果不使用 --onto，默认在 A 分支最新的 commit 上进行回放，最常遇到的情况是 A 分支上既有 release tag 之前的 commit，也有之后的 commit，我们通常需要在 release tag 的基础上回放，暂时不引用未 release 的 commit。

```sh
git rebase upstream/master --onto nginx-0.25.1
```

upstream/master 中的 upstream 是远程仓库，上面的命令表示在 upstream 中的 master 分支的  nginx-0.25.1  tag 上回放当前分支中的 commit。查看远程仓库的方法：

```sh
git remote -v
origin	http://gitlab.XXXXX.cn/infrastructure/ingress-nginx.git (fetch)
origin	http://gitlab.XXXXX.cn/infrastructure/ingress-nginx.git (push)
upstream	https://github.com/kubernetes/ingress-nginx.git (fetch)
upstream	https://github.com/kubernetes/ingress-nginx.git (push)
```

[Git rebase onto a tag when master and a branch is ahead of the current commits][2] 中讨论了这个问题。

## 如果 rebase 过程出现冲突

用 git status 查看冲突文件：

```sh
$ git status
rebase in progress; onto 5179893a9
You are currently rebasing branch 'nginx-0.25.0-fp' on '5179893a9'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

	both modified:   Makefile

no changes added to commit (use "git add" and/or "git commit -a")
```

修改冲突文件，并提交，然后继续 rebase （git rebase --continue）：

```sh
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
```

## 参考

[1]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/04/01/git.html "Git使用手册"
[2]: https://stackoverflow.com/questions/39768124/git-rebase-onto-a-tag-when-master-and-a-branch-is-ahead-of-the-current-commits "Git rebase onto a tag when master and a branch is ahead of the current commits"
