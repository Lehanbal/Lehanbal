title: Ubuntu环境搭建
author: Lehanbal
tags:
  - 效率
categories:
  - linux
date: 2022-07-26 00:16:00
---
# ubuntu环境搭建

1. 在[Ubuntu Releases](https://releases.ubuntu.com/)中选择想要的版本，我选择了22.04的server版本，不需要图形界面。
2. 刻录启动盘，推荐使用ventoy。
3. 进入服务器引导启动安装系统

# 镜像源

位置在

` /etc/apt/sources.list` 

更改好源之后需要执行命令更新源内容

`sudo apt update`

`sudo apt upgrade`

## aliyun

```bash
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse

# deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse

## Pre-released source, not recommended.
# deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
```

## 清华源

```bash
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

## Pre-released source, not recommended.
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

## 中科大

```bash
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

## Pre-released source, not recommended.
# deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

## 网易163

```bash
deb http://mirrors.163.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-backports main restricted universe multiverse

# deb-src http://mirrors.163.com/ubuntu/ jammy main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ jammy-backports main restricted universe multiverse

## Pre-released source, not recommended.
# deb http://mirrors.163.com/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ jammy-proposed main restricted universe multiverse
```

# 配置GIT

## 基础配置信息

```bash
$ git config --global user.name “username”
$ git config --global user.email “useremail”
```

## 生成ssh key

```bash
ssh-keygen -t rsa -C "youremail@example.com"
```

一路回车

公钥生成路径在

`~/.ssh/id_rsa.pub`

把这串内容贴到github的key上

添加一下刚刚生成的ssh key

```bash
ssh-add ~/.ssh/id_rsa
```

# 常用软件配置

## VIM

因为没有图形界面，所以vim的编辑成了首选。

1. 安装vim命令

   ` sudo apt install vim`

2. 主题安装

   vim ~/.vim/colors/monokai.vim

   ```bash
   " Vim color file
   " Converted from Textmate theme Monokai using Coloration v0.3.2 (http://github.com/sickill/coloration)
   set background=dark
   highlight clear
   if exists("syntax_on")
     syntax reset
   endif
   set t_Co=256
   let g:colors_name = "monokai"
   hi Cursor ctermfg=235 ctermbg=231 cterm=NONE guifg=#272822 guibg=#f8f8f0 gui=NONE
   hi Visual ctermfg=NONE ctermbg=59 cterm=NONE guifg=NONE guibg=#49483e gui=NONE
   hi CursorLine ctermfg=NONE ctermbg=237 cterm=NONE guifg=NONE guibg=#3c3d37 gui=NONE
   hi CursorColumn ctermfg=NONE ctermbg=237 cterm=NONE guifg=NONE guibg=#3c3d37 gui=NONE
   hi ColorColumn ctermfg=NONE ctermbg=237 cterm=NONE guifg=NONE guibg=#3c3d37 gui=NONE
   hi LineNr ctermfg=102 ctermbg=237 cterm=NONE guifg=#90908a guibg=#3c3d37 gui=NONE
   hi VertSplit ctermfg=241 ctermbg=241 cterm=NONE guifg=#64645e guibg=#64645e gui=NONE
   hi MatchParen ctermfg=197 ctermbg=NONE cterm=underline guifg=#f92672 guibg=NONE gui=underline
   hi StatusLine ctermfg=231 ctermbg=241 cterm=bold guifg=#f8f8f2 guibg=#64645e gui=bold
   hi StatusLineNC ctermfg=231 ctermbg=241 cterm=NONE guifg=#f8f8f2 guibg=#64645e gui=NONE
   hi Pmenu ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi PmenuSel ctermfg=NONE ctermbg=59 cterm=NONE guifg=NONE guibg=#49483e gui=NONE
   hi IncSearch term=reverse cterm=reverse ctermfg=193 ctermbg=16 gui=reverse guifg=#C4BE89 guibg=#000000
   hi Search term=reverse cterm=NONE ctermfg=231 ctermbg=24 gui=NONE guifg=#f8f8f2 guibg=#204a87
   hi Directory ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi Folded ctermfg=242 ctermbg=235 cterm=NONE guifg=#75715e guibg=#272822 gui=NONE
   hi SignColumn ctermfg=NONE ctermbg=237 cterm=NONE guifg=NONE guibg=#3c3d37 gui=NONE
   hi Normal ctermfg=231 ctermbg=235 cterm=NONE guifg=#f8f8f2 guibg=#272822 gui=NONE
   hi Boolean ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi Character ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi Comment ctermfg=242 ctermbg=NONE cterm=NONE guifg=#75715e guibg=NONE gui=NONE
   hi Conditional ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi Constant ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi Define ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi DiffAdd ctermfg=231 ctermbg=64 cterm=bold guifg=#f8f8f2 guibg=#46830c gui=bold
   hi DiffDelete ctermfg=88 ctermbg=NONE cterm=NONE guifg=#8b0807 guibg=NONE gui=NONE
   hi DiffChange ctermfg=NONE ctermbg=NONE cterm=NONE guifg=#f8f8f2 guibg=#243955 gui=NONE
   hi DiffText ctermfg=231 ctermbg=24 cterm=bold guifg=#f8f8f2 guibg=#204a87 gui=bold
   hi ErrorMsg ctermfg=231 ctermbg=197 cterm=NONE guifg=#f8f8f0 guibg=#f92672 gui=NONE
   hi WarningMsg ctermfg=231 ctermbg=197 cterm=NONE guifg=#f8f8f0 guibg=#f92672 gui=NONE
   hi Float ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi Function ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi Identifier ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=italic
   hi Keyword ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi Label ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi NonText ctermfg=59 ctermbg=236 cterm=NONE guifg=#49483e guibg=#31322c gui=NONE
   hi Number ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi Operator ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi PreProc ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi Special ctermfg=231 ctermbg=NONE cterm=NONE guifg=#f8f8f2 guibg=NONE gui=NONE
   hi SpecialComment ctermfg=242 ctermbg=NONE cterm=NONE guifg=#75715e guibg=NONE gui=NONE
   hi SpecialKey ctermfg=59 ctermbg=237 cterm=NONE guifg=#49483e guibg=#3c3d37 gui=NONE
   hi Statement ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi StorageClass ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=italic
   hi String ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi Tag ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi Title ctermfg=231 ctermbg=NONE cterm=bold guifg=#f8f8f2 guibg=NONE gui=bold
   hi Todo ctermfg=95 ctermbg=NONE cterm=inverse,bold guifg=#75715e guibg=NONE gui=inverse,bold
   hi Type ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi Underlined ctermfg=NONE ctermbg=NONE cterm=underline guifg=NONE guibg=NONE gui=underline
   hi rubyClass ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi rubyFunction ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi rubyInterpolationDelimiter ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi rubySymbol ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi rubyConstant ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=italic
   hi rubyStringDelimiter ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi rubyBlockParameter ctermfg=208 ctermbg=NONE cterm=NONE guifg=#fd971f guibg=NONE gui=italic
   hi rubyInstanceVariable ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi rubyInclude ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi rubyGlobalVariable ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi rubyRegexp ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi rubyRegexpDelimiter ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi rubyEscape ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi rubyControl ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi rubyClassVariable ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi rubyOperator ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi rubyException ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi rubyPseudoVariable ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi rubyRailsUserClass ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=italic
   hi rubyRailsARAssociationMethod ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi rubyRailsARMethod ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi rubyRailsRenderMethod ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi rubyRailsMethod ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi erubyDelimiter ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi erubyComment ctermfg=95 ctermbg=NONE cterm=NONE guifg=#75715e guibg=NONE gui=NONE
   hi erubyRailsMethod ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi htmlTag ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi htmlEndTag ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi htmlTagName ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi htmlArg ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi htmlSpecialChar ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi javaScriptFunction ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=italic
   hi javaScriptRailsFunction ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi javaScriptBraces ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi yamlKey ctermfg=197 ctermbg=NONE cterm=NONE guifg=#f92672 guibg=NONE gui=NONE
   hi yamlAnchor ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi yamlAlias ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   hi yamlDocumentHeader ctermfg=186 ctermbg=NONE cterm=NONE guifg=#e6db74 guibg=NONE gui=NONE
   hi cssURL ctermfg=208 ctermbg=NONE cterm=NONE guifg=#fd971f guibg=NONE gui=italic
   hi cssFunctionName ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi cssColor ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi cssPseudoClassId ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi cssClassName ctermfg=148 ctermbg=NONE cterm=NONE guifg=#a6e22e guibg=NONE gui=NONE
   hi cssValueLength ctermfg=141 ctermbg=NONE cterm=NONE guifg=#ae81ff guibg=NONE gui=NONE
   hi cssCommonAttr ctermfg=81 ctermbg=NONE cterm=NONE guifg=#66d9ef guibg=NONE gui=NONE
   hi cssBraces ctermfg=NONE ctermbg=NONE cterm=NONE guifg=NONE guibg=NONE gui=NONE
   
   ```

3. 安装插件管理

   ```bash
   curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
   ```

4. 配置vimrc

   vim ~/.vimrc

   ```bash
   "显示行号
   set nu
   
   "tab键宽度
   set tabstop=4
   
   ""语法高亮
   syntax enable
   
   "主题颜色
   colorscheme monokai
   
   "NERDTree快捷键
   map <C-n> :NERDTreeToggle<CR>
   
   "NERDTree剩下一个窗口时自动关闭
   autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
   
   "ctrlp配置
   set runtimepath^=~/.vim/plugged/ctrlp.vim
   call plug#begin('~/.vim/plugged')
   Plug 'itchyny/lightline.vim'
   Plug 'preservim/nerdtree'
   Plug 'kien/ctrlp.vim'
   Plug 'vim-airline/vim-airline'
   Plug 'jiangmiao/auto-pairs'
   Plug 'powerline/powerline'
   Plug 'preservim/nerdcommenter'
   Plug 'vim-scripts/a.vim'
   call plug#end()
   
   map <leader>r :NERDTreeFind<cr>
   
   "控制vim窗口分界线位置
   map <F2> <ESC><C-W>-
   map <F3> <ESC><C-W>+
   map <F4> <ESC><C-W><
   map <F5> <ESC><C-W>>  
   ```

   键入`source ~/.vimrc`

   进入vim，键入:Pluginstall，待插件安装完毕

## z.lua

性能很棒的目录跳转工具

[GitHub - skywind3000/z.lua: A new cd command that helps you navigate faster by learning your habits.](https://github.com/skywind3000/z.lua)

先安装lua

`apt install lua5.3`

之后再zshrc中配置该命令

`eval "$(lua /home/lehanbal/z.lua/z.lua  --init zsh once enhanced fzf)"`

相关的alias封装命令

```bash
alias zc='z -c'
alias zf='z -I'
alias zb='z -b'
alias zz='z -i'
alias zh='z -I -t .' 
```

具体请看zsh内容

## tmux

终端管理工具

`apt install tmux`

相关的alias命令封装

```bash
alias tns='tmux new-session -s ' #创建一个session
alias tkss='tmux kill-session -t ' #杀掉一个session
alias ta='tmux a -t ' #进入一个session
```

## ssh

配置ssh key，用密码连接不太安全

复制公钥到`~/.ssh/authorized_keys`

赋权authorized_keys文件 `chmod 644 authorized_keys`

也可以在已有sshkey的client上执行以下

`ssh-copy-id -i ~/.ssh/id_rsa.pub user@ip`(-i 参数表示指定sshkey的位置)

修改sshd配置`vim /etc/ssh/sshd_config`

```bash
# 开启密钥登入的认证方式
PubkeyAuthentication yes 
# 允许密码登录选项 yes为不禁用
PasswordAuthentication yes
```

修改完成后重启sshd服务

`systemctl restart sshd`

## ZSH

1. 安装zsh和ohmyzsh

   `sudo apt install zsh`

   `sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)" `

2. 修改主题是能显示目录绝对路径

   `vim ~/.oh-my-zsh/themes/对应主题`

   ```
   # 在PROMPT最后的地方加上
   PROMPT=${PROMPT/\%c/\%~}
   ```

3. 安装插件

   1. zsh-autosuggestions

      `git clone https://gitee.com/phpxxo/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`

   2. zsh-syntax-highlighting

      `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`

   3. extract

      自带，无需安装，加上即可

   4. zsh-completions

      `  git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions`

最后都需要在zshrc中的plugins=配置上对应的插件名字，请参考下文的zshrc配置文件

一切都修改完成后，请重新source配置文件

`source ~/.zshrc`

### zshrc

以下为个人的zshrc文件

```bash
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="af-magic"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment one of the following lines to change the auto-update behavior
# zstyle ':omz:update' mode disabled  # disable automatic updates
# zstyle ':omz:update' mode auto      # update automatically without asking
# zstyle ':omz:update' mode reminder  # just remind me to update when it's time

# Uncomment the following line to change how often to auto-update (in days).
# zstyle ':omz:update' frequency 13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# You can also set it to another string to have that shown instead of the default red dots.
# e.g. COMPLETION_WAITING_DOTS="%F{yellow}waiting...%f"
# Caution: this setting can cause issues with multiline prompts in zsh < 5.7.1 (see #5765)
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(
	git
	zsh-autosuggestions	
	zsh-syntax-highlighting
	extract
)

eval "$(lua /home/lehanbal/z.lua/z.lua  --init zsh once enhanced fzf)"    # ZSH 初始化

fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src

source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"

alias zc='z -c'
alias zf='z -I'
alias zb='z -b'
alias zz='z -i'
alias zh='z -I -t .'

alias tns='tmux new-session -s '
alias tkss='tmux kill-session -t '
alias ta='tmux a -t '
```



# DDNS配置

非管理员不需要配置

**该操作不支持IPV6**

该操作所需要准备

1. TokenID
2. Token
3. 域名ID
4. 记录ID
5. 子域名

## Token与TokenID

登录[阿里云API 密钥 ](https://console.dnspod.cn/account/token/token)创建DNSPod Token密钥，即可拿到tokenID与token

![image-20220725232619036](http://cdn.lehanbal.top/image-20220725232619036.png)

## 域名ID

登录[控制台](https://console.dnspod.cn/dns/list)，点击域名设置后可以看到域名信息，其中的DomainID即是域名ID

![image-20220725232829788](http://cdn.lehanbal.top/image-20220725232829788.png)

## 记录ID

通过[控制台](https://console.dnspod.cn/dns/list)获取 在管理页面找到操作日志，“值”后面括号内数字为记录ID

![image-20220725233133580](http://cdn.lehanbal.top/image-20220725233133580.png)

## 子域名

子域名为你要修改的A记录的子域名，自己设定

![image-20220725233251748](http://cdn.lehanbal.top/image-20220725233251748.png)

拿到以上信息后，就可以拼接出ddns的参数信息了

```bash
curl -X POST https://dnsapi.cn/Record.Ddns -d 'login_token=TokenID,Token&domain_id=域名ID&record_id=记录ID&record_line=默认&sub_domain=子域名&lang=en'
```

## 脚本定时操作

### 设计思路

1. 将当前的ip记录在一个文件中
2. 每隔一分钟检查一次当前ip与文件中的ip是否一直，若不一致则进行ddns操作

### 安装cron

`sudo apt install cron`

开启cron服务

`systemctl start cron`

允许自启服务

`systemctl enable cron`



### 脚本内容

首先创建/etc/shell/tmpddns文件，并赋予可读权限

```bash
cd /etc
sudo mkdir shell
sudo touch tmpddns
sudo chmod 777 tmpddns
echo -1 > tmpddns
```

创建脚本

`touch ddns`

`sudo chmod +x ddns`

```bash
#!/bin/zsh

IP_PREV=$(cat /etc/shell/tmpddns)
IP_NOW=$(curl -s ip.sb)

if [ $IP_PREV != $IP_NOW ];then
        echo $IP_NOW > /etc/shell/tmpddns
        curl -X POST https://dnsapi.cn/Record.Ddns -d 'login_token=TokenID,Token&domain_id=域名ID&record_id=记录ID&record_line=默认&sub_domain=子域名&lang=en'
fi
```

### 设定定时任务

`vim /etc/crontab`

```bash
*/1 * * * * root /etc/shell/ddns
```

表示每一分钟都会使用root用户去执行/etc/shell/ddns脚本