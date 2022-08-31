---
title: "使用 CODING 自动化部署 PHP+SWOOLE 框架 Hyperf 项目"
date: 2022-08-31T23:40:14+08:00
draft: true
toc: false
images:
tags: [              
    "blog",
    "deploy",
    "php",
    "hyperf"
]
---

## 创建基础项目
1. 首先我们在右上角创建一个项目。

![1](/img/blog/SCR-20220831-x0m.png)

2. 然后选全功能 DevOps 项目，创建好项目。

![2](/img/blog/SCR-20220831-x44.png)

#### 进入项目后点击右上角，创建一个代码仓库

![3](/img/blog/SCR-20220831-x5z.png)

3. 安装 Hyperf, 或者直接去github拉取hyperf项目，此处不做示范

## 编写构建计划

首先检出分支，然后会根据分支下的 Dockerfile 生成镜像，然后推送镜像到 coding 的制品仓库。最后登录服务器拉取镜像进行部署。

### 创建构建计划

1. 首先我们先创建一个构建计划

![4](/img/blog/SCR-20220831-xbg.png)

2. 然后点击右上角，选择构造计划模板，我们这里选择自定义模板，按照文章最下面配置即可

![5](/img/blog/SCR-20220901-13.png)


3. 选择文本编辑器开始进行流程配置前，我们需要先定义一个${IMAGE_NAME}变量。

![6](/img/blog/SCR-20220901-6y.png)

4. 然后新建一个制品库，就是存放 docker 镜像的地方

![7](/img/blog/SCR-20220901-87.png)

5. 点击仓库管理，配置访问令牌

![8](/img/blog/SCR-20220901-9t.png)

6. 然后复制仓库中的登录和推送命令配置到我们构造计划文件中，我们这里选择使用令牌的方式进行登录，点击这个按钮，复制生成的令牌。替换刚才的登录命令

7. 由于登录密码是敏感的，我们按照第一次创建环境变量的方式再次新增一个变量 ${DOCKER_TOKEN} 替换登录密码

8. 上面我们注意有个 credentialsId，这个是 coding 能登录我们服务器的关键。这个东西会在 condig 读取秘钥，然后使用秘钥登录我们的服务器进行操作。

+ 配置好服务器ssh，确保能连接上去。
+ 生成密钥
 ```sh
  ssh-keygen -m PEM -t rsa -b 4096
  cd .ssh
  cat id_rsa.pub >> authorized_keys
  chmod 600 authorized_keys
  chmod 700 ~/.ssh
  vim /etc/ssh/sshd_config
  ##修改配置
  RSAAuthentication yes
  PubkeyAuthentication yes
  PermitRootLogin yes
  PasswordAuthentication no
  ##重启
  service sshd restart
```
+ 点击左下角的项目设置，开发者选项-凭据管理，录入凭据，就是你服务器IP，ssh私钥

![9](/img/blog/SCR-20220901-oa.png)

9. 最后我们还需要配置触发规则，也就是什么情况执行部署。比如推送到 master，或者推送新标签后触发。

![10](/img/blog/SCR-20220901-114.png)

10. 由于自动部署需要访问我们的 docker 镜像仓库。所以我们需要把执行构建的服务器 IP 添加到 docker 镜像仓库的白名单里面。

![11](/img/blog/SCR-20220901-130.png)

+ 点击右上角头像，个人账号设置，访问令牌, 编辑访问令牌，把ip添加进去（由于隐私问题，就不截图了）

![12](/img/blog/SCR-20220901-15o.png)

## 自动部署

把下面的配置好，复制到流程配置里的文本编辑器，配置好环境变量即可
+ 注意配置IMAGE_NAME、DOCKER_TOKEN、CID

![13](/img/blog/SCR-20220901-18p.png)

### 完整版 Jenkinsfile
```sh
pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: GIT_BUILD_REF]],
          userRemoteConfigs: [[
            url: GIT_REPO_URL,
            credentialsId: CREDENTIALS_ID
          ]]])
        }
      }
      stage('生成镜像') {
        steps {
          echo '生成镜像中……'
          sh 'ls'
          sh 'docker build -t ${IMAGE_NAME} -f Dockerfile ./'
          echo '生成镜像完成'
        }
      }
      stage('推送镜像') {
        steps {
          echo '推送镜像中...'
          sh 'docker login -u hyperf-xxxx -p ${DOCKER_TOKEN} xxxx.pkg.coding.net'  ## 你的访问令牌的登录命令，对照制品仓库的操作指引
          sh 'docker tag ${IMAGE_NAME} xxxx-docker.pkg.coding.net/hyperf/hyperf/${IMAGE_NAME}'  ## 本地镜像打标签
          sh 'docker push xxxx-docker.pkg.coding.net/hyperf/hyperf/${IMAGE_NAME}'  ## 命令进行推送
          echo '推送完成'
        }
      }
      stage('部署') {
        steps {
          echo '部署中...'
          script {
            def remote = [:]
            remote.name = 'my-server'
            remote.allowAnyHosts = true
            // 主机地址
            remote.host = '1.1.1.1' ## 你的服务器ip
            remote.port = 22
            // 用户名
            remote.user = 'root'
            // credentialsId: coding登录主机的秘钥
            withCredentials([sshUserPrivateKey(credentialsId: "${CID}", keyFileVariable: 'id_rsa')]) {
              remote.identityFile = id_rsa

              ## 下面的dokcer指令跟你的自己仓库命令配置

              // 登录并拉取镜像
              sshCommand remote: remote, command: "docker login -u hyperf-xxxx -p ${DOCKER_TOKEN} xxxx-docker.pkg.coding.net"
              sshCommand remote: remote, command: "docker pull xxxx-docker.pkg.coding.net/hyperfhyperf/hyperf/${IMAGE_NAME}:latest"

              // 停止旧的服务，注意加 ||true，不然第一次部署会报错
              sshCommand remote: remote, command: "docker stop ${PROJECT_NAME} || true"
              sshCommand remote: remote, command: "docker rm ${PROJECT_NAME} || true"

              // 启动新服务
              sshCommand remote: remote, command: "docker run -d --restart always -p 9501:9501 -v /www/go_pocket_api.env:/opt/www/.env --name ${PROJECT_NAME} -d xxxx-docker.pkg.coding.net/hyperf/hyperf/${IMAGE_NAME}:latest"

              // 再启动一个容器，防止服务中断
              sshCommand remote: remote, command: "docker stop ${PROJECT_NAME}2 || true"
              sshCommand remote: remote, command: "docker rm ${PROJECT_NAME}2 || true"
              sshCommand remote: remote, command: "docker run -d --restart always -p 9502:9501 -v /www/hyperf.env:/opt/www/.env --name ${PROJECT_NAME}2 -d xxxx-docker.pkg.coding.net/hyperf/hyperf/${IMAGE_NAME}:latest"
            }
          }
          echo '部署完成'
        }
      }
    }
  }

```

### 然后我们本地修改代码，推送到 master 或者推送标签根据自己设置的触发规则来即可
![14](/img/blog/SCR-20220901-1bq.png)


## 总结
到此为止就会自动部署到你的服务器上，就可以使用了。有什么不懂得可以联系作者本人。