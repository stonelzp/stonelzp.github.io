---
title: zsh找不到gulp工具包
date: 2018-05-30 22:56:07
categories:
- 工具
tags:
- 笔记
---
不能上网，一上网问题就接踵而来。今天突然看到了hexo博文压缩这个功能，就想着这个可以有，就按照教程安装了gult。安装完运行`hexo g && gulp`之后准备舒舒服服的打包的时候，我一向视同己出的zsh弹出来了这个提示
```bash
zsh: command not found: gulp
```
<!--more-->

刚装的你跟我说找不到？我信了你的邪。

自己解决应该是有点难度了，只能借用大家的智慧了。在网上搜到了这两篇博文[COMMAND NOT FOUND
WITH A NODE MODULE (NPM) SOLUTION](http://blog.webbb.be/command-not-found-node-npm/)与[【gulp】zsh: command not found: gulp!!!「急にgulpが壊れた!」と思ったら読む記事](http://kenjimorita.jp/gulp-zsh-command-not-found-gulp/)完美的解决了我的问题。

原因可以从下面的命令中看出来

```zsh
➜  blog npm root
/Users/stone/Documents/mynote/BLOG/blog/node_modules
➜  blog npm root -g
/Users/stone/node_modules
```
gulp被安装到了个人文件夹中去而不是NPM命令的全局文件夹。

解决办法是运行下面的命令
```zsh
npm config set prefix /usr/local
```
再运行一次`npm root -g`应该就会看到执行后的结果变化
```zsh
➜  blog npm root -g
/usr/local/lib/node_modules
```
然后再次安装gulp，应该是全局的安装
```zsh
npm i -g gulp
```
确认gulp的版本
```zsh
➜  blog gulp -v
[23:25:50] CLI version 3.9.1
[23:25:50] Local version 3.9.1
```
出现了一个CLI版本跟一个本地的版本。嘛，反正是好用了。
