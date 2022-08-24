---
title: "使用 GitHub Actions 实现博客自动化部署"
date: 2022-08-24T22:21:41+08:00
draft: true
toc: false
images:
tags: [              
    "blog",
    "deploy"
]
---

## 使用 GitHub Actions 自动化

实现代码提交的自动化工作流，要依靠持续集成（或者加上持续交付）服务。现在使用 GitHub Actions 是 GitHub 自家的持续集成及自动化工作流服务。

### 建立 SSH 密钥

自行百度，略过

### 将自动化配置写到 GitHub 仓库

打开你的网站代码仓库，点击 Settings 标签，找到 Secrets 设定：
选择 Add a new secret，添加一个配置项DEPLOY_KEY，将刚才复制的私钥的内容粘贴在其中。然后，你可以像我上图中一样，把你的服务器 host 、port 和用户名也添加到配置中。这里用户名应该与你上一步操作使用的登录用户一致。

![1](/img/blog/Secrets.png)

添加在这里的配置，将只对你可见，不用担心会泄露给他人。

### 编写工作流文件

好，准备工作都做好了，现在我们来写自动化工作流的配置。

在仓库根目录中创建.github/workflows文件夹，再创建一个 YAML 文件，文件名自定，我这里起名叫deploy.yml，所以文件的完整路径应该为.github/workflows/deploy.yml，我将配置的意义写在注释中，文件内容如下：

```sh
name: Deploy site files

on:
  push:
    branches:
      - master
    paths-ignore:
      - README.md
      - LICENSE

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Server
        uses: AEnterprise/rsync-deploy@v1.0
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
          ARGS: -avz --delete --exclude='*.pyc'
          SERVER_PORT: ${{ secrets.SERVER_PORT }}
          FOLDER: ./
          SERVER_IP: ${{ secrets.SSH_HOST }}
          USERNAME: ${{ secrets.SSH_USERNAME }}
          SERVER_DESTINATION: /home
      - name: Restart server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.DEPLOY_KEY }}
          # 
          script: |
            cd /home
            hugo -D
```

把文件写好，提交到仓库，就可以发现 GitHub Actions 已经启动了！可以在提交历史后面的状态，或者 Actions 标签中看到运行的状态。

![2](/img/blog/actions.png)

## 总结
到此为止就会自动部署到你的服务器上，就可以使用了。有什么不懂得可以联系作者。