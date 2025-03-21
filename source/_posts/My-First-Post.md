---
title: 使用 Hexo 项目并在 GitHub 上部署指南
date:  2022-03-02 23:10:35
---

# 使用 Hexo 项目并在 GitHub 上部署指南

## Hexo 简介
Hexo 是一个快速、简洁且高效的博客框架，基于 Node.js 开发，适用于个人和团队博客的搭建。

## 我为什么选择Hexo,作为博客框架的搭建
原因：近期在使用air2软件的服务中，相关bt离线服务无法正常下载，在google搜索后发现了一个对该问题分析原因的网站[网站地址]（https://p3terx.com/ "p3terx"），点了点，加载响应超级快，体验感觉非常好，我就希望我的博客主站就用这个来做吧，那么是怎么做呢，先通过openai的chatgpt 进行交互，了解到网站是由Hexo 及butterfly主题来生成的。那么我决定来搭建一个属于个人的博客网站。

## 步骤一：安装 Hexo

1. **前置：本地服务一定有 Node.js 和 npm**
   - 访问 [Node.js 官网](https://nodejs.org/) 下载并安装最新版本的 Node.js。
   - npm 会随 Node.js 一同安装。

2. **安装 Hexo**
    - npm install -g hexo-cli
    - hexo init myblog  

3.  **hexo的一些指令及文件格式介绍**
   ``` bash
      $ hexo init #命令用于初始化一个本地文件夹为网站的根目录
      $ hexo new title #新建一篇文章
      $ hexo generate #可以简写成 hexo g该命令用于生成静态文件
      $ hexo server #命令用于启动本地服务器，一般可以简写成 hexo s
         #可以加一些参数
            -p    #选项 ，指定服务器端口，默认为 4000
            -i    #选项，指定服务器 IP 地址，默认为 0.0.0.0
            -s    #选项，静态模式 ，仅提供 public 文件夹中的文件并禁用文件监视
      $ hexo deploy #命令用于部署网站，一般可以简写成 hexo d (这个基本就是推送部署到指定位置)
      $ hexo clean #命令用于清理缓存文件，是一个比较常用的命令
      $ hexo --safe #表示安全模式，用于禁用加载插件和脚本
      $ hexo --debug #表示调试模式，用于将消息详细记录到终端和 debug.log 文件
      $ hexo --silent  #表示静默模式，用于静默输出到终端
      ```

## 步骤二：配置主题与文件目录

   1. **Butterfly主题下载**
      -项目根目录下安装 Butterfly 主题
         ``` bash
         npm install hexo-theme-butterfly
         ```
      -配置文件
       >hexo配置文件
         -项目根目录：_config.yml
         -找到 theme 配置项，将其值改为 butterfly：
         ``` bash
         theme: butterfly
         ```
   2. **Butterfly 主题配置**
      -Butterfly 主题有自己的配置文件 _config.butterfly.yml，通常在 themes/butterfly/ 目录下。
      > _config.butterfly.yml
         -网站基本信息：如标题、描述、语言等。
         -菜单和导航栏：定义导航栏中的菜单项。
         -外观：如主题颜色、字体、背景图等。
         -插件和小工具：如搜索功能、标签云、分类、最近文章等。

### 撰写文章 ###
   1. **Hexo 使用 Markdown 格式来撰写文章。**
      - 手动创建：在 source/_posts/ 目录下，1.可以创建一个新的 Markdown 文件：**.md
      - 自动创建：
         ``` bash
         $ hexo new todaynode #新建一篇文章
         ```

## 步骤三：hexo部署：配置 Git 
   1. **在 GitHub 上创建仓库主站。**
      - 登录 GitHub，创建一个新的空仓库，例如 username.github.io（其中 username 是你的 GitHub 用户名）。

   2. **hexo 部署**
      ``` bash
         $ hexo g  #该命令用于生成静态文件  public 
      ```
      - 在 Hexo 项目根目录下，安装 hexo-deployer-git 插件
      - 配置 _config.yml 文件，添加部署设置：
         ``` bash
         $ deploy:
            type: git
            repo: https://github.com/your-username/your-repo.git
            branch: main
          ```
      -  配置 Git
         >使用 SSH 密钥认证：
            生成 SSH 密钥：如果你还没有 SSH 密钥，可以使用以下命令生成：
            ``` bash
            #复制代码
            ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
            #按照提示一路回车，生成密钥对。
            #添加 SSH 密钥到 GitHub：
            #复制你的 SSH 公钥：
            cat ~/.ssh/id_rsa.pub  #(ssh 目录)
            #将公钥添加到 GitHub 的 SSH 设置中。
            #登录 GitHub，转到 Settings -> SSH and GPG keys -> New SSH key，将复制的公钥粘贴到 Key 文本框中，然后点击 Add SSH key。
             ```
      -  推送 hexo d 
         ``` bash
         $ hexo d  #会自动推送public目录下的文件到git 
      ```
### 其它GIT相关
   - 使用git 托管工具推送部署到 GitHub Pages
   - Git 基本操作
      >   
          git info   初始化
          git remote add origin git@github.com:wangyaoqi/wangyaoqi.github.io.git        设置远程仓库
          git remote -v  确认 origin 远程仓库
          git add .  #提交更改到本地仓库
          git commit -m "提交说明"   
          git push origin main      #推送本地更改到远程仓库
          git checkout main   #切换分支
          git pull origin main  #拉取分支变更
          git merge my-new-branch  #合并分支
          git merge my-new-branch --allow-unrelated-histories  #强制合并  前提已在主分支下 
          
 