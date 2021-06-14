---
title: First blog
date: 2021-06-11 14:33:44
tags: [hexo]
---


## 使用的工具

> Hexo + Github

需要下载：
1. npm
2. Git

## 操作步骤

首先要有一个GitHub账号，建立一个Repository，名称为
~~~
yourname.github.io
~~~

这样博客的网站就是http://YourName.github.io

之后在电脑中创建一个存放博客文件的文件夹，在该目录下面打开Git Bash,输入`npm i hexo-cli g` 安装hexo

验证：`hexo -v` 

初始化： `hexo init`

`npm install` 安装必备的组件

`hexo g` 生成静态网页

`hexo s` 打开本地服务器

在浏览器中输入localhost:4000，即可预览效果，注意端口不要被占用了

### 连接Github与Hexo

在Git Bash中输入
~~~
git config --global user.name "YourName"
git config --global user.email "YourEmail"
~~~

配置SSH Key以便不用输入用户名和密码

生成SSH Key密钥：
~~~
ssh-keygen -t rsa -C "YourEmail"
cat ~/.ssh/id_rsa.pub
~~~

能够查看到已经生成的SSH 

在Github中Settings中点击SSH and GPG Keys设置，将刚才输出的内容复制在Key中

输入`ssh -T git@github.com`，如果成功出现用户名，那就成功了。

### 修改Hexo配置文件

Hexo有很多主题可以选择，可以在_config.yml文件中修改Theme

为了连接到Github需要修改deploy部分：
~~~
deploy:
  type: git
  repository: git@github.com:YourName/YourName.github.io.git
  branch: master
~~~

### 写文章与发布

需要安装一个扩展：
~~~
npm i hexo-deployer-git
~~~

新建一个.md文章
~~~
hexo new post "article title"
~~~
文章目录存放在`source\_posts`中

文章写完之后
~~~
hexo g && hexo d
~~~
进行网页的生成和提交


## 常用Hexo命令
~~~
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本

hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy

hexo s -g #生成并本地预览
hexo d -g #生成并上传
~~~