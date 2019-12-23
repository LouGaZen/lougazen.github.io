---
title: 打造Mac下舒适的终端
date: 2019-12-12 23:54:46
tags:
---

## 序

拖延症拖了一年终于要开始打造自己的博客了_(´ཀ`」 ∠)_，先拿最近捣鼓的zsh开刀吧

## 准备

[brew](https://brew.sh/index_zh-cn)、iTerm2和[Oh-My-Zsh](https://ohmyz.sh/)

使用brew安装nerd-font字体

```shell script
brew tap caskroom/fonts
brew cask install font-hack-nerd-font
```

配置iTerm2针对非ASCII码字符的字体

![配置iTerm2针对非ASCII码字符的字体](https://i.loli.net/2019/12/16/moWqMef9wHILBYU.png)

## 配置zsh主题

zsh的配置文件存放在`~/.zshrc`下，看到

```markdown
ZSH_THEME="robbyrussell"
```

这一行，这里就是配置主题的地方，默认自带一堆主题放在`~/.oh-my-zsh/theme`目录下，也可以上[github](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)上查看各个主题的效果

当然个人感觉最叼的还是这个[powerlevel9k](https://github.com/Powerlevel9k/powerlevel9k)主题

![powerlevel9k主题](https://i.loli.net/2019/12/16/NUrfXGsnjuo8Zxv.gif)

首先把主题安装到本地

```shell script
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

修改`~/.zshrc`的主题配置

```markdown
ZSH_THEME="powerlevel9k/powerlevel9k"
```

重新加载zsh使配置生效

```shell script
source ~/.zshrc
```

然后提示就变样了（先忽略苹果logo的信息，后面会提到）

![安装powerlevel9k](https://i.loli.net/2019/12/16/pCnDS4baKfAdetZ.png)

根据自己的喜好配置一下（注意需要加在ZSH_THEME的前面，不然前面说的nerd-font字体会加载不出来），详细可进入p9k的github阅读原文档

```markdown
POWERLEVEL9K_MODE="nerdfont-complete"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon root_indicator context ssh dir_writable dir vcs status)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=()
POWERLEVEL9K_PROMPT_ADD_NEWLINE=true
```

* `POWERLEVEL9K_MODE`：设置 powerlevel9k 的字体是我们前面下载的
* `POWERLEVEL9K_LEFT_PROMPT_ELEMENTS`：将前面居右的几个元素放在左边了
* `POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS`：右边不放置任何元素（如果你喜欢在右边也可以加）
* `POWERLEVEL9K_PROMPT_ADD_NEWLINE`：在每个提示之前添加换行符

修改完的效果应该是这样子的

![配置powerlevel9k](https://i.loli.net/2019/12/16/6xi9PybSLRKcJz3.png)

## 配置iTerm2配色

iTerm2默认黑色还是太丑了，上[这里](https://github.com/mbadolato/iTerm2-Color-Schemes)挑了个Ubuntu主题

![iTerm2配色](https://i.loli.net/2019/12/16/CynTDgk7PRGXh8f.png)

啊舒服了٩(๑´0`๑)۶

## 安装zsh插件

以上那些都是过眼隐的，真正让使用者感到舒服的地方来了

先列一下我的插件列表

* git
* zsh-syntax-highlighting
* zsh-autosuggestions
* command-not-found
* zsh_reload
* git-open
* z
* safe-paste
* sudo
* extract

### git

默认自带，不说

### zsh-syntax-highlighting

用于显示你当前输入的命令是否正确

```shell script
brew install zsh-syntax-highlighting
```

正常来讲可以直接把`zsh-syntax-highlighting`加到配置文件中即可启用，如果出现`plugin not found`的情况则改用如下方式进行配置

```markdown
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

### zsh-autosuggestions

历史记录提示，按➡️方向键自动补全

```shell script
brew install zsh-autosuggestions
```

跟上一个配置同理，如果出现`plugin not found`的情况则改用如下方式进行配置

```markdown
source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

### command-not-found

对于command not found的结果会提示一个相似的命令（多出现于拼写错误）

### zsh_reload

直接执行`src`即可完成`source ~/.zshrc`操作

### git-open

在终端里打开当前项目的远程仓库地址

```shell script
git clone https://github.com/paulirish/git-open.git $ZSH_CUSTOM/plugins/git-open
```

### z

zsh自带的autojump

### safe-paste

顾名思义，对于有换行符的内容不会立马执行

### sudo

双击 Esc，zsh 会把上一条命令加上 sudo 给你

### extract

万能解压命令

## 其他非zsh的插件

### colorls

列出文件时带图标

```shell script
gem install colorls
```

![colorls](https://i.loli.net/2019/12/16/qrk1oOXgwPUsd4p.png)

然后加个alias到配置文件

```markdown
alias cll='colorls -l --gs'
```

以后输入`cll`即可

### archey

显示系统信息

```shell script
brew install archey
```

![archey](https://i.loli.net/2019/12/16/85scRPWveNCi4kM.png)

然后把`archey`这个命令加到配置文件即可每次打开都显示系统信息

顺带提一下archey那种图案的生成方式，有两个小工具可以做到，一个是figlet，另一个是toilet

```shell script
brew install figlet
brew install toilet
```

![figlet与toilet](https://i.loli.net/2019/12/16/gwlevVHQUCZAFr6.png)

## ~~后记（挖坑）~~

~~这一套下来在iTerm下面使用简直是强无敌，但是考虑还有系统自带的终端及idea和vscode的终端都要配置一遍是在是烦躁，在寻找有没有办法针对不同的shell加载不同的配置文件~~

## （2019-12-22填坑）针对不同终端使用不同的主题

通过查看[archey](https://github.com/obihann/archey-osx/blob/master/bin/archey)发现，终端软件（不是shell）的信息存放在`${TERM_PROGRAM}`这个变量里，遂修把配置文件中`ZSH_THEME="powerlevel9k/powerlevel9k"`这一行改为

```markdown
if [ "${TERM_PROGRAM}" = "iTerm.app" ]; then
    POWERLEVEL9K_MODE="nerdfont-complete"
    POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon root_indicator context ssh dir_writable dir vcs status)
    POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=()
    POWERLEVEL9K_PROMPT_ADD_NEWLINE=true
    ZSH_THEME="powerlevel9k/powerlevel9k"
else
    ZSH_THEME="ys"
fi
```

（顺便把powerlevel9k的一些配置也放到了一起）

最终效果

![自带Terminal和vscode](https://i.loli.net/2019/12/22/6pcKn2rWsANwaMf.png)
![iTerm](https://i.loli.net/2019/12/22/euFiva8HWTqzMk1.png)

完美
![👏](https://i.loli.net/2019/12/22/XiYMjbp7cIRslaE.gif)

## 鸣谢

> [打造 Mac 下高颜值好用的终端环境](https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html)
> [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)
> [powerlevel9k](https://github.com/Powerlevel9k/powerlevel9k)
> [iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)
