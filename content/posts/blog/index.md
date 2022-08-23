---
title: "搭建hugo博客记录"
date: 2022-08-24T01:22:44+08:00
draft: true
toc: false
images:
tags: [              
    "blog",
]
---

## Hugo简介

`Hugo`是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

## Hugo使用

### 安装Hugo

由于我用的是`mac`系统，直接使用`brew`，其他系统参考[Hugo中文文档](https://www.gohugo.org/)

```shell
brew install hugo
```

检查是否安装成功

```bash
hugo version
```

然后输出

```bash
hugo v0.92.0+extended darwin/arm64 BuildDate=unknown
```

就说明安装成功了

### 创建blog站点

在当前目录执行命令创建blog站点

```bash
hugo new site blog
```

这个`blog`就是项目的名字了，创建的目录如下

```
├── archetypes
│   └── default.md
├── config.toml         # 博客站点的配置文件
├── content             # 博客文章所在目录
├── data                
├── layouts             # 网站布局
├── static              # 一些静态内容
└── themes              # 博客主题
```

我们的博客文章就放在`content`目录下的`posts`中，只需要按照Markdown格式编写，hugo就会读取到文章然后展示在博客中。

### 安装主题

我这里用的是`hermit`主题，开发者是[Track3](https://ojbk.im/)

安装依次执行以下命令：

```bash
cd myblog 
git clone https://github.com/Track3/hermit.git ./themes/hermit
```

### 使用主题

将`hermit`主题中`exampleSite`目录下的内容拷贝到当前目录`blog`下

可以通过修改`config.toml`文件来更改配置

贴上我的`config.toml`文件配置，是抄了煎鱼佬(#^.^#)

```toml
baseURL = "http://52yaya.cn"
languageCode = "zh-hans"
defaultContentLanguage = "en"
title = "很高兴与你相遇！"
theme = ["hermit"]
# enableGitInfo = true
pygmentsCodefences  = true
pygmentsUseClasses  = true
# hasCJKLanguage = true  # If Chinese/Japanese/Korean is your main content language, enable this to make wordCount works right.
rssLimit = 10  # Maximum number of items in the RSS feed.
copyright = "This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License." # This message is only used by the RSS template.
enableEmoji = true  # Shorthand emojis in content files - https://gohugo.io/functions/emojify/
googleAnalytics = "UA-166045776-1"
# disqusShortname = "yourdiscussshortname"
buildFuture = true

[author]
  name = "Errorist"

[blackfriday]
extensionsmask = ["noIntraEmphasis"]
fractions = false
  # hrefTargetBlank = true
  # noreferrerLinks = true
  # nofollowLinks = true

[markup.goldmark.renderer]
unsafe = true

[taxonomies]
  tag = "tags"
  # Categories are disabled by default.

[params]
  since = "2018"
  toc = true
 usemermaid=true

 customCSS = ["css/a.css"]
  
  dateform        = "Jan 2, 2006"
  dateformShort   = "Jan 2"
  dateformNum     = "2006-01-02"
  dateformNumTime = "2006-01-02 15:04 -0700"

  # Metadata mostly used in document's head
  # description = ""
  # images = [""]

  homeSubtitle = "Coding, Thinking, Life"
  footerCopyright = ' &#183; <a href="http://beian.miit.gov.cn/">浙ICP备2020045357号</a>'
  # bgImg = ""  # Homepage background-image URL

  # Prefix of link to the git commit detail page. GitInfo must be enabled.
  # gitUrl = "https://github.com/username/repository/commit/"

  # Toggling this option needs to rebuild SCSS, requires Hugo extended version
  justifyContent = false  # Set "text-align: justify" to `.content`.

  relatedPosts = false  # Add a related content section to all single posts page

  code_copy_button = true # Turn on/off the code-copy-button for code-fields
  
  # Add custom css
  # customCSS = ["css/foo.css", "css/bar.css"]

  # Social Icons
  # Check https://github.com/Track3/hermit#social-icons for more info.


  [[params.social]]
    name = "github"
    url = "https://github.com/ErroristZ"

	
  [[params.social]]
    name = "email"
    url = "zhangkangweiruan@gmail.com"

[params.utteranc]
  enable = true
  repo = "Errorist/blog" 
  issueTerm = "pathname"
  theme = "github-light"

[menu]

  [[menu.main]]
    name = "文章"
    url = "posts/"
    weight = 10
	
  [[menu.main]]
    name = "标签"
    url = "tags/"
    weight = 10

  [[menu.main]]
    name = "关于"
    url = "about/"
    weight = 20

```

设置好配置文件后在`blog`目录下执行`hugo`命令即可生成`public`文件夹，这个文件夹就是我们站点的根目录文件夹，后面nginx中部署时指定的根目录也是这个。如果想使用`github pages`只要将这个目录放在`github`托管，每次改完提交即可。

```shell
hugo
```

## 使用Nginx部署

### 安装Nginx

使用docker安装nginx，步骤略过。

### 配置Nginx

修改配置文件

```shell
cd /etc/nginx/conf.d
vi web.conf
```

配置nginx。

```sh
  server {
      listen       80;
      server_name  52yaya.cn;

      root /www/dist;

      location / {
          try_files $uri $uri/ /index.html;
      }

      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  }
```

按下Esc键，并输入`:wq`保存退出文件。

### 启动Nginx

运行docker以下命令重启Nginx服务。

```shell
docker nginx restart
```