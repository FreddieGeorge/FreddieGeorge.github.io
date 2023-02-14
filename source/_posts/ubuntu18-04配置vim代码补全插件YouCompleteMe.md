---
title: ubuntu18.04配置vim代码补全插件YouCompleteMeu
date: 2023-02-09 16:12:08
tags:
- ubuntu
- vim
- Python
categories: 
- Linux
excerpt: 配置YouCompleteMe的踩坑经历

---

## 背景

这几天都在忙碌公司的项目，没空更新其他博客，但是也在忙中偷闲把工作虚拟机的vim界面给配置了一下，安装了nerdtree等好用的插件，用起来舒服多了。

然后也了解到了强大的代码补全插件 YouCompleteMe 插件，便想着配置一下。结果这玩意弄了一两天。今天也在这里开个博客记录一下。

> 这里一开始是用的--all想要一步到位，但是后面出现了golang的bug，我又用不上go，所以后面换--clang-completer
> 所以有些bug不一定能复现出来

## 环境介绍

虚拟机：VMWare 16.1.1

ubuntu18.04

vim 9.0

vim 插件管理工具 ：vundle

## 拉取源码

> 首先如果有代理的话最好使用代理，不然拉下源码等过程非常漫长，我主机用的clash，在clash里面打开Allow lan就可以在虚拟机配置代理了

Vundle有多种[拉取源码的方式](https://github.com/VundleVim/Vundle.vim#quick-start),直接在vundle#begin() 和vundle#end() 之间加入一句 `Plugin 'Valloric/YouCompleteMe'`,然后在vim中敲入 `:PluginInstall` 即可自动下载插件。

## 编译


YouCompleteMe(以下简称YCM)需要python编译，在git 拉下YCM的源码之后还需要输入 `git submodule update --init --recursive`添加子模块 **ycmd**。
 
在添加完成之后输入 `python3 ./install.py --clang-completer` 直接失败,提示 `ycmd requires Python >= 3.8.0`,而ubuntu18.04的默认python版本是python3.6.9。那好吧，我们来配置一下python3.8的环境

### python3.8问题

首先，我强烈建议ubuntu18.04中的python3的默认版本不要随便修改，否则可能会有很多问题。

按照[这个博客](https://www.jianshu.com/p/55a2a009fc1e)很快就配置好了3.8的环境，这里我并没有直接修改软链接为python，而是选择直接绝对路径调用python3.8。

配置好之后再次来到YCM的路径，输入`/usr/local/python3/bin/python3.8 ./install.py --clang-completer`，然后再次报错

```shell
found static Python library (/usr/local/python3/lib/libpython3.8.a) but a dynamic one is required. You must use a Python compiled with the --enable-shared flag. If using pyenv, you need to run the command:
  export PYTHON_CONFIGURE_OPTS="--enable-shared"
```

好嘛，意思就是我编译出来的python3.8是静态库`.a`,而YCM需要动态库`.so`，那我只能再回到python3.8的源码目录，在./configure 后面加了一串 `--enable-shared`.终于编译出来了动态库`libpython3.8.so.1.0`,并且把这个文件复制到了 /usr/lib下面。

再再再回到YCM的目录，执行`/usr/local/python3/bin/python3.8 ./install.py --clang-completer`，还在报错。好在这次不是python的问题，这次是g++的问题。

### g++问题

报错信息为：

```shell
CMake Error at CMakeLists.txt:232 (message):
  Your C++ compiler does NOT fully support C++17.
```

那应该就说g++的版本有问题，再次按照[这个回答](https://stackoverflow.com/questions/65284572/your-c-compiler-does-not-fully-support-c17)输入以下指令更改了g++版本

```shell
sudo apt-get install g++-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 700 --slave /usr/bin/g++ g++ /usr/bin/g++-7
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8
```

继续编译，g++问题解决

### Clang 问题

还没完，还有报错

```shell
CMake Error at ycm/CMakeLists.txt:115 (file):
  file DOWNLOAD HASH mismatch
```

累了，直接找到github的issue找到[这个问题](https://github.com/ycm-core/YouCompleteMe/issues/1711#issuecomment-145131863),按照这个回答，我们把[他提供的下载链接](http://llvm.org/releases/3.7.0/clang+llvm-3.7.0-x86_64-linux-gnu-ubuntu-14.04.tar.xz)对应的文件下载下来，然后放入`~/.vim/bundle/YouCompleteMe/third_party/ycmd/clang_archives`中，不需要解压，然后重新执行 install.py,发现还是在下载，仔细一看这个回答都已经是2015年的回答了。于是继续找原因。

> 奇怪的是，错误之后我就下班了，今天上班重新执行了install，一样的指令，这次就通过了，想不明白为什么。。。

<
我又不用golang，
### golang环境问题

继续报错。

```text
can't load package: package golang.org/x/tools/gopls@v0.9.4
//....
package golang.org/x/tools/gopls@v0.9.4: unrecognized import path "golang.org/x/tools/gopls@v0.9.4" 
```

应该是golang的环境没有配，看来一下应该是缺少gopls，尝试手动下载，在终端输入`go get golang.org/x/tools/gopls`,等待一段时间后报错`dial tcp 142.251.42.241:443: i/o timeout`，那大概知道什么问题了。再次尝试ping这个ip，果然连接不上。奇怪的是我明明有配置代理，在主机端（也就是win下）ping这个ip，还是失败。

然后我选择直接直接--all换--clang-completer，直接通过编译，然后打开vim输入:PluginInstall  即可

至此安装完成
