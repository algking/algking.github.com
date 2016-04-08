---
layout: post
category: "工具"
tags: [zsh,fish,linux]
---

[TOC]

### zsh 与yosemite 的bug? ###
在更新了Mac Yosemite 后,发现各种问题,首先是php,macport等问题
接着就是zsh了,不知道为什么,`zsh`总是几乎占了100%的cpu,这让我的macbook电池
暴跌,非常郁闷. 开始怀疑是插件的问题,但是即使把插件全部关了,也还是那样.
之前也用过fish,发现还是不错的一个shell,从设计上面说,非常方便.功能也不错.
于是就准备换到fish算了.


![image](/public/img/zsh-yosemite-bug.png)
> 发了封邮件给zsh后,
>reply: Any chance that it's this issue with zsh-autosuggestions?

[问题解决](https://github.com/tarruda/zsh-autosuggestions/issues/24 )
发现原来是因为zsh-autosuggestions 的问题.


### fish优点 ###
- Autosuggestions 自动提示history,命令补全等很方便
- 命令的completions 甚至包括man的提示
- 一些`zsh`我喜欢的插件fish也有 例如`autojump` 通过[oh-my-fish](https://github.com/bpinto/oh-my-fishURL ) 可以很方便安装
- `fish-config`命令 可以在线的编辑fish的配置

其实以上一些功能其实`zsh`也可以做到,不过个人觉得补全做的没有`fish`好,只是一直以来`zsh`的社区比较强大
而`fish` 插件会少点,但是一般使用其实用不上很多插件,而且`zsh`用多几个插件就变得很慢.
一直使用 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh ) ,但是发现了[oh-my-fish](https://github.com/bpinto/oh-my-fishURL ) 后
就想转过去了,因为以前一直以为fish没有插件支持.


### 安装[oh-my-fish](https://github.com/bpinto/oh-my-fishURL )###
    brew install fish
    sudo vi /etc/shells 将/usr/local/bin/fish加上,否则下面的命令会报错
    chsh -s /usr/local/bin/fish
    
    git clone git://github.com/bpinto/oh-my-fish.git ~/.oh-my-fish
    copy配置文件
    cp ~/.oh-my-fish/templates/config.fish ~/.config/fish/config.fish
    
### 设置fish ###

- 编辑~/config/fish/config.fish

```
   set fish_plugins autojump bundler brew
   set -xu PATH  /usr/local/bin:$PATH
   
   比较不爽的就是export 在这里不能用要使用`set -x`代替:

   `set -x PATH  /usr/local/bin $PATH`
   
   -x : -export 
   -u : 意思是对所有fish session都使用
```
### 编写fish插件 ###

fish 的插件看起来非常好懂,是基于函数的

```ruby
function rg
  rails generate $argv
end

```

    







