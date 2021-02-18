---
title: Git学习笔记
date: 2019-06-22 14:14:17
permalink: git-learning

categories:
- 工具

tags:
- Git

---
Git的使用尤为重要，只要是做项目开发没有版本管理的话，那就没个玩。但是Git的使用虽然是很简单但是精通的话还是要时间。

<!--more-->

这里就是要记下我在使用Git的过程中遇到的问题和解决方案。像记事本一样。

## Git
- [git-recipes](https://github.com/geeeeeeeeek/git-recipes/wiki)

这是Git的中文Wiki，我还没看过，真是罪过，一定要看！！

- [Flight rules for Git](https://github.com/k88hudson/git-flight-rules/blob/master/README.md)

有中文翻译。顺便一提这个是日语翻译[【永久保存版】Gitのあらゆるトラブルが解決する神ノウハウ集を翻訳した](https://blog.labot.jp/entry/2019/07/01/183204)


## Git使用中遇到的

### 关于换行符
- [気をつけて！Git for Windowsにおける改行コード](https://qiita.com/uggds/items/00a1974ec4f115616580)

### 关于差分（diff）
在提交(add)代码之前，使用差分工具看看自己修改过的内容是一个极好的习惯，确保自己代码修改的地方，防止提交一些自己失误添加的内容。

最近就发生了我提交了迷之代码的尴尬事情，查看历史的提交记录，追溯到我，场面一度非常尴尬。

- [忘れやすい人のための git diff チートシート](https://qiita.com/shibukk/items/8c9362a5bd399b9c56be)


### 关于忽略文件的变动
有的时候会遇见一些Run-Time类型的文件，可以上传一个版本但是每次运行的时候总会变，每次都要commit就很不现实，于是便有了这个问题：

如何停止对一个文件（或者文件夹中的文件）进行版本管理？

找到的方法是：

- [How to stop tracking and ignore changes to a file in Git?](https://stackoverflow.com/questions/936249/how-to-stop-tracking-and-ignore-changes-to-a-file-in-git)

> There are 3 options, you probably want #3
>
> 1. This will keep the local file for you, but will delete it for anyone else when they pull.
>
>    `git rm --cached <file-name>` or `git rm -r --cached <folder-name>`
>
> 2. This is for optimization, like a folder with a large number of files, e.g. SDKs that probably won't ever change. It tells git to stop checking that huge folder every time for changes, locally, since it won't have any. The assume-unchanged index will be reset and file(s) overwritten if there are upstream changes to the file/folder (when you pull).
>
>     `git update-index --assume-unchanged <path-name>`
> 3. This is to tell git you want your own independent version of the file or folder. For instance, you don't want to overwrite (or delete) production/staging config files.
> 
>     `git update-index --skip-worktree <path-name>`
>
> It's important to know that git update-index will not propagate with git, and each user will have to run it independently.

我实际上使用的是第一种方法，但由于我是一个人开发的所以没有验证过是不是别人Pull的时候会删掉我取消Track的文件。

但我感觉我实际上应该使用的是第二种方法，的确是使用了某个SDK，即Plugins，基本上不会再有改动，除非其他人对插件的内容进行了修改。


## SourceTree
最近一直在用这个工具，很方便了，代替Git吧，但是遇到了问题。

在SourceTree中使用Git LFS的时候，文件超过一定大小当你准备提交的时候就会提醒你把这个文件交给Git LFS来处理，于是我照办了。由于这个文件的路径有些奇葩，有`space`的存在，于是我明明在`.gitattribute`文件中添加了路径，SourceTree还是会提醒我这个文件要用Git LFS来处理。

文件路径奇葩带空格又不是我的错，我只是导入一个插件而已。至于解决方案，那就是使用命令行工具来提交啊。关键时候还得是Bash。

参考了下面的文章：
- [[[:space:]] seam to prevent Git lfs to work with SourceTree](https://github.com/git-lfs/git-lfs/issues/2668)

