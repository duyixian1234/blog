---
title: 推荐一些好用的git别名
date: 2020-03-10 21:11:02
tags: git
categories: git, 开发
---

## 背景

开发人员每天运行最多的命令是哪一个？`npm`,`mvn`,`make`...或者是大家都猜到的————`git`。

我们在完成一个需求的开发，或者修复一个问题后，往往会进行如下一系列`git`命令：

```bash
git add .
git commit -m "Feature A (OR Fix BUG B)"
git push
```

另外还有一些命令如`git status`,`git fetch`,`git log`,`git checkout`也是我们经常要用到的。

其实大家完全可以把一些常用的`git`命令设置成`git`别名，方便日常使用。

<!-- more -->

## 实战

`git`别名需要设置在`~/.gitconfig`的`alias`项下，一个最简单的`git`别名只需要如下修改：

```Properties
# ~/.gitconfig
[alias]
    st =  status
    aa = add .
```
然后在终端输入`git st`,就会执行`git status`了。

在这个例子中我们只是少了三次按键，不过我们完全可以设置一些更强大的`git`别名。

下面是一些我个人推荐的一些好用的`git`别名:

```properties
# ~/.gitconfig
# This is Git's per-user configuration file.
[user]
# Please adapt and uncomment the following lines:
    name = duyixian
    email = duyixian1234@qq.com

[format]
  pretty = format:%h %Cblue%ad%Creset %an %Cgreen%s%Creset # 设置log格式，--oneline的输出不带作者和日期

[alias]
    a = add
    b = branch
    c = commit
    st =  status

    # 同步
    get = !git fetch --prune && git pull --rebase=preserve && git submodule update --init --recursive
    put = !git commit --all && git push

    # 提交
    aa = add .
    amend = commit --amend --reuse-message=HEAD # 合并当前缓冲内容到上一次的提交并复用提交信息
    aamend = commit -a --amend --reuse-message=HEAD # 将修改过的文件添加到缓冲区合并到上一次提交并复用提交信息
    uncommit = reset --soft HEAD~1 # 撤销上次提交

    # 推送
    publish = "!git push -u origin $(git branch-name)" # 推送当前分支


    # 日志
    recent = log -n 5 # 列出最近五次提交
    today = !git log    --after="6am"  # 列出今天的提交
    week = !git log   --author $(git config user.email) --after="1.weeks" --date=short  # 列出当前作者这周的提交，周报专用
```

## 总结

大家可以设置自己常用的`git`别名，不用照本宣科，重要的是理解背后的`git`命令，从而能够切实提高效率。