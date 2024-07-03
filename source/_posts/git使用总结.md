---
title: Git使用总结
tags:
  - git
author: Eric
date: 2023-8-20
comments: false
cover: /img/2.png
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

### git 注意事项

> 个人分支 merge 回 dev 前，必须先 merge 最新的 dev 代码；命令执行顺序如下：

```js
git add .
gitcommit -m ''
git push
gitcheckout dev
git pull --rebase
gitcheckout jiaojie_20150906
git pull --rebase
git merge dev
gitcheckout dev
git merge jiaojie_20150906
```

- 最后要切回自己的分支,另外，关于 git 操作最近出问题比较多的是分支合并。正确的方法是：

> 1. 先在个人分支提交
> 2. 切换到 dev 分支，并且将本地 dev 更新到最新状态，即执行 git pull（如果有需要 commit 的，更新完后要 push，使本地和 remote 同步）
> 3. 切换到个人分支，merge dev（如果有冲突，解决冲突，切勿直接将个人分支合并到 dev 分支）
> 4. 切换到 dev 分支，将个人分支合并到 dev
>
> 给个建议，每次操作后执行 gitstatus，查看当前状态，可以帮助你清楚的知道上一步操作的结果和目前的状态，有利于你 在执行下一步操作前作出正确的判断。如果需要删除已经被 git 管理的文件，指令是 git rm xxx，支持批量删除，用通配 符就行。
>
> 注：如果是 untracked 文件，不能使用 git rm，而是 rm 指令。每一步操作完如果不清楚当前状态，执行 gitstatus 查看，确定是你要的状态再做下一步操作

- merge 代码的顺序是：

```js
1. 提交自己开发分支（yourBranch）的代码
2. 切到dev，将本地dev代码更新到最新，指令是：git pull
3. 切到自己开发分支，将dev合并过来并提交，指令是：git merge dev 和git push
4. 切到dev，将自己开发分支合并到dev并提交，指令git merge yourBranch，git push
5. 新建远程分支 gitcheckout -b 名称
6. 提交该分支到远程仓库 git push origin -u 名称
```

```js
//git push --set-upstream origin jiaojie_20151229
git branch -d branch-name 删除本地分支
 //删除远程分支
git branch -r -d origin/branch_name
git push origin :branch_name
//配置git快捷键
git config --global alias.st status
git config --global alias.co checkout
git config --global user.name "username"
git config --global user.email "email"
ssh-key生成 $ssh-keygen -t rsa -C "jiaojie@intra.nsfocus.com"
//把id_rsa.pub加载10.65.120.68 /home/git/.ssh/authorized_keys 中就可以不输入密码了
//查看远端仓库git地址
git remote -v
git clone git@10.65.120.68:repositories/espc-web
git log //则不能察看已经删除了的commit记录(使用q退出)
git log --stat  //查看提交文件的历史记录
git reflog //可以查看所有分支的所有操作记录（包括（包括commit和reset的操作）
git fetch --prune //清除本地陈旧分支
//1. Git 回滚前一个版本的命令：
git reset --hard HEAD~1
git checkout 分支名
git cherry-pick [hashKey]
// http://blog.csdn.net/ybdesire/article/details/42145597
// 1. 本地分支重命名
Git branch -m oldbranchname newbranchname
// 1. 远程分支重命名 (假设本地分支和远程对应分支名称相同)
// a. 重命名远程分支对应的本地分支
git branch -m old-local-branch-name new-local-branch-name
// b. 删除远程分支
git push origin :old-local-branch-name
// c. 上传新命名的本地分支（把本地分支上传到远程）
git push origin new-local-branch-name: new-local-branch-name
git reflog show --date=iso <branch name> 查看分支创建时间
// 1. git 删除本地分支
git branch -D/-d [branch_name]
// 删除远程分支
git push origin :[branch_name]
//或者
git branch -r -d origin/[branch_name]
// 1.  清理本地陈旧分支
git fetch --prune
// 1.  git 日志查看
git log --pretty=oneline
// 1.  git 标签相关操作
//a. 打标签
git tag [tagName] // ---- 创建轻量标签
git tag -a [tagName] -m "commit state" //---- 创建附注标签 （推荐）
//b. 切换到标签
git checkout [tagName]
// c. 查看标签信息
git show [tagName]
// d. 删除本地标签
git tag -d [tagName]
// e. 给指定的 commit 打标签
git tag -a [tagName] [commit] -m "标签说明"
// f. 标签发布
git push origin [tagName] # //将[tagName]标签提交到远端
git push origin --tags //#将本地所有标签一次性提交到远端
// g. 删除远程标签
git push origin :refs/tags/[tagName] 或
git push origin --delete tag [tagName]
git reset --hard commit_id //随意切到某个版本
git push -f origin branch_name //强制提交代码
// PPT 版
```

![参考文献](https://github.com/xirong/my-git/blob/master/useful-git-command.md)
