---
title: "Git Lfs"
description:
date: "2023-01-12T15:41:12+08:00"
slug: "gitlfs"
# image: "git-lfs.gif"
license: false
hidden: false
comments: false
draft: false
tags: ["Git"]
categories: ["Git"]
---

在GitHub或者GitLab提交超过50M的大文件的时候，无法上传
![git-maximum-file-size](git-maximum-file-size.png)

## 什么是Git LFS

> Git 是分布式版本控制系统，这意味着在克隆过程中会将仓库的整个历史记录传输到客户端。\
> 对于包涵大文件（尤其是经常被修改的大文件）的项目，初始克隆需要大量时间，因为客户端会下载每个文件的每个版本。\
> Git LFS（Large File Storage）是由 Atlassian, GitHub 以及其他开源贡献者开发的 Git 扩展，它通过延迟地（lazily）下载大文件的相关版本来减少大文件在仓库中的影响，具体来说，大文件是在 checkout 的过程中下载的，而不是 clone 或 fetch 过程中下载的（这意味着你在后台定时 fetch 远端仓库内容到本地时，并不会下载大文件内容，而是在你 checkout 到工作区的时候才会真正去下载大文件的内容）。

<!-- ![git-lfs](git-lfs.gif) -->

## 参考文献

- [Atlassian官方LFS文章](https://www.atlassian.com/git/tutorials/git-lfs)
