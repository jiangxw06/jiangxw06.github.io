---

categories: meta

layout: post
title:  "用Github，Jekyll，Markdown撰写技术博客"
---

​    这套技术博客写作环境有令人着迷的优点：

1. 发布简单。无须大动干戈地租用服务器，Github在管理你的源代码之余就顺手管理了你的个人/项目博客。更新博客像上传代码一样简单。
2. 页面优雅。没有广告，没有插件。实际上，Jekyll使得博客中一切可见的元素都是可定制的。默认主题和Github浑然一体。
3. 写作方便。使用Markdown编辑器，比如Typora，让你以自然语言撰写html，而不用担心排版问题。

​    缺点是有一定的技术门槛。

​    Jekyll是用Ruby语言实现的，而且官方不支持Windows版本。在Linux/Mac环境下搭建Jekyll环境最好能有一点Ruby包管理的基本知识。

​    如果你以前没有使用过Ruby，那么在使用Ruby的包管理工具下载Jekyll的过程中可能会遇到“gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer……”的错误。这是因为Ruby的官方网站[rubygems.org](https://rubygems.org)的Gem资源放在目前被墙了的Amazon S3上的缘故。在终端的任意目录下输入以下命令：

```
$ gem sources --remove https://rubygems.org/
$ gem sources -a https:/ruby.taobao.org/
```

用淘宝的镜像网站代替Ruby官方网站即可。
