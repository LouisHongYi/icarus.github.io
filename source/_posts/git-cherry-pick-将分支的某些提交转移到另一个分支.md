---
title: git cherry-pick 将分支的某些提交转移到另一个分支
date: 2022-04-29 16:28:41
tags: git
cover: /img2/git.png
copyright_author: hong
copyright_author_href: 
copyright_url: https://louishongyi.github.io/icarus.github.io/2022/04/21/use-hexo-and-github-to-create-you-blog/
copyright_info: 此文章版权归 hong 所有，如有转载，请註明来自原作者
---

`cherry-pick`，不是挑选樱桃，而是挑选最好的。= =！

`cherry-pick` 可以将一个分支的某一些提交，转移到另外一个分支，这样在另外一个分支，就可以拥有这些 `commit` 记录。

我从 `develop` 分支拉了分支A进行开发，开发完以后，还没有到发版的日期，但是这个需求比较急，就要以hotfix的形式上到 `master` 分支。所以我又从 `master` 拉了分支B， 需要将A的提交转移到B上，然后将B `merge` 到 `master` 。这个时候 `cherry-pick` 就非常有用了。

[cherry-pick 文档](https://git-scm.com/docs/git-cherry-pick)

先说一说这一次我是怎么做的。

`cherry-pick` 既然是分支B接收分支A的提交，那么首先我需要知道这个分支A做了哪些提交，哪些提交是需要合并到分支B的。

先查看A分支的历史记录。由于我本地就有A分支，所以使用 `git reflog`

![Untitled](1.png)

红色部分就是我在A分支做的提交，由于时间最久的 `c9219f9f50` 是不需要 `merge` 的，所以它不需要合并到B分支。所以实际上我需要合并的只有三个commit。

```bash
-- 切换到B分支，然后合并commit到B分支
git checkout B
git cherry-pick df3cdf3aea 8af5372a10 82146db6a3 
```

有一点需要注意的是，我们在 `cherry-pick` 后面**加commit的 `head hash` 的时候，是有顺序的，最早的提交放到最前面。**

然后我们就可以在B上push代码了, push的时候就可以看到，有三个commit。

这是我第一次使用 `cherry-pick` ，很简单也很好用，没有碰到有冲突的情况。

`cherry-pick` 还有很多其他的命令。

合并单个commit 记录

```bash
git cherry-pick <commit>
```

合并从 `start-commit` 到 `end-commit` 的提交记录。 **不包含** `start-commit`

```bash
git cherry-pick <start-commit>..<end-commit>
```

合并从 `start-commit` 到 `end-commit` 的提交记录。加了一个 `^`, **包含** `start-commit`

```bash
--
-- 合并从 start-commit 到 end-commit 的提交记录
git cherry-pick <start-commit>^..<end-commit>
```

题外话： 你是使用git多久的时候遇到`cherry-pick`?

