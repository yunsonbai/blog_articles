---
title: git常用命令
date: 2016-05-19 10:41:26
tags:
    - git
    - 常用命令
    - 删除远程分支
    - fetch
    - status
    - branch
    - diff
    - 回滚
categories: git
---
## 引言
```
    git可以说已经渗入到大部分互联网企业,下边记录一下在工作的这段时间用到的命令
```
<!--more-->
### 删除远程分支
``` bash
    git branch -r -d origin/branch-name
    git push origin :branch-name
```
### 创建本地/push到远端分支
``` bash
    git branch dev
    git checkout dev
    git push origin dev
```
### 删除本地分支
``` bash
    git branch 分支名称 -d
    例如，在master分支下，删除新开的dev分支：
    git branch dev -d
    注意：如果dev的更改，push到远程，在GitLab(或者其他git系统)上面进行了merge操作，但是本地master没有pull最新的代码，会删除不成功，可以先git pull origin master，或者强制删除
    git branch dev -D
```
### 创建tag号并push到远端
``` bash
    git tag -a v1.01 -m "Relase version 1.01"
    git push origin -tags(提交全部tag)
    或者：
    git tag -a v1.01 -m "Relase version 1.01"
    git push origin v1.01
```
### 删除远端tag号
``` bash
    git tag -d v1.01
    git push origin :refs/tags/v1.01
```
### merge
``` bash
    git merge 需要合并的分支名
    例如，刚刚已经切换回master，现在需要合并dev的内容：
    git merge dev
    建议在GitLab(或者其他git系统)上面创建merge request的形式来进行分支的合并和代码审核
```
### status
``` bash
    查看发生变动的文件
    git status
```
### 与远程保持同步
```
    git fetch origin
    git rebase origin/master
```
### 追踪远程分支
```
    git branch -vv
```
### 回滚
```
    git reset --hard f46821f94eeb106ea841b9fbafa4bfb76beea691 
    (f46821f94eeb106ea841b9fbafa4bfb76beea691为commit编号)
```
### 查看commit编号
```
    git log
```
### 显示不通
```
    git diff  全部的改动，包括没有add的
    git diff --stage   显示add的，
    git diff  commit1 commit2  （显示两次的不通，输入commit号）
```