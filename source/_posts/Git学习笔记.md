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

### 添加远程分支
为初始的仓库添加远程分支。
```
git remote add origin git-url
```

删除远程分支
```
git remote rm git-url
```

修改远程分支
```
git remote set-url origin git-url
```
也是修改远程分支的指令。

修改仓库名称
```
git remote rename name-before name-after
```

顺便一提查看远程分支的指令：
```
git remote -v
```
还有查看远程ping得通得指令
```
git ls-remote
```
- [git "ping": check if remote repository exists](https://superuser.com/questions/227509/git-ping-check-if-remote-repository-exists)


要说为什么想起来写这个部分的内容，公司的一个Gitlab的仓库我是死活连接不上，原因则是用的ssh的路径而不是http的路径。
话说回来，我也是不太清楚ssh和http路径在gitlab中使用的区别，至少我目前遇到的都是ssh路径不行的情况下http就可以，而SourceTree中使用的基本上都是http路径。

### Add操作
```
git add .
```
这个算是最常用的命令之一了，这个命令的含义是将当前文件夹下的所有的变动（**增删改**）都执行add操作。

无关当前文件夹，当前Repo里所有的变动都执行add操作：
```
git add -A
// or
git add --all
```

无关当前文件夹，所有变动（只有**删改**）执行add操作：
```
git add -u
// or
git add --update
```

参考文章
-[git add -u と git add -A と git add . の違い](https://note.nkmk.me/git-add-u-a-period/)


### 取消已经add的操作
有的时候习惯性就会在commit之前add，真正准备commit的时候才发现有不想提交的内容。
结论就是使用这个
```
git rm --cached .
// or
git rm --cached <added_file_to_undo>
```
或者需要递归操作的话用
```
git rm -r --cached .
```
这会清除所有add的内容。

在执行add的时候推荐用这个
```
git add -n .
```

参考资料：
- [How do I undo 'git add' before commit?](https://stackoverflow.com/questions/348170/how-do-i-undo-git-add-before-commit)
回答里的最高赞内容，只能说是跟我的情况一模一样了。

### Git Stash
可以一时间保存本地的修改，在切换分支的时候也可以保留自己的修改。

指定文件进行stash操作：
```
// 指定文件进行stash操作
$ git stash push -- <filepath>

// 指定stash的id进行复原
$ git stash pop stash@{0}
```

参考：
- [[Git] ファイル単位でstashする](https://qiita.com/yukitaka13-1110/items/935bfa233c6e024f82dc)

这里再稍微罗嗦一下：
```
// Stash当前所有的修改
git stash

// 查看stash列表
git stash list

// 应该是选择最上面的那样stash进行复原（我没有试过，需要验证的）
git stash pop
```

### Git clean
```
git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <path>…​
```
各个参数的含义：
- `-d` : 版本管理外的文件包含文件夹都会以递归的方式
- `-f` : `--force`，没有这个参数的话，文件以及文件夹不会被删除。
- `-x` : 版本管理外的文件包含文件夹都会被删除
- `-n` : `--dry-run`,输出删除对象，但不会真正的删除文件
- `-i` : `--interactive`,可以一个一个确认删除文件
- `-q` : 安静的删除文件，不会输出删除日志，除非出错误日志
- `-X` : 手动添加的文件可以不会被删？（没有用过）

一般的情况下我都是使用`git clean -df`或者`git clean -xdf`。

参考资料:
- [git clean](https://www-creators.com/git-command/git-clean)

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

#### 基本使用
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


#### 进阶使用
```
git lfs prune
```
当项目发展到一定规模之后，使用GitLFS自己的仓库空间就会越来越大，电脑空间再大早晚就会被吃空，一个T两个T的SSD都被吃的干干净净，看着变红的硬盘，再看看空空的钱包，还是想想技术吧。

- [【Git】Git LFSを使用したリポジトリが肥大化した場合に使いたい「git lfs prune」](https://zenn.dev/rossam/articles/73d4f14d368caf02fd9a)

调查的时候发现一篇文章稍微有些在意，先记下：
- [Gitリポジトリをメンテナンスして軽量化する](https://qiita.com/kaneshin/items/0d19fc1cd86f931dc855)
