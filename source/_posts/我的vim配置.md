---
title: 我的vim配置
date: 2023-02-14 08:49:14
tags:
- vim
categories: 
- Linux
excerpt: 本文记录了我的简单vim配置
---

## 前言

我很多时间都是在使用vscode，vim也只会一些基础的操作，但是这段时间看了几个博主的vim界面，觉得vim也有它很强大好看的地方，便决定给自己的vim简单配置一下。

本文基本上都是按照官方文档在配置，也参考了很多博客。

> 这篇没啥技术含量，基本就是看一下官方的README对着改，然后也没有自己额外配置，属于很基础简单的配置

## 配置环境

Ubuntu18.04(经试验Manjaro同样适用)

vim 9.0


## 基础配置

先按照我的个人喜好将vim的原生配置设置一下。网上也有很多相关的教程配置。我的vim配置已经放到了我的[github仓库](https://github.com/FreddieGeorge/my_vim.git)

官方的配置文档可以在vim命令模式输入`:h option-list`查看完整的选项列表。

在我的`~/.vimrc`中,最开始的配置如下，主要作用和功能都有写注释

```text
set showcmd         " 显示指令
set showmode        " 显示当前模式

set encoding=utf-8  " 设置编码格式

set laststatus=2    " 状态栏
set number          " 显示行号
set showmatch       " 显示括号匹配
set hlsearch        " 高亮搜索结果
set ruler           " 显示光标当前位置

set tabstop=4       " tab长度
set autoindent      " 继承前一行的缩进方式
set shiftwidth=4    " 设置自动缩进长度
set expandtab       " 空格替换tab
set softtabstop=4   " 退格键一次删除4个空格


" 启用鼠标
set mouse=a
set selection=exclusive
set selectmode=mouse,key

" 设置上下滚动保留行数为5
set scrolloff=5
" 禁止创建交换文件
set noswapfile

" 设置自动切换目录
set autochdir

" true color
set termguicolors

```
## 插件管理

插件管理我使用的是[Vundle](https://github.com/VundleVim/Vundle.vim),从仓库的README中也可以看到它的安装方法，具体为：

1. 输入`mkdir ~/.vim/bundle`
2. `git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`
3. 编辑`~/.vimrc`,在上面的配置后添加如下语句
    ```text
    set nocompatible              " be iMproved, required
    filetype off                  " required

    " set the runtime path to include Vundle and initialize
    set rtp+=~/.vim/bundle/Vundle.vim
    call vundle#begin()
    " alternatively, pass a path where Vundle should install plugins
    "call vundle#begin('~/some/path/here')

    " let Vundle manage Vundle, required
    Plugin 'VundleVim/Vundle.vim'

    " ==========================
    " Plugin Place
    " ==========================

    " All of your Plugins must be added before the following line
    call vundle#end()            " required
    filetype plugin indent on    " required
    ```
4. 之后需要安装的插件只需要放在`vundle#begin()`和`vundle#end()`之间即可

## 安装插件

### 主题插件

主题插件使用的是很经典的[vim-colors-solarized](https://github.com/altercation/vim-colors-solarized),只需要在`vundle#begin()`和`vundle#end()`之间加入`Plugin 'altercation/vim-colors-solarized'`，然后在vim中输入`:PluginInstall`即可自动下载。

在`~/.vimrc`中添加配置如下

> 建议注释好，做好分类


```text
let g:solarized_termtrans = 1 " use terminal background
let g:solarized_visibility = "high"

" gui下为light，其他是dark
if has('gui_running')
    set background=light
else
    set background=dark
endif

" 主题设置为solarized
colorscheme solarized

```

### 目录插件

目录插件使用的是[NERDTree](https://github.com/preservim/nerdtree),安装方法在官方README中都很详细了

`~/.vimrc`中

```text
call vundle#begin()

  " ... ...
  Plugin 'preservim/nerdtree'

call vundle#end()
```

neadtree配置上我是安装README配置的，具体如下，更多详细使用方法这里不展开，自行查看文档或者资料

```text
" Start NERDTree and put the cursor back in the other window.
" vim 打开时自动打开nerdtree但是不聚焦在nerd中
autocmd VimEnter * NERDTree | wincmd p

"  Exit Vim if NERDTree is the only window remaining in the only tab.
" 退出vim时如果只剩下nerdtree窗口，nerdtree也退出
autocmd BufEnter * if tabpagenr('$') == 1 && winnr('$') == 1 && exists('b:NERDTree') && b:NERDTree.isTabTree() | quit | endif

" If another buffer tries to replace NERDTree, put it in the other window, and bring back NERDTree.
" 防止nerdtree被替代，新窗口自动在nerd之外的窗口打开
autocmd BufEnter * if bufname('#') =~ 'NERD_tree_\d\+' && bufname('%') !~ 'NERD_tree_\d\+' && winnr('$') > 1 |
    \ let buf=bufnr() | buffer# | execute "normal! \<C-W>w" | execute 'buffer'.buf | endif

" F2 打开NERDTree
map <F2> :NERDTreeToggle<CR>
let NERDTreeWinSize=30
" 靠左显示
let NERDTreeWinPos="left"
```

### 状态栏插件

使用[vim-airline](https://github.com/vim-airline/vim-airline),配置如下

```text

" ~/.vimrc

call vundle#begin()

  " ... ...
  Plugin 'vim-airline/vim-airline'
  Plugin 'vim-airline/vim-airline-themes'

call vundle#end()

" ... ...

" ============= vim-airline ============
" 字体
let g:airline_powerline_fonts=1
" 主题
let g:airline_theme='minimalist'
```

### 代码补全插件

详见[ubuntu18.04配置vim代码补全插件YouCompleteMeu](https://freddiegeorge.github.io/2023/02/09/ubuntu18-04%E9%85%8D%E7%BD%AEvim%E4%BB%A3%E7%A0%81%E8%A1%A5%E5%85%A8%E6%8F%92%E4%BB%B6YouCompleteMe/)


### tagbar插件

使用[tagbar](https://github.com/preservim/tagbar),配置如下

> 这个暂时没有用上，不知道具体好不好用，应该是跟vscode的大纲类似的东西

```text

" ~/.vimrc

call vundle#begin()

  " ... ...
  Plugin 'majutsushi/tagbar'

call vundle#end()

" ... ...

" ============= tagbar ============
let g:tagbar_width=35
let g:tagbar_autofocus=1
let g:tagbar_left=1 " tagbar放在窗口左边
" tagbar启动绑定到F3
nmap <F3> :TagbarToggle<CR>  
```

### goyo插件

[goyo](https://news.itsfoss.com/configuring-vim-writing/)还是我在[这篇文章](https://news.itsfoss.com/configuring-vim-writing/)看到的，觉得挺有意思，也加上去了。goyo主要还是给写作这种用途使用的，用来专注写作，比如写博客，写小说搭配同一个作者的[limelight](https://github.com/junegunn/limelight.vim)效果很不错。但是我这里并没有安装limelight，主要是在ubuntu下面的goyo模式上下会有黑边，在看了issue之后也还没找到解决方法，所以暂时先不管，晚上试试manjaro下效果会不会好一点。

安装和使用也很简单，在官方的README中就有

```text
" ~/.vimrc

call vundle#begin()

  " ... ...
  Plugin 'junegunn/goyo.vim'

call vundle#end()

```

### 注释插件

注释插件使用[NERDTree](https://github.com/preservim/nerdtree)

```text
" ~/.vimrc

call vundle#begin()

  " ... ...
  Plugin 'preservim/nerdcommenter'

call vundle#end()

" ... ...
"============ NERDCommenter ======
" Create default mappings
let g:NERDCreateDefaultMappings = 1

" Add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

" Add your own custom formats or override the defaults
let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Enable NERDCommenterToggle to check all selected lines is commented or not
let g:NERDToggleCheckAllLines = 1

```
## 总结

最后的配置成果还是挺不错的，但是我对vim还是不够熟悉，仅限于一些基本的增删改查操作，目前也没有这个应用习惯，写项目还是使用vscode居多，接下来在我的Manjaro中尽量多尝试一下vim的操作，熟悉一下吧。
