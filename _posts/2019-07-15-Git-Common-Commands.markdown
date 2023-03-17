---
layout: post
title:  "Git常用命令"
date:   2019-07-15 11:07:35 +0200
categories: git
---

## 远程仓库相关

* git remote 查看远程仓库名称
* git remote -v 查看远程仓库名称及地址
* git remote show 查看远程仓库详情
* git remote add <shortname> <url> 添加远程仓库
* git fetch <branchname> 拉取远程更新到本地
* git merge <branchname> 合并分支到当前分支
* git pull = git fetch +git merge

## 分支相关

* git branch 创建分支
* git checkout 切换分支
* git checkout -b 创建并切换
* git branch -v 查看每个分支最后一次提交
* git branch --merged 过滤出合并到当前分支的分支

## 标签相关

* git tag 列出标签
* git tag -a -m 创建附注标签并指定信息
* git tag <tagname> 创建标签
* git tag -a <tagname> <commitid> 给指定提交打标签

## 提交相关

* git add . 当前修改全部加入git
* git commit 提交当前修改
* git commit -a 把已跟踪的文件暂存起来一并提交
* git commit --amend 撤销提交
* git checkout --<filename> 撤销修改
* git rm -f 强制删除；git rm --cached 从git仓库中删除但保留在工作目录中
* git mv file_from file_to 移动文件

## 日志状态相关

* git log 查看提交日志
* git status:检查当前文件状态；git status -s(-short):状态简览



