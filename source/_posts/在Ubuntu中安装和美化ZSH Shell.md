---
title: 在Ubuntu中安装和美化ZSH Shell
tags: ["Linux"]
categories: Linux
date: 2018-07-16
grammar_cjkRuby: true
copyright: ture
---

> ZSH 或称 Z Shell 是一个类似于 Bash 和 SH 的 Linux Shell，它具有一些 Bash 和其它 Shell 不具备的高级功能。流行的 Git 版本控制系统也可以使用插件与 ZSH 很好的集成，这一点对软件开发人员来说非常有用。而且 ZSH 有非常多的主题和插件支持，比 Bash 更具可定制性。简单来说，ZSH 就是一款替代 Bash 且非常好用的 Linux Shell。

<!-- more -->

### 安装ZSH Shell
```shell
sudo apt update
sudo apt install zsh
```

### 设置为默认Shell

```shell
whereis zsh
sudo usermod -s /usr/bin/zsh $(whoami)
```

现在使用 reboot 或 init 6 命令重新启动 Ubuntu 计算机。

重启后，打开终端，输入`数字 2`，ZSH 会使用推荐的设置创建一个新的 ~/.zshrc 配置文件，此后就可以正常使用 ZSH Shell 了。

### 安装Powerline和Powerline字体

「Powerline」是 ZSH Shell 的状态行插件，「Powerline 字体」也允许 ZSH 在 Shell 中使用不同的图标和符号。而「Powerline」和「Powerline 字体」也可以在 Ubuntu 的官方软件包存储库中找到。

```shell
sudo apt install powerline fonts-powerline
```

### ZSH、Git与「Oh My ZSH」集成

安装 Git

```shell
sudo apt install git
```

Git 安装好之后，请执行以下命令在 Ubuntu 系统中安装「Oh My ZSH」

```shell
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

> 注意：安装「Oh My ZSH」会自动更改 ~/.zshrc 配置文件，这意味着安装的 ZSH Syntax Highlighting 插件将会被禁用。可以在「Oh My ZSH」安装好之后执行zsh的相关命令，重新启用 ZSH Syntax Highlighting 插件。

### 启用语法高亮显示

使用它可以突出显示 ZSH Shell 中的命令

```shell
sudo apt install zsh-syntax-highlighting
```

将`/usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`目录下在读文件复制到`~/.oh-my-zsh/plugins/zsh-syntax-highlighting`目录下，然后编辑`~/.zshrc`文件，在最末尾加入`source ~/.oh-my-zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`。

打开一个新的「终端」窗口开始输入命令，应该就可以看到命令以不同的颜色突出、高亮显示。

### 自动补齐插件

下载此插件，[插件官网](http://mimosa-pudica.net/zsh-incremental.html)

```shell
cd /temp
wget http://mimosa-pudica.net/src/incr-0.2.zsh
```

将此插件放到oh-my-zsh目录的插件库下 `~/.oh-my-zsh/plugins/incr`。

在`~/.zshrc`文件末尾加上

```shell
source ~/.oh-my-zsh/plugins/incr/incr-0.2.zsh
```

### 更新配置

```shell
source ~/.zshrc   
```

### 优化zsh的配置

修改`~/.zshrc`文件

```shell
# zhs的主题
ZSH_THEME="ys"	
```

```shell
# z命令快速跳转目录     x命令解压一切文件         命令行可以直接google  
plugins=(
  git z zsh-autosuggestions extract web-search zsh-syntax-highlighting 
)
```

### 与vim的提示相冲突的解决方案

使用自动补全插件可能会与vim的提示功能相冲突，如会报以下错误：

```shell
vim t
_arguments:451: _vim_files: function definition file not found
```

解决方法：将~/.zcompdump*删除即可

```shell
rm -rf ~/.zcompdump*
exec zsh
```



