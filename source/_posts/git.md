---
title: git
date: 2018-01-14 19:56:55
tags: Pro Git
---

```
git difftool //使用对比工具
git log -p -2//查看最近2次提交的详细信息
git log -stat //可以看到变更的文件
git log --since=2.weeks//2周内的日志
git log --author zhichen//指定作者
git log -S function_name //找出有对function_name的修改日志
git remote -v//git 地址
git push origin --tags//推送所有tag
git branch -vv//查看各分支head情况
git push origin --delete serverfix//删除线上分支
git rebase master branchName; gitmerge branchName;//将branchName分支变基为master 在合并到master 这样功能和合并分支一样, 但是提交历史看起来像是线性开发而不是并行开发再合并的 
git reset --hard HEAD; //取消本地操作 恢复成上一次提交记录

```