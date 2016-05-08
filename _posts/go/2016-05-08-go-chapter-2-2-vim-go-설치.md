---
layout: post
title: 디스커버리 Go 2 - vim-go
excerpt: "Go"
tags: [Go, GOROOT, GOPATH]
comments: true
---

### Overview

vim에서 go언어로 코딩을 하기 위한 Setting을 정리합니다.

### Vundle

vim-go를 설치하기에 앞서 vim plug-in 관리를 위해 `Vundle`을 설치하도록 하겠습니다.

*Vundle*은 간단히 말해 vim plug-in의 설치, 삭제와 같은 행위들을 도와주는 도구입니다.

> 이 또한 vim plug-in입니다.

설치법은 아래와 같습니다.

1. 먼저 *Vundle*을 clone 받습니다.  

	```bash
	$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
	```

2. *~/.vimrc*에 아래 내용을 추가합니다.

	```vim
	set nocompatible              " be iMproved, required
	filetype off                  " required

	" set the runtime path to include Vundle and initialize
	set rtp+=~/.vim/bundle/Vundle.vim
	call vundle#begin()
	" alternatively, pass a path where Vundle should install plugins
	"call vundle#begin('~/some/path/here')

	" let Vundle manage Vundle, required
	Plugin 'VundleVim/Vundle.vim'

	" All of your Plugins must be added before the following line
	call vundle#end()            " required
	filetype plugin indent on    " required
	" To ignore plugin indent changes, instead use:
	"filetype plugin on
	"
	" Brief help
	" :PluginList       - lists configured plugins
	" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
	" :PluginSearch foo - searches for foo; append `!` to refresh local cache
	" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
	"
	" see :h vundle for more details or wiki for FAQ
	" Put your non-Plugin stuff after this line
	```

Vundle 설치과 완료되었습니다.

vim을 실행하고 `:PluginInstall`을 입력하면 정상적으로 실행되는 것을 알 수 있습니다.


### vim-go 설치

1. *~/.vimrc* 에서 *call vundle#begin* 과 *call vundle#end()* 사이에 아래 문구를 추가합니다.

	```vim
	Plugin 'faith/vim-go'
	```

2. vim을 실행하고 아래 명령어를 입력하면 vim-go 설치가 완료됩니다.

	```vim
	:PluginInstall
	```

3. 또한 아래 명령어로 몇가지 vim-go 사용을 위해 필요한 바이너리들을 설치할 수 있습니다.(vim에서 입력하며 설치를 매우 권장함)

	```vim
	:GoInstallBinaries
	```


### vim-go

vim-go에서 사용할 수 있는 기능은 아래와 같습니다.

* **GoRun**: 현재 파일 실행
* **GoDef**: 선언부로 이동
* **GoDoc**: Go 문서(API document)
* **GoImports**: 자동으로 패키지 import
* **GoLint**: 현재 파일 Lint
* **GoVet**: 현재 파일 정적 분석
* **GoImplements, GoCallees, GoReferrers...**
* **GoTest**
* **GoBuild**
* **GoInstall**

자질 구레한(?) 기능으로는 아래 2가지 기능을 제공합니다.

* Go 파일을 저장하면 자동으로 gofmt가 실행됩니다.
* `Ctrl` + `x` + `o`: Auto completion 기능




### reference

[VundleVim/Vundle.vim: Vundle, the plug-in manager for Vim](https://github.com/VundleVim/Vundle.vim)
[fatih/vim-go: Go development plugin for Vim](https://github.com/fatih/vim-go)

