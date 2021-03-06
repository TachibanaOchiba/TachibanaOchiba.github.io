---
title: 个人博客搭建记录
date: 2019-12-16 12:31:25
tags:
- git
- Hexo
categories:
- 技术宅的日常
---

<!--在此处插入头图-->
{%asset_img header.jpg "Cover"%}

<!--在此处插入概述-->

- 本文记录了搭建这个站点的过程。
- 2020/10/5: 追加如何更新站点

<!--more-->

## 技术要求
### 站点
- 自主域名(<blog.ochiba.io>)
- 维护简单，不容易坏，毕竟不是专业码农(避免直接数据库、直接写网页等神触操作)
- 写作语法简单(WYSIWYG 或markdown等简单的写作方式)
- 成本低廉 (除域名外白嫖上佳)

### 文章
- 可能涉及的内容
    - 代码块
    - iframe (油管，twitter,etc.)
    - 图片
    - 结构化的文本
    
### 用户交互
- 简洁美观
- 没有广告
- 最好支持留言
- https
- 拥有手机和电脑的适应性界面

## 平台选择
- 域名: namecheap
- hosting: GitHub Pages
- 站点框架：hexo

## 操作过程

>-  参考教程: [韦阳的博客](https://godweiyang.com/2018/04/13/hexo-blog/), [hexo 配置教程](https://hexo.io/zh-cn/docs/configuration)
### 主要步骤阐述
- 安装环境
- 注册Github, 并在上面创建用于Github Pages的repo
- 将repo复制到本地，安装hexo
- 写一篇文章，测试是否正常运转
- 绑定自定义域名


### 安装环境
- 需要安装以下环境：
    - hexo基于的node.js
    - 向github上传文件需要的git
- 同时因为网络原因，需要把软件源设置到国内。

#### 安装环境
- node.js下载地址：<https://nodejs.org/en/download/>
- git下载地址：<https://git-scm.com/download/win>

#### 将软件源设置到国内
- win+R 运行`cmd`，打开命令提示符
- 运行`npm config set registry https://registry.npm.taobao.org`

### 注册Github账号并创建repo
#### 注册
- 访问注册页面，按提示操作：[Join Github](https://github.com/join)
#### 创建repo
- 创建一个`public`的repo，名字为`[your_id].github.io`. 本案为`TachibanaOchiba.github.io`. 
    - 如何创建repo，参考:[Create a repo](https://help.github.com/en/github/getting-started-with-github/create-a-repo)
    - 记下repo的URL，之后修改hexo安配置文件时要用。本案为
    - 
### 安装Hexo
#### 安装base
- 新建一个用于hexo的文件夹,本案为`D:\repo\TachibanaOchiba.github.io`
- win+R 运行`cmd`，打开命令提示符
- 转到刚才创建的文件夹
   - `D:`
   - `cd \repo\TachibanaOchiba.github.io`
- 在该文件夹中安装hexo
    - `hexo init`
    - `npm install`
#### 修改配置文件
> 参考：[hexo官方说明](https://hexo.io/zh-cn/docs/configuration)
- 配置文件是`_config.yaml`.
    - 一些必选配置：
        ```
        # Site
        title: [blog名]
        subtitle: [blog描述]
        description: 
        keywords:
        author: [你的名字]
        language: cn
        timezone: Asia/Shanghai
        ```
        ...
        ```
        # URL
        url: [你的域名]
        ```
        ...
        ```
        # Writing
        post_asset_folder: true
        deploy:
        type: git
        repository: [repo的地址]
        branch: master
        ```
#### 生成网站、本地查看效果
- 生成静态网站：`npx hexo g`
- 打开本地hexo服务器查看效果：`npx hexo s`
- 在浏览器中访问`localhost:4000`，如果看到一个没有报错的网页就成功了。在命令窗口中按下`ctrl_+C`，关闭服务器。

### 连接Github与本地
- 在用于hexo的文件夹中打开Git Shell. 打开方法为在资源管理器中，`shift+右键`，点击`Git shell here`.
- 设定git命令：
    ```
    git config --global user.name "[你的Github用户名]"
    git config --global user.email "[你的Github邮箱地址]]"
    ssh-keygen -t rsa -C "[你的Github邮箱地址]"
    cat ~/.ssh/id_rsa.pub
    ```
- 将 `ssh-rsa ... = [你的Github邮箱地址]`这段输出复制到剪贴板。
- 打开浏览器，进入github，在头像下面点击settings，进入`SSH and GPG keys` \ `New SSH keys`菜单。
    - Title: 任意
    - key: `ssh-rsa ... = [你的Github邮箱地址]`
- 回到Git shell。执行`ssh -T git@github.com`. 
    - 出现形如`Hi TachibanaOchiba! You've successfully authenticated, but GitHub does not provide shell access.`的输出即表示成功。

### 博客的日常维护

#### 文章的新建、修改、删除和发布
##### 新建文章
- 执行`npx hexo new post "文章标题"`
- hexo将会自动在`\source\_posts`中创建文章标题为名的一个md文件和一个文件夹。
- 使用一般md语法写作
- 资源文件（如图片、音频等）可以放在与文章名对应的文件夹中。在md文件中的方法是：
-  图片
     - `{%asset_img [图片名] [描述]%}`.此种做法可以使该图在所有页面上都能加载，适合封面图等；
     - `![]([图片名])`.此种做法只在文章页面生效，但语法比较简洁。
- 一般文件：
  - 小文件可以通过`![链接文字](文件名)`的形式增加下载链接。效果预览:
  [下载CPD记录文件](internal_training.pdf)
  - 大文件建议使用OneDrive等网盘链接。
- 推文、油管视频等
  - 如写网页一样加入iframe。
#### 修改文章
- 修改对应md文件即可。
#### 删除文章
- 删除对应的md文件和文件夹即可。
### 修改模板
#### 文章模板
 在`\scaffolds\post.md`是生成文章的模板。可以根据需求改写。
#### 本地编译、上传
- 编译命令为`npx hexo g`.
- 运行`npx hexo s` 在本地查看修改效果。
- 运行`npx hxo d`发布。

### 绑定域名
> 参考文档：
>- [Github官方帮助 - Configuring a subdomain](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)
>- [Github官方帮助 - About GitHub Pages](https://help.github.com/en/github/working-with-github-pages/about-github-pages#types-of-github-pages-sites)
- 在域名DNS商的网站上添加`CNAME记录`。本案中，Host是`blog`,Value是`TachibanaOchiba.github.io`
- 在repo的`setting\GitHub Pages\Custom domain`上填入自己的地址，并勾上`enforce HTTPS`。
- 在博客源文件夹的`\source`子目录下，新建一个名为`CNAME`的空白文本文件，在里面写入自定义地址。

### 备份和恢复

#### 备份

##### 首次备份
-   [从官方链接下载安装Github Desktop](https://desktop.github.com)
-   在你用于博客的repo上新建一个分支，名字随意(本例使用`hexo-source`)，用于存储博客源代码；
-   将该分支设定为默认分支；
-   在Github desktop上，把`hexo-source`clone到本地文件夹`X`；
-   找到博客源代码文件夹，删除所有`.git`文件夹；
-   将博客源代码文件夹里的所有内容复制到`X`；
-   在Github desktop上，把对`hexo-source`的更改推送到github.
##### 后续备份
- 在GitHub desktop上，commit 和push即可。

#### 恢复
- [从官方链接下载安装Github Desktop](https://desktop.github.com)
- 从官方链接下载node.js
- 将前例分支clone到本地文件夹`Y`；
- 打开控制台，转到`Y`文件夹，安装hexo和其他必要的包：
  - `npx npm install hexo-cli`
  - `npx npm install hexo-deployer-git`
  - `npx npm install`

#### 更新
> 定期检查更新，以便保持站点的安全性。
- 检查更新
    - `npm install -g npm-check` 
    - `npm-check`
- 执行更新
    - `npm install -g npm-upgrade`
    - `npm-upgrade`
- 在其他终端fetch之后，记得
    - `npm install`

## References
- [Hexo博客及环境依赖包的正确升级方法](https://hexo.imydl.tech/archives/51612.html)