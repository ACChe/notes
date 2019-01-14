# 快速搭建gohugo博客

为了远离Ruby,努力的学好Go, 搜索到[gohugo](http://gohugo.io)这个框架，于是折腾了一下，把原来的Octopress都删掉搬到gohugo上。

  因为时间的仓促，没有认真的去看好文档，用最简单的流程来搭到github上。其中感谢[Ulric Qin](http://ulricqin.com/post/how-to-use-hugo/)的解答，解决了一些细节问题。

  整个流程是基于官方教程中的[Hosting on Github](http://gohugo.io/tutorials/github-pages-blog/)。首先是要熟悉git的最基本操作，详细可以参照[Github Help](https://help.github.com/).
  最简易的搭建过程如下:


## 1 网站文件结构
```html
▾ <Root>/
	▾ content/
		▾ posts/
	▾ static/
	▾ theme/hyde/
	config.yaml
```
## 2 创建config.yaml文件
```html
---
baseurl: "http://your.github.io/hugo_blog"
title: "Hugo Blog Template for GitHub Pages"
languageCode: "en-us"
canonifyurls: true
theme: "hyde"

indexes:
  category: "categories"
...
```

## 3 添加一篇文章

在/content/posts/ 下添加一篇Markdown格式(.md)的文章，大致的结构如下:

```html
---
title: "Just another sample post"
date: "2014-03-29"
description: "This should be a more useful description"
categories:
    - "hugo"
    - "fun"
    - "test"
---
```
## 4 在github上建立一个repo

现在要和github关联起来，先在github上建立一个repo，然后将本地项目和git远程仓建立起关系。假设你在github上repo的地址是:github.com/yours/gh_blog，本地操作就应该是:

```html
＃ 进入项目根目录，初始化Git,
git init
git add .
git commit -m "Init 1"

# 和远程仓建立映射
git remote add origin git@github.com:yours/gh_blog.git

# 提交到远程仓
git push -u origin master
```

## 5 用hugo生成静态页面提交到github上

接下来是一堆git的操作了，把hugo生成的/public/ 文件夹推送到远程仓的一个分支里.(也可以放到其他的远程仓.)

```html
# 创建新的 orphand 分支 (没有commit的历史纪录的) 命名为 gh-pages
git checkout --orphan gh-pages

# 撤销所有文件缓存记录
git rm --cached $(git ls-files)

# 添加和提交文件
git add .
git commit -m "INIT: initial commit on gh-pages branch"

# 推送到远程gh-pages分支
git push origin gh-pages

# 切换到master分支
git checkout master

# 删除public文件夹
rm -rf public

# 讲public文件夹作为子项目添加到gh-pages分支
git subtree add --prefix=public git@github.com:yours/gh_blog.git gh-pages --squash

# 下拉gh-pages分支合并项目
git subtree pull --prefix=public git@github.com:yours/gh_blog.git gh-pages

# 运行hugo命令，如果要使用主题后面加 -t ThemeName (在config.yaml里有设置theme也行.)
hugo -t ThemeName


# 添加所有文件
git add -A

# 提交并推送到远程仓
git commit -m "Updating site" && git push origin master

# 推送子项目到gh-pages分支
git subtree push --prefix=public git@github.com:yours/gh_blog.git gh-pages
```
所有命令都顺利跑通的话，应该就能通过yours.github.io/gh_blog 访问到搭起的博客.
