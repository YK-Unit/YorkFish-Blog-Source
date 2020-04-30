---
title: 配置多个SSH-Key
date: 2017-04-16 12:34:56
categories: ["技术"]
tags: ["2017", "教程"]
comments: true
---

为了安全性，个人的github和公司的gitlab需要配置不同的SSH-Key。具体如下：

1. 切换到系统的SSH目录

    ``` shell
    cd ~/.ssh
    ```

2. 为个人的github生成SSH-Key（若还没有）

    ``` shell
    ssh-keygen -t rsa -C "your_mail@example.com" -f github_rsa
    ```

    然后，前往github添加SSH公钥。

3. 为公司的gitlab生成SSH-Key（若还没有）

    ``` shell
    ssh-keygen -t rsa -C "your_mail@company.com" -f company_rsa
    ```

    然后，前往gitlab添加SSH公钥。

4. 添加配置文件（若还没有）

    ``` shell
    touch config
    ```

5. 为配置文件`config`添加如下内容

    ``` shell
    # github.com
    Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa
    
    # gitlab.company.com
    Host gitlab.company.com
    HostName gitlab.company.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/company_rsa
    ```

6. 测试

    ``` shell
    ssh -T git@github.com
    ```
    输出：
    >Hi YK-Unit! You've successfully authenticated, but GitHub does not provide shell access.
    
    以上表示成功连接到了个人的github。
    然后可以用同样方式测试公司的gitlab。

