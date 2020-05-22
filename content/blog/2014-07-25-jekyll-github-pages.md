---
title: How to setup Jekyll with GitHub Pages
subtitle:
date: '2014-07-25'
slug: jekyll-github-pages
categories:
  - Setup
  - Front End
tags:
---

## 安装Ruby
1. 安装curl：

        # 检查是否已经安装
        dpkg -s curl
        # 安装命令
        sudo apt-get install curl
2. 安装RVM：

        curl -L get.rvm.io | bash -s stable

    在 `$HOME/.bash_profile` 和 `$HOME/.bashrc` 中添加

        [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
        # This loads RVM into a shell session.

    重启终端后查看RVM版本

        rvm -v

    通过 `rvm requirements` 命令，查看需要安装那些依赖包，之后将它们安装

        sudo apt-get install a b c d
        # 两个包之间用空格隔开
3. 安装 ruby

    察看当前 RVM 中已经安装的 ruby 版本

        rvm list	#应该是没有任一版本

    察看 RVM 可供安装的 ruby 版本

        rvm list known

    安装 ruby 2.1-head

        rvm install 2.1-head

    检查版本

        rvm list

    选择 2.1 作为当前的使用版本，并且设置为缺省

        rvm use ruby-2.1-head --default

    设置好之后察看 ruby 版本

        ruby -v

    察看 ruby 的路径，就是 RVM 帮我们安装的

        which ruby

##	安装 node.js
安装原因可参见 [Ruby China](https://ruby-china.org/topics/692)

1. 从 nodeJS 官网 [http://nodejs.org/](http://nodejs.org/) 下载最新源代码包：`node-vx.x.xx.tar.gz`
2. 解压并复制到 */usr/lib* 中

        cp -r [源文件夹] [目的文件夹]

3. 切换到文件目录

        ./configure 
        make
        make install
        node --version

## 安装 Jekyll
1. 执行代码

        gem install jekyll
        jekyll -v
        gem install rdiscount
        gem install kramdown 
2. 建立博客

        cd ~
        jekyll new blog_name	#blog_name为博客目录名字，可自取
        cd blog_name
        jekyll server
3. 访问 [http://localhost:4000](http://localhost:4000) 即可在本地访问。

## Jekyll 与 GitHub Pages 配合
1. Create a repository

    Head over to GitHub and create a new repository named username.github.io, where username is your username (or organization name) on GitHub.
2. Clone the repository

    Go to the folder where you want to store your project, and clone the new repository:

        git clone https://github.com/username/username.github.io
3. Hello World

    Enter the project folder and add an index.html file:

        cd username.github.io
        echo "Hello World" > index.html
4. Push it

    Add, commit, and push your changes:

        git add --all
        git commit -m "Initial commit"
        git push
5. …and you're done!

	Fire up a browser and go to http://username.github.io. Give it a couple of minutes for your page to show up—there will be a delay this very first time. In the future, changes will show up pretty much instantly.
