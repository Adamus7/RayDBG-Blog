---
layout: '[post]'
title: 我的Mac OS生产力配置
date: 2019-05-02 22:16:19
tags:
- Mac
---
写一个博客记录自己在Mac上做的一些生产力配置。
<!-- more -->
# Terminal升级
Mac原生的Terminal中规中矩，没有特别好看，也没有特别有效率。因此参考之前的经验决定：
1.  用iTerm2代替Terminal。
2.  用zsh代替bash。
3.  针对zsh做部分优化。

## iTerm2安装与配置
因为Homebrew的update最近一直很慢，因此直接从[iTerm2网站](https://www.iterm2.com/index.html)下载并安装。
修改iTerm2的Color Schemes，在这个[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes) repro中可以找到很多schemes，下载后通过`iTerm2` > `Preferences` > `Profiles` > `Colors` > `Color Presets...`来load需要的scheme。

## 安装zsh和oh-my-zsh
Bash作为Mac的默认shell已经足够好用，但是zsh在一些细节上更胜一筹。通过命令`cat /etc/shells`可以查看当前系统的shell类型。通过下面的命令来设置zsh作为默认shell:
```
chsh -s /bin/zsh
```
Zsh的一个问题是配置起来比较麻烦，然而[oh-my-zsh](https://ohmyz.sh/)的出现改变了这一现状。通过oh-my-zsh的theme，可以轻松的实现zsh的定制化。
### 安装oh-my-zsh:
```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
配置oh-my-zsh的theme：`sudo vim ~/.zshrc`。在.zshrc中找到并修改`ZSH_THEME="agnoster"`
### 安装nerd font
Agnoster主题的prompt依赖[nerd font project](https://nerdfonts.com/)中的一些特殊符号，如果没有，prompt中会有乱码出现，因此需要按照对应的字体，安装方法：
```
brew tap caskroom/fonts
brew cask install font-sourcecodepro-nerd-font
```
最后到iTerm2中，通过`Preferences` > `Profiles` > `Text` > `Change Font`来启用安装的nerd font`。
### 隐藏hostname
当前的hostname又臭又长，不想在本机使用的时候看到username@hostname的形式。按照这个[issue](https://github.com/agnoster/agnoster-zsh-theme/issues/39#issuecomment-307338817)中的comment来修改prompt的形式：
```
# 编辑.zshrc
# 在文件的末尾粘贴如下内容：
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```
最终的效果如下：
{% asset_img effect-1.png 600 %}
## iTerm2快捷键
```
新建Tab：Cmd + t
关闭Tab：Cmd + w
切换Tab：Cmd + 数字 Cmd + 左右方向键
切换全屏：Cmd + enter

屏幕结果搜索：Cmd + F
剪切板历史记录：Cmd + Shift + H
cmd历史记录：Cmd + Option + B
Tab排列：Cmd + Option + E

清除当前行：ctrl + u 
到行首：ctrl + a 
到行尾：ctrl + e 
删除光标之前的单词：ctrl + w 
删除到文本末尾：ctrl + k 
Undo：ctrl + _ 
Paste the last thing to be cut：ctrl + y 
```
## Vim语法高亮
开启Vi语法高亮：
```
#编辑.vimrc
vi ~/.vimrc

#添加
syntax on
```

# SSH配置





