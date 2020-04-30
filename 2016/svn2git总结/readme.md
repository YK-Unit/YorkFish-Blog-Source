---
title: svn2git总结
date: 2016-11-26 12:34:56
categories: ["技术"]
tags: ["2016", "教程"]
comments: true
---

## 前言
本文主要总结了`svn`迁移到`git`的步骤。

<!-- more -->

## 1. 下载迁移工具[subgit](https://subgit.com/)

`subgit`是一个基于java开发的svn2git商业迁移工具，夸平台，其`import功能`（一次性把代码从`svn`迁移到`git`）是免费的，其他功能（主要是各种`mirror`功能）则是收费的。

## 2. 迁移前的准备工作

由于`svn`用户格式（只有用户名）与`git`用户格式（由用户名和用户邮箱组成）是不一样的，需要创建一个用户映射文件`authors.txt`，以在迁移记录时进行转换。`authors.txt`的内容格式如下：

```
york = york <york@example.com>
kiii = kitty <kitty@example.com>
```

> 如何快速获得`svn`仓库里曾经提交过记录的的用户呢？可通过以下命令行获得：
> 
>```shell
> cd path/to/svn_repo
> svn log --quiet | grep -E "r[0-9]+ \| .+ \|" | cut -d'|' -f2 | sed 's/ //g' | sort | uniq
> ```
> 
> 或者直接从远程仓库获得：
> 
> ```shell
>svn log --quiet http://path/to/root/of/project | grep -E "r[0-9]+ \| .+ \|" | cut -d'|' -f2 | sed 's/ //g' | sort | uniq
>```
>


## 3. 开始迁移

1. 使用subgit的import功能，一次性把代码从svn迁移到git
    ``` shell
    cd svn2git_workspace
    path/to/subgit-3.2.2/bin/subgit import --non-interactive --default-domain YOUR_DOMAIN --authors-file path/to/authors.txt --trunk trunk --tags tags --branches branches --username SVN_USERNAME --password SVN_PASSWORD --svn-url http://svn.example.com/path/to/repo repo.git
     ```
     > 如果迁移过程中遇到错误导致中断，执行 `subgit import repo.git`进行恢复
     >

2. 克隆一个裸库，去掉无用的svn信息
    ```shell
    git clone --bare repo.git repo-clone.git
    ```

3. 推送代码到git远程仓库

    ```shell
    cd repo-clone.git
    git remote add gitlab http://gitlab.example.com/path/to/repo.git
    ```

4. 推送需要的分支到远程参考

    ```shell
    //推送所有本地分支到远程仓库
    git push gitlab --all 
    
    //或者
    //推送指定分支
    git push gitlab master:master
    git push gitlab v1.3.0:develop
    ```

5. 推送所有本地tag到远程仓库

    ```shell
    git push gitlab --tags
    ```

