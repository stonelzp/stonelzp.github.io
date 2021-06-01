---
title: Hexo+Next博客修缮笔记
date: 2019-05-19 17:37:35
updated: 2019-05-19 17:37:35
permalink: Hexo+Next-maintain-myblog-note

tags:
- Markdown
- 笔记

categories:
- 备忘

description: 今后准备认真的做好自己Blog的维护，于是记下一些平常虽然想用但是嫌麻烦的使用方法，以备自己随时查找。为了自己今后能更好的自定义自己的博客内容和格式。

---

自从发现自己写的东西被人原封不动的复制粘贴扔到了CSDN之后，曾经一度想要废了这个博客，写博客对我来说就是一个记笔记的过程，记录自己工作中遇到的问题和解决过程，顺便写写自己的心得什么的，就是很随意。未想着给别人看，但是要是写的某一个地方帮上了某个地方的某个人也是好事情，哪怕不知道有没有，但也是一种安慰了。所以博客自建成起，就根本没设置什么自欺欺人的评论功能点赞功能。也没想着会有。

但是整篇文章直接贴走，连个转载链接都不留是不是就有些过分了，生气不可避免了但是也只是只能无能狂怒了。去CSDN注册个账户举报？说到底对于我去CSDN注册账户这件事情都很抵触。算了，就算了吧。

<!--more-->

# 关于博客内容书写
- 2020/11/20 : 对新建文章的模板进行了修改
- 2021/02/19 : 把博客的配置及源文件上传到了Github仓库分支方便多台PC整理博客
- 2021/03/21 : 对新电脑上的博客重新配置遇到了好多问题


## 代码部分

果然UE4中的有些代码不会给你高亮，看起来就很不舒服，有时间填一下坑。
# 关于博客构建

## 博文压缩
我之前貌似有整过这个叫做`gulp`的东西。而且连`gulpfile.js`这个文件也存在。

因为用起来麻烦就忘了。

执行

`hexo g && gulp`

就会压缩public中的静态资源文件。

## 定义新建文章的模板
因为每次都要手动添加新的项目，心血来潮找了找有没有修改新建文章模板的方法
- [Hexo の 新規投稿テンプレート を カスタマイズ](https://azriton.github.io/2016/11/04/Hexo%E3%81%AE%E6%96%B0%E8%A6%8F%E6%8A%95%E7%A8%BF%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%92%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA/)


## 修改`code`代码块自定义样式
`\themes\next\source\css\_custom\custom.styl`

```
// Custom styles.
code {
    color: #ff7600;
    background: #fbf7f8;
    margin: 2px;
}
// 大代码块的自定义样式
.highlight, pre {
    margin: 5px 0;
    padding: 5px;
    border-radius: 3px;
}
.highlight, code, pre {
    border: 1px solid #d6d6d6;
}
```

我加入了上面的代码，原先是什么都没有的...

看了下效果，嗯，感觉什么都没变。

寝るか

## 转移电脑的博客遇见的大坑
将自己的博客的内容转移到新的电脑遇见了不少的大坑，在这里稍作记录。

首先是如何使用双分支管理博客的源文件和生成的静态网页文件。**SourceBranch**用来管理博客的源文件，**master**则用来上传博客的静态文件。

平常的工作流程一般是先`pull origin SourceBranch`，嘛如果有在两个电脑上进行更新的话。先对源文件及`source/_post`文件夹下面的源文件进行博文的添加和书写。其次是`hexo d -g -m "comment"`对源文件进行镜头网页的生成和网页部署，即生成网页并推送到Github上。最后则是将本地的**SourceBranch**的内容改动commit结束。

前提是自己的部署Deploy文件会默认将部署推送到远程的**master**的分支上。

然后就是这个过程中遇到的大坑。首先是Window上需要运行Hexo的环境构建，Hexo的运行需要NodeJs，但是过高的版本Hexo并不支持。我自己选的版本是nodejs.v12.14版本。太新的版本没有办法将本地生成的网页部署到（推送）远程。安装完后基本上用到的命令就是`npm install`安装所有的node_module(注意要在博客的根目录，也就是这个文件夹所在的目录),`npm install hexo-cli -g`（Hexo不好用的时候重新安装Hexo）。

其次是主题的版本，我用的next的主题但是又不能随便找一个网上的主题，只好沿用自己之前的主题，把之前的电脑的主题文件整个文件夹压缩转移到了新电脑，然后运行`hexo clean`重新配置生成网页，才解决了网页里出现了不知道是哪国语言的尴尬问题。

---
- 2021/05/05 前来更新

对于上面的问题，由于这之前遇到了博客界面格式没有了的问题，迫不得已逐个解决了。首先是上面的next主题文件的语言问题，主要是中文的解析没对上的原因。具体的解决问题的操作的是去next主题的language文件夹中找到自己在博客的整体`_config.yml`文件中设置的语言是否存在。

我上面网页中出现了不知道哪国语言的问题，经过查看之后发现,`_config.yml`文件中设置的语言是**zh-Hans**,而主题中的语言没有这个，解决方案是把其中的`zh-CN.yml`重命名为`zh-Hans.yml`，文字的问题就解决了。

参考文章：
- [Hexo语言不生效问题](https://blog.csdn.net/science_Lee/article/details/84633237)

然后就是格式的问题，我在本地运行的环境(`hexo s`)中博客的格式都会完美的显示，但是部署到Github之后格式就没有了，尝试了很多，最后是从CSS文件找不到路径的问题找到了答案。</br>
原因则是网页的URL设置有些问题：
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://stonelzp.github.io
root: /
```
修改之后的结果。

参考链接:
- [生成路径的问题，导致css，js无法加载](https://github.com/hexojs/hexo/issues/1121)

---
- 2021/06/01 更新

博客换了电脑之后，主题虽然没变但是好多设定都丢了，包含我用的很顺手的本地搜索。实在是太常用了，没有的话很困扰，就前来解决了。

需要检查的有两处：
  1. hexo-generator-search插件是否安装
  2. 本地搜索是否开启

第一个直接搜命令使用npm安装就好。
```
$ npm install hexo-generator-search --save
```
第二个去主题的配置文件找到**local_search**条目
```
# Local search
local_search:
  enable: true
```
