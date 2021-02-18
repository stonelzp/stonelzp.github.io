---
title: Markdown文件转换为PDF文件
date: 2018-05-29 22:56:35
categories:
- 工具
tags:
- 笔记
- Markdown
- 碎碎念
---
最近闲来无事(实际上由于跳槽面试的原因忙的要死)，在知乎上看到了一篇搭建博客的文章，看了一眼发现对于我这种WEB盲还挺友好，于是就自己搭建了这个博客。

就在ZH上捞鱼捞的正爽的时候，面试(未)公司发来了要看看我的作品的消息。我一看表现的时候到了，就掏出我刚学的Markdown来写了几篇我的作品的说明文。正当我想把刚build好的html文件打包压缩发送过去的时候，(自动脑补柯南的灵光一闪音效)我发觉事情并不是这么简单。

<!--more-->

要是我作死直接发了一堆html过去，估计就别想见到人家了，就算见到了人家公司的HR，估计见面也有不小的概率会被锤。这个时候慌乱的我,手不由自主的打开了谷歌并输入了:How to convert markdown to pdf.然后发现正确的提问用法是:How to convert from markdown to pdf.

(；´∀｀)嘛，英语不好这得批评，早晚得去考一次托福...

搜到的第一个是说用**谷歌上的插件**:[Markdown Preview Plus - Chrome Webstore](https://chrome.google.com/webstore/detail/markdown-preview-plus/febilkbfcbhebfnokafefeacimjdckgl?hl=ja).ドラッグアンドドロップ就可以转换了。什么？ドラ...什么的，打开谷歌翻译:drag and drop.我...

但是利用网上的插件总感觉有些难受，还要上传文件什么的，万一上传到人家服务器上被人家看到了里面的内容就不好了(WEB盲)。于是看到了第二个方法：使用**Node.js的工具**[markdown-pdf](https://github.com/alanshaw/markdown-pdf)

```bash
npm -g install markdown-pdf
```

安装好工具之后

```bash
markdown-pdf 我的说明文.md
```

就能生成想要的PDF文件了。浏览生成的PDF文件的时候唯一在意的就是Markdown给隐藏起来的URL链接PDF文件也一并显示了出来。难不成是制作人有意这样，亦或者是机关在作祟，我就暂时不得而知了。因为现在的我得等人家的面试通知。

唉...寝よう
