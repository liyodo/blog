---
title: "使用Hugo + GitHub搭建静态网站"
date: 2018-09-14T21:50:50+08:00
draft: false
categories:
  - "Development"
tags:
  - "Hugo"
---

Hugo是一个开源的静态网站生成器。
<!--more-->

## 搭建环境(macOS)

### 安装Homebrew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 安装Hugo
```
brew install hugo
```

### 安装Git

## 搭建网站

### 创建目录
```
hugo new site <SITE_NAME>
```
执行后会在当前目录下创建一个名为SITE_NAME的文件夹，所有网站相关的文件都在里面。

### 使用主题
```
cd <SITE_NAME>
git init
git submodule add <THEME_URL> themes/<THEME_NAME>
```
在项目根目录初始化git仓库，添加主题文件至themes文件夹。
在这里初始化git仓库的目的仅仅是为了使用git命令添加主题。
当然也可以直接在这里把项目与GitHub上的仓库关联：
```
git remote add origin <REMOTE_URL>
```
注意：在`.gitignore`文件中增加`public/`，原因见下文。

[这里](https://themes.gohugo.io/ "Hugo Themes")有各式各样的主题.

在`<SITE_NAME>/config.toml`中指定主题：
```
theme = "<THEME_NAME>"
```
第一次指定主题也可以使用命令：
```
# Edit your config.toml configuration file
# and add the theme.
echo 'theme = "<THEME_NAME>"' >> config.toml
```
还可以在config.toml中配置更多的主题效果，详情查看主题的文档。

### 增加文章
```
hugo new posts/my-first-post.md
```
使用该命令会在content目录下创建posts目录，并在posts目录中创建my-first-post.md文件。文件包含默认Front Matter：
```
---
title: "使用Hugo + GitHub搭建静态网站"
date: 2018-09-14T17:21:12+08:00
draft: true
---
```
在Front Matter下面就可以输入文章正文啦。

### 启动Hugo服务
```
hugo server -D
hugo server -t <THEME_NAME> --buildDrafts
```
常用参数：

* -D, --buildDrafts include content marked as draft
新创建文章的front matter中包含`draft: true`属性，这表示该文章为草稿状态。若不使用`-D`标记，草稿是不会在网站中显示的。
* -t, --theme string theme to use (located in /themes/THEME_NAME/)
`-t`用于指定网站使用的主题，例如themes目录下有名为"mainroad"的主题，则可使用`-t mainroad`。

### 访问网站

http://localhost:1313/

## 部署GitHub Page

### 创建GitHub Page
GitHub Page有两种，分别是用户/组织网站（User or organization site）和项目网站（Project site），我选择前者。
与创建一个普通的repository方法基本相同，只是repository的名字必须是\<USERNAME\>.github.io，
USERNAME为Github账号的用户名或组织名，必须完全一致。

### 为public目录创建submodule
首先前提是使用浏览器访问过网站，并按下Ctrl+C杀掉服务。
然后使用`rm -rf public`完全删除public目录(我执行操作2后没有生成public目录，所以跳过了这一步)。
然后执行命令将刚创建的GitHub Page添加为submodule，位于public目录下：
```
git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public
```
这样的效果是public目录下的文件会单独提交到GitHub Page仓库。

### 部署脚本
这个名为`deploy.sh`的脚本用于构建并提交public目录下的内容到GitHub Page仓库。
使用`chmod +x deploy.sh`赋予执行权限并放在项目根目录下。
用法：
```
./deploy.sh "Your optional commit message"
```
脚本代码：
```
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```
因为public文件夹作为GitHub Page submodule，所以项目仓库要忽略public文件夹，
这样在GitHub仓库上看public文件夹是这样的："public @ cfdf4a0"，点击可以跳转到GitHub Page仓库。

### 访问网站

通过https://\<USERNAME\>.github.io浏览网站。

## Others

使用mainroad主题时，主页会将文章内容忽略markdown格式排版直接显示出来，很丑。想避免这种情况，可以使用`<!--more-->`。
在它之前的内容会以markdown格式排版后的样式显示。在它之后的内容不会显示。
不知道是只适用于mainroad主题，还是markdown通用的，还是Hugo通用的。


### Todo

* 了解git submodule，包括更新、删除等。[Git - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97 "Git - 子模块")

## 参考资料

* [Hugo](https://gohugo.io/)
* [Hugo中文文档](http://www.gohugo.org/ "Hugo中文文档")
* [GitHub Pages](https://pages.github.com/)
* [Vimux/Mainroad](https://github.com/Vimux/Mainroad)
