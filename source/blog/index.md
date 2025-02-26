---
title: Hexo+Next部署博客
date: 2025-02-24 17:31:09
tags: hexo
---

使用hexo + next在Github Page免费托管个人博客

### 第1步 本地博客项目生成
* 安装Node.js [https://nodejs.org/zh-cn]

* 使用npm安装hexo [https://hexo.io/zh-cn/]

```sh
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```
* 完成后初始化后，您的项目文件夹将如下所示

```sh
├── <code>_config.yml</code>  // 配置文件
├── package.json
├── scaffolds
├── source      // 页面
|   ├── _drafts
|   └── _posts
└── themes      // 主题
```

### 第2步 创建github项目

* 打开[Github](https://github.com/dashboard)
* 点击新增创建新项目
![alt text](/images/hexo/new.png)

* 想一个名字比如blog，然后选择公开
![alt text](/images/hexo/image-1.png)

### 第3步 本地博客配置

* 安装Git部署插件
```sh
npm install hexo-deployer-git --save
```
编辑根目录 <code>_config.yml</code> （示例值如下所示）：
```sh
deploy:
  type: git
  repo: <repository url> # 第二步创建的git项目克隆地址，示例：https://github.com/nealHong/blog.git
  branch: [branch]   # 发布分支
  message: [message] # 非必填
```
* 配置域名

编辑根目录 <code>_config.yml</code> （示例值如下所示）：
```sh
url: <repository url> # github page地址，可查看下面的截图 示例：https://nealhong.github.io/blog/
```
github进入项目设置，就能看到对应的网页地址
![alt text](/images/hexo/image-2.png)

* 安装主题，示例使用[next主题](https://hexo-next.readthedocs.io/zh-cn/latest/next/base/%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86/)
```sh
git clone https://github.com/theme-next/hexo-theme-next themes/next
```
* 配置主题
编辑根目录 <code>_config.yml</code> （示例值如下所示）：
```sh
theme: next
```
* 主题设置

Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme，他们是：
> Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
> Mist - Muse 的紧凑版本，整洁有序的单栏外观
> Pisces - 双栏 Scheme，小家碧玉似的清新(当前博客是此主题)

编辑<code>themes/next</code>目录 <code>_config.yml</code> （示例值如下所示）：
```sh
#scheme: Muse
#scheme: Mist
scheme: Pisces
```

* 菜单设置
  编辑<code>themes/next</code>目录 <code>_config.yml</code> （示例值如下所示）：
```sh
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  #tags: /tags/ || fa fa-tags
  #categories: /categories/ || fa fa-th
  #archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```
> 菜单图标使用[Fontawesome](https://fontawesome.com/start)

* 示例about页面
在根目录<code>source</code>文件夹下生成文件夹<code>about</code>，菜单就会显示about，点击时会读取文件
```sh
├── source      // 页面
|   ├── about
|   |   ├── index.md
```
* 其他打赏&语言&布局等配置可查看文档

### 第4步 部署
```sh
# 清除
npm run clean
# 打包
npm run build
# 部署发布
npm run deploy
```
执行之后即可访问github page的网址进行访问，示例：https://nealhong.github.io/blog/
