## 一、写在前面

git merge和rebase区别

> 参考链接：https://www.cnblogs.com/xueweihan/p/5743327.html

## 二、GIT

首先要明确，merge和rebase都是用来合并分支的。

### 1、合并分支

每个人创建一个分支进行开发，当开发完成，需要合并到develop分支的时候，就需要用到**合并**的命令。

> 建议每个repo都有master分支和develop分支，每个feat分支或者fix分支开发完成后，都先merge到develop分支，然后定期发布到master分支，这看起来是一个最佳实践？

### 2、合并冲突

合并的时候，有可能会产生冲突。

冲突的产生是因为在合并的时候，**不同分支修改了相同的位置**。所以在合并的时候git不知道那个到底是你想保留的，所以就提出疑问（冲突提醒）让你自己手动选择想要保留的内容，从而解决冲突。（解决方法往往是手动修改完代码后 ，再手动git commit一次就好了）

### 3、git merge和git rebase的区别

1. 采用merge和rebase后，git log的区别，比如在合并到master分支时，

   merge命令：在master分支下不会保留该分支的commit

   rebase命令：在master分支下会保留该分支的commit，见下图：

   (准确的说，不应该叫保留该分支的commit，应该说是，有没有抹去该分支存在过的证明？)



![img](https://images2015.cnblogs.com/blog/759200/201608/759200-20160806092734215-279978821.png)

### 4、处理冲突的方式

- （一股脑）使用`merge`命令合并分支，解决完冲突，执行`git add .`和`git commit -m'fix conflict'`。这个时候会产生一个commit。

- （交互式）使用`rebase`命令合并分支，解决完冲突，执行`git add .`和`git rebase --continue`，不会产生额外的commit。这样的好处是，‘干净’，分支上不会有无意义的解决分支的commit；坏处，如果合并的分支中存在多个`commit`，需要重复处理多次冲突。

  > ==所以每次报冲突，都是在一个一个commit的报冲突？一个报完下一个接着报？应该不是这样吧？==



### 5、`git pull`和`git pull --rebase`区别

`git pull`做了两个操作分别是‘获取’和合并（fetch和merge）。所以加了rebase就是以rebase的方式进行合并分支，默认为merge。

==下面内容待检验！！！！==

### 6、`git merge` 和 `git merge --no-ff`的区别

1、我自己尝试`merge`命令后，发现：merge时并没有产生一个commit。不是说merge时会产生一个merge commit吗？

**注意**：只有在冲突的时候，解决完冲突才会自动产生一个commit。

如果想在没有冲突的情况下也自动生成一个commit，记录此次合并就可以用：`git merge --no-ff`命令，下面用一张图来表示两者的区别：







![img](https://images2015.cnblogs.com/blog/759200/201608/759200-20160806092744747-1816899042.png)

2、如果不加 --no-ff 则被合并的分支之前的commit都会被抹去，只会保留一个解决冲突后的 merge commit。

## 如何选择合并分支的方式

我的理解：主要是看那个命令用的熟练，能够有效的管理自己的代码；还有就是团队用的是那种方式。

我对于rebase比较熟悉，所以我一般都用`rebase`，但是现在的公司用的是`merge --no-ff`命令合并分支。所以，我在工作上就用merge，个人项目就用rebase。

也可以两者结合：

- 获取远程项目中最新代码时：`git pull --rebase`，这个时隐性的合并远程分支的代码不会产生而外的commit（但是如果存在冲突的commit太多就像上面说的，需要处理很多遍冲突）。
- 合并到分支的时候：`git merge --no-ff`，自动一个merge commit，便于管理（这看管理人员怎么认为了）
