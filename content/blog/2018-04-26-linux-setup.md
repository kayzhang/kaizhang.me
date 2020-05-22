---
title: My Linux Development Environment Setup
subtitle:
date: '2018-04-26'
slug: linux-setup
categories:
  - Setup
tags:
  - Linux
  - Ubuntu
---

本文记录我在 Ubuntu 上搭建开发环境及其他配置的具体配置信息。

# 开发环境
## 基本配置
**> 启用 root 账户**

    sudo passwd root

**> 安装 oh-my-zsh**

官方指南：[oh-my-bash](https://github.com/robbyrussell/oh-my-zsh)

1. 安装 zsh

        sudo apt install zsh

    通过 `zsh --version` 查看版本

2. 设置 zsh 为默认 shell

    首先通过 `echo $SHELL` 查看默认 shell，结果为 `/bin/bash`；`$SHELL --version` 结果为 `GNU bash, version 4.4.19(1)-release (x86_64-pc-linux-gnu)`。

    然后确认 zsh 在 shell 列表 */etc/shells* 里边，之后设置默认 shell 为 zsh：

        chsh -s $(which zsh)

    注销账户之后，查看 `echo $SHELL` 结果为 `/usr/bin/zsh`；`$SHELL --version` 结果为 `zsh 5.4.2 (x86_64-ubuntu-linux-gnu)`。

3. 安装 oh-my-zsh

    通过 curl 安装：

        sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

    或通过 wget 安装：

        sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

    **注意：**安装 JDK 时只在 `~/.bashrc` 中添加了环境变量，则 zsh 无法找到 JDK，因此要将环境变量的设置代码同时加在文件 `~/.zshrc` 的结尾。

**> 安装 Guake 下拉式终端**

    sudo apt install guake

**> 在设置中安装中文支持环境**

**> 设置中文优先级**

在安装时选择英文语言的情况下，部分汉字（如“复”）会显示不正常，只占据一半的宽度，这是因为系统默认优先显示日文汉字，之后是汉文、简体、繁体，因此要更改中文优先级为最优，具体为打开文件 `/etc/fonts/conf.avail/64-language-selector-prefer.conf` 将相应部分更改为 SC-->TC-->JP-->KR 的顺序，然后注销一下即可。

**> 安装 ibus-libpinyin 拼音输入法**

    sudo apt install ibus-libpinyin

**> 安装搜狗输入法**

官方指南：[搜狗输入法 for Linux](https://pinyin.sogou.com/linux/)

    sudo apt install fcitx

    dpkg -i install sogoupinyin_2.2.0.0108_amd64.deb

注意：不要登录个人中心，否则重启之后候选词列表会出现乱码。若登录导致出现乱码，可以通过一下删除以下文件恢复未登录状态：

    cd ~/.config
    rm -r Sogou*

**> 安装 google 输入法**

    sudo apt install fcitx-googlepinyin

**> 更新软件**

首先更改软件源，然后更新源

    sudo apt update

之后在软件中心进行软件更新。

**> Chrome**

官方指南：[Linux Software Repositories](https://www.google.com/linuxrepositories/)

Chrome 安装包会自动配置 repository，直接从官网下载 deb 包安装即可。

## Git & GitHub
**> Git**

官方指南：[http://www.git-scm.com/book/zh/](http://www.git-scm.com/book/zh/) & [https://help.github.com/articles/set-up-git/](https://help.github.com/articles/set-up-git/)

    sudo apt install git

设置用户名和邮件

    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com

查看用户名和邮件

    git config --list

**> GitHub**

官方文档：[https://help.github.com/articles/set-up-git/](https://help.github.com/articles/set-up-git/)

注：开了两步验证之后，必须用 token 作为密码 push，因此选择 store 而非 cache 会更加方便一点。

    git config --global credential.helper store

下边是之前用的 cache 方式：

Set git to use the credential memory cache:

    git config --global credential.helper cache

Change the default password cache timeout (15 minutes) to 1 hour:

    git config --global credential.helper 'cache --timeout=3600'

## 编辑器
**>	Vim**

    sudo apt install vim

直接把 */usr/share/vim..../vimxx/* 下的 `vimrc_example.vim` 拷到主目录下并改名 `.vimrc` 来保存配置信息。

    cp /usr/share/vim/vimxx/vimrc_example.vim ~/.vimrc

在文件结尾添加以下设置

    set shiftwidth=4   "设置缩进的空格数为4
    set nu             "打开行号

取消自动生成备份文件，找到以下代码

    if has("vms")
    set nobackup " do not keep a backup file, use versions instead
    else
    set backup " keep a backup file

注释掉后两行，即

    if has("vms")
    set nobackup " do not keep a backup file, use versions instead
    "else
    " set backup " keep a backup file

**> Emacs**

    sudo apt install emacs25

在主目录下建立配置文件 *~/.emacs*，并添加以下内容

    ;; emacs配置信息
    ;; 取消自动备份
    (setq make-backup-files nil)
    ;; 代码折叠
    (add-hook 'c-mode-hook 'hs-minor-mode)
    (add-hook 'c++-mode-hook 'hs-minor-mode)
    (add-hook 'java-mode-hook 'hs-minor-mode)

**> Sublime Text 3**

官方指南：[Linux Package Manager Repositories](https://www.sublimetext.com/docs/3/linux_repositories.html#apt)

注：https 地址貌似用不了，改成 http 即可。

    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
    sudo apt install apt-transport-https
    echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
    sudo apt update
    sudo apt install sublime-text

采用 Package Control 自动配置，具体参照 [https://github.com/kayzhang/sublime-config](https://github.com/kayzhang/sublime-config)

**> Atom**

官方指南：[Installing Atom on Linux](https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux)

注：https 地址貌似用不了，改成 http 即可。

    curl -L https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
    sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
    sudo apt update
    sudo apt install atom-beta

Atom 配置自动同步：[Sync Settings for Atom](https://atom.io/packages/sync-settings)

安装 `sync-settings`

GITHUB_TOKEN: d8(kai: delete this)dafa8621684fcca595b0fb423fa254b540e863

GIST_ID: 24c2d067a9e79805ece05b8a26eaf0f6

## Python
在 Ubuntu 18.04 中默认安装的 python3，但是有些软件需要 python2 的支持，如 Dropbox，因此就会出现 2 个 python 版本，下边是设置 python3 为默认版本的方法。

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 10
    sudo update-alternatives --config python

## Java
一个更简单的方法：

    sudo add-apt-repository ppa:linuxuprising/java
    sudo apt update
    sudo apt install oracle-java11-installer

下面为常规的安装方式：

**> JDK**

官方指南：[Installation of the JDK and JRE on Linux Platforms](https://docs.oracle.com/javase/10/install/installation-jdk-and-jre-linux-platforms.htm#JSJIG-GUID-79FBE4A9-4254-461E-8EA7-A02D7979A161)

参考 Fedora 教程：[Installing Oracle JDK on Fedora](https://fedoraproject.org/wiki/JDK_on_Fedora)

1. 到 [官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载压缩包 `jdk-x.x.x_linux-x64_bin.tar.gz`
2. 解压并将解压后的文件复制到 */usr/local/java/*

        sudo mkdir -p /usr/local/java
        sudo tar zxvf ./jdk-x.x.x_linux-x64_bin.tar.gz  -C /usr/local/java

3. 修改环境变量，`vim ~/.bashrc`，添加以下内容

        # 添加 JDK 环境变量
        JAVA_HOME=/usr/local/java/jdk-x.x.x
        PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
        export JAVA_HOME
        export PATH

    注：若系统中没有预装 openjdk，那么第 4、5 步可以忽略。

4. Tell the system that the new Oracle Java version is available by the following commands:

        sudo update-alternatives --set java /usr/local/java/jdk-x.x.x/bin/java
        sudo update-alternatives --set javac /usr/local/java/jdk-x.x.x/bin/javac
        sudo update-alternatives --set javaws /usr/local/java/jdk-x.x.x/bin/javaws

5. 设置 Oracle JDK 为默认版本

        sudo update-alternatives --set java /usr/local/java/jdk-x.x.x/bin/java
        sudo update-alternatives --set javac /usr/local/java/jdk-x.x.x/bin/javac
        sudo update-alternatives --set javaws /usr/local/java/jdk-x.x.x/bin/javaws

6. 重载配置信息：`source ~/.bashrc`

7. 查看 JDK 版本：`java -version`

## 前端
**> R**

官方指南：[https://www.r-project.org/](https://www.r-project.org/)

以下为针对 ubuntu artful，即 ubuntu 17.10 的安装方法，18.04 暂时可以用这一个源：

    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

在 `/etc/apt/sources.list` 结尾添加（注意，一定不要用 https）：

    deb http://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/linux/ubuntu artful/

之后安装

    sudo apt update
    sudo apt install r-base
    sudo apt install r-base-dev

**> RStudio**

下载 deb 安装包：[https://www.rstudio.com/products/rstudio/download/](https://www.rstudio.com/products/rstudio/download/)

安装本地 deb 包安装工具

    sudo apt install gdebi

安装 RStudio

    sudo gdebi rstudio-xenial-1.x.xxx-amd64.deb

**> blogdown**

    ## Install from CRAN
    install.packages("blogdown")
    ## Or, install from GitHub
    if (!requireNamespace("devtools")) install.packages("devtools")
    devtools::install_github("rstudio/blogdown")

**> 其他需要的包**

    install.packages(c("processx", "later"))
    options(blogdown.generator.server = TRUE)

**> hugo**

    blogdown::install_hugo()

更新或重装 hugo

    blogdown::update_hugo()

或

    install_hugo(force = TRUE)

查看 hugo 版本，最新版本：[https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)

    blogdown::hugo_version()

## Machine Learning
**> 安装 Anaconda**

参考 [官方教程](https://docs.anaconda.com/anaconda/install/linux) 安装 Anaconda

将 Anaconda 自动添加到 ~/.bashrc 中的环境变量复制到 ~/.zshrc 中：

    # added by Anaconda3 installer
    export PATH="/home/kai/anaconda3/bin:$PATH"

添加 conda-forge 为最优先的源：

    conda config --add channels conda-forge
    conda config --set show_channel_urls yes
    conda config --show

更新 conda 及 Anaconda：  
注：不要在 root 环境中使用 `conda update --all`，会导致依赖出现问题。在其他的环境中可以使用此命令进行更新所有包。

    conda update conda
    conda update anaconda

**> 安装 TensorFlow**

参考 [官方教程](https://www.tensorflow.org/install/install_linux#installing_with_anaconda)

通过调用以下命令创建名为 tensorflow 的 conda 环境，以运行某个版本的 Python：

    conda create -n tensorflow pip python=3.6

更新 tensorflow 环境下的 conda 及所有的 packages

    source activate tensorflow
    (tensorflow)$ conda update --all

发出以下格式的命令以在 conda 环境中安装 TensorFlow：

    # 注：以下为官方推荐的原生 pip 安装方式，但是会和 conda 产生冲突，导致软件出现问题。
    # (tensorflow)$ pip install --ignore-installed --upgrade tensorflow
    # 因此，可以选择安装 conda 的版本，虽然不受官方支持，但应该不会有大的影响。
    (tensorflow)$ conda install tensorflow

更新：

    (tensorflow)$ conda update --all

查看 tensorflow 版本：

    (tensorflow)$ python
    import tensorflow as tf
    tf.__version__

验证安装成功：

```python
# Python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

若为 pip 方式安装的 tensorflow，则更新如下：

    # (tensorflow)$ pip install --upgrade pip
    # (tensorflow)$ pip install --upgrade tensorflow

安装常用的 packages：

    (tensorflow)$ conda install ipython
    (tensorflow)$ conda install jupyter
    (tensorflow)$ conda install matplotlib
    (tensorflow)$ conda install keras
    (tensorflow)$ conda install tqdm
    (tensorflow)$ conda install pillow
    (tensorflow)$ conda install pandas
    (tensorflow)$ conda install nltk
    (tensorflow)$ conda install opencv

卸载 Anaconda：  
参考 [Uninstalling Anaconda](https://docs.anaconda.com/anaconda/install/uninstall)

    conda install anaconda-clean
    anaconda-clean --yes
    rm -rf ~/anaconda3

# 其他配置
**> gnome-tweak-tool**

    sudo apt install gnome-tweak-tool

安装 [gnome shell extensions](https://extensions.gnome.org/)

首先安装浏览器插件，然后安装本地支持

    sudo apt install chrome-gnome-shell

安装扩展 `User Themes` 和 `Dash to Dock` 和 `Lock Keys`

**> Albert**

官方指南：[Installing Albert](https://albertlauncher.github.io/docs/installing/)

以下针对 Ubuntu 18.04，首先是添加 key

    wget -nv https://download.opensuse.org/repositories/home:manuelschneid3r/xUbuntu_18.04/Release.key -O Release.key
    sudo apt-key add - < Release.key
    sudo apt update

然后安装 Albert

    sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_18.04/ /' > /etc/apt/sources.list.d/home:manuelschneid3r.list"
    sudo apt update
    sudo apt install albert

**> Transmissionbt**

    sudo add-apt-repository ppa:transmissionbt/ppa
    sudo apt update
    sudo apt install transmission

**> 新立得软件包管理器**

    sudo apt install synaptic

**> Ubuntu 额外的版权受限程序**

一些受限的音频和视频解码器

**> MPV**

    sudo add-apt-repository ppa:mc3man/mpv-tests
    sudo apt update
    sudo apt install mpv

**> SMPlayer**

官方指南：[Install SMPlayer for Ubuntu](https://www.smplayer.info/en/downloads)

    sudo add-apt-repository ppa:rvm/smplayer
    sudo apt update
    sudo apt install smplayer smplayer-themes smplayer-skins

[如何在 SMPlayer 中启用 mpv](http://www.smplayer.info/zh/mpv)  
要想使用 mpv 替代 MPlayer，请打开 SMPlayer 首选项 在通用选项卡中选择 mpv 作为多媒体引擎。

**> Dropbox**

官方指南：[安装 Dropbox](https://www.dropbox.com/install)

1. 首先第一步可以通过下载官网的 deb 包安装 Dropbox 前端：

        sudo dpkg -i dropbox_2015.10.28_amd64.deb

    这一步，也可以通过以下命令直接安装：

        sudo apt install nautilus-dropbox

    在安装的时候可能需要安装一些依赖库，按照提示操作就行。

2. 在启动器中打开 Dropbox

    此时可能会刚开始没反应，之后[报错](https://askubuntu.com/questions/562018/dropbox-and-ubuntu-software-center-doesnt-work-after-setting-python3-4-as-defau?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)，Google 之后发现是将默认 python 由 python2 设置为 python3 导致的，解决方案为：

    将文件 `/usr/bin/dropbox` 中第一行的 `#!/usr/bin/python` 改为 `#!/usr/bin/python2`。

3. 打开 Dropbox，下载后端程序 dameon

    此时发现无法下载，提示 *Trouble connecting to Dropbox servers. Maybe your internet connection is down, or you need to set your http_proxy environment variable.*

    原因在于，Dropbox 下载过程中不使用系统代理，因此选择手动安装 demeon：


        cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

    接着，从新建的 *.dropbox-dist* 文件夹运行 Dropbox 守护程序

        ~/.dropbox-dist/dropboxd

    之后在弹出的页面中登陆即可。

    此时可以直接启动器启动，但是有一个问题就是终端打开 `~/.dropbox-dist/dropboxd` 可以使用系统代理，而启动器打开不能使用，只需在 Dropbox 的设置里边把代理服务器配置一下就行了。

**> 网易云音乐**

注意，官方版本太久未更新，不要用，从以下链接下载：http://archive.ubuntukylin.com/ubuntukylin/pool/main/n/netease-cloud-music/

**> Kazam**

录屏软件，1.5.3 版本加入了摄像头支持，但是在 18.04 上无法调用摄像头，可以装旧版 1.4.5：

    sudo apt install kazam

**> SimpleScreenRecorder**

录屏软件，比 Kazam 定制性更强：

    sudo apt install simplescreenrecorder

**> Guvcview**

摄像头录制软件

    sudo add-apt install guvcview

**> OpenConnect (用来代替 Cisco AnyConnect)**

    sudo apt install network-manager-openconnect
    sudo apt install network-manager-openconnect-gnome

**> Shadowsocks Qt5**

    sudo add-apt-repository ppa:hzwhuang/ss-qt5

最新只更新到 17.10 Artful，需要去软件中心将源由 bionic 更改为 artful，然后安装即可：

    sudo apt update
    sudo apt install shadowsocks-qt5

**> 截图工具 shutter**

    sudo apt install shutter

**> Shotwell**

参考 [PPA for Shotwell releases](https://launchpad.net/~yg-jensge/+archive/ubuntu/shotwell)

    sudo add-apt-repository ppa:yg-jensge/shotwell
    sudo apt update
    sudo apt install shotwell

**> GIMP**

参考 [GIMP](https://launchpad.net/~otto-kesselgulasch/+archive/ubuntu/gimp)

    sudo add-apt-repository ppa:otto-kesselgulasch/gimp
    sudo apt update
    sudo apt install gimp

**> Okular pdf 阅读器**

    sudo apt install okular

**> PDF-Shuffler**

    sudo apt update
    sudo apt install pdfshuffler

**> Spotify**

    snap install spotify

**> Aria2**

    sudo apt install aria2

**> Electronic WeChat**

    sudo snap install electronic-wechat

**> Wunderlist**

参见 [wunderlistux](https://github.com/edipox/wunderlistux)  
直接下载 deb 包用 dpkg 安装即可。

**> 解决耳机白噪音的问题**

终端打开 alsamixer，将 headphone mic boost 从 0 设置为非零的一个比较小的数即可，但是这样每次重启设置并未保存，需要进行一下[配置](https://lists.freedesktop.org/archives/pulseaudio-discuss/2016-January/025271.html)：

将以下两个文件中的 [Element Headphone Mic Boost] 部分注释掉：

/usr/share/pulseaudio/alsa-mixer/paths/analog-input-internal-mic.conf  
/usr/share/pulseuadio/alsa-mixer/paths/analog-input-headphone-mic.conf
