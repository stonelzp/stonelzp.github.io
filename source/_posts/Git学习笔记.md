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

### 删除远程分支
多人一起工作的时候就会经常遇到需要删除自己提交的远程分支的情况，在这里做一下记录。

首先先看一下自己本地的分支和远程的分支：
```
git branch -a
```
在确认了想要删除的分支的名字之后首先执行本地的分支删除操作吧：
```
git brance -d Branch-Name
```
如果没有什么问题本地的分支就会被删除。下一步是删除远程的分支
```
git push -d origin Branch-Name
```
这样就完成了，上面的`-d`跟`--delete`是等效的。

参考文章：
- [Git で不要になったローカルブランチ・リモートブランチを削除する方法](https://qiita.com/iorionda/items/c7e0aca399371068a9b8)

## SourceTree
最近一直在用这个工具，很方便了，代替Git吧，但是遇到了问题。

在SourceTree中使用Git LFS的时候，文件超过一定大小当你准备提交的时候就会提醒你把这个文件交给Git LFS来处理，于是我照办了。由于这个文件的路径有些奇葩，有`space`的存在，于是我明明在`.gitattribute`文件中添加了路径，SourceTree还是会提醒我这个文件要用Git LFS来处理。

文件路径奇葩带空格又不是我的错，我只是导入一个插件而已。至于解决方案，那就是使用命令行工具来提交啊。关键时候还得是Bash。

参考了下面的文章：
- [[[:space:]] seam to prevent Git lfs to work with SourceTree](https://github.com/git-lfs/git-lfs/issues/2668)

## Git LFS 2
在UE4中可以使用这个插件来完成项目的多人协作，割舍不下的功能就是在UE4中对Asset进行编辑时可以远程加锁，UE4的Asset版本管理很麻烦的，又不能像C++代码那样简单的Merge。
- Github项目[Unreal Engine 4 Git Source Control Plugin](https://github.com/SRombauts/UE4GitPlugin)

### Git LFS
这东西用于管理大数据文件很有用，使用的时候也会遇到很多问题，当然是我的问题，上面SourceTree的内容就有提到。

关于一些命令的使用：
```
$ git lfs locks             #获取现在加锁文件的一览
$ git lfs unlock --id=555   #为555号id文件解锁，当然是自己加锁的文件
```

还有就是一些批量操作默认的GitLFS命令并不支持，这也是我要在这里记录这个的原因。

在`ProjectName/.git/config`文件里面添加下面的内容
```
[alias]
        lock = "!f(){ git pull ; for file in \"$@\"; do git lfs lock $file; done };f"
        unlock = "!f(){ for file in \"$@\"; do git lfs unlock $file; done };f"
        unlockall = "!f() { git lfs locks | grep [my-git-user-name] | cut -f 1 | while read -r file; do git lfs unlock $file; done; };f"
        locks = "!f() { git lfs locks; };f"
```
这里的`[my-git-user-name]`根据自己的账户名字进行替换，使用`git lfs locks`就可以看到自己加锁的账户名字，`[]`是必须的，不要省略。

这样在使用下面的命令的时候，就会把自己加锁的文件全部解锁。
```
$ git unlockall
```

`git unlock a.txt b.txt`和`git lock a.txt b.txt`这样可以对文件进行批量的加锁解锁操作，但还是需要指定路径，也许可以加以改造，指定id就可以简单的批量解锁文件，但是我也只是参照别人的文章复制过来的，对`.git/config`文件的内容语法也不是很了解。

参考文章:
- [Git LFS lockを使うためのエイリアス メモ](https://qiita.com/gecko655/items/89085bad77fb83c98267)
