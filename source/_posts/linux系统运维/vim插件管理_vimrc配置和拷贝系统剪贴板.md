---
title: vim插件管理_vimrc配置和拷贝系统剪贴板
tags:
  - vim
  - linux
abbrlink: 1b9bff9b
categories:
  - linux系统运维
date: 2018-05-15 22:06:13
---


### vim-plug 安装

```
https://github.com/junegunn/vim-plug

:PlugInstall 执行安装命令
```


### vim 拷贝到系统剪贴板

1. which vim可以看到当前使用的vim是哪个，vim --version可以看到当前使用的vim支持哪些feature，'+'前缀表示拥有的feature，'-'前缀表示未拥有；

2. '+clipboard'是支持使用系统剪切板的feature；如果你当前使用的vim不支持clipboard，那需要brew upgrade vim装一个新的；

3. 安装新的以后，要把这个新的vim设置为默认vim，通常使用alias设置一下别名，或者通过环境变量设置，或者删掉旧的，做个软连接；

4. 确认+clipboard以后，在.vimrc文件中加入set clipboard=unamed，就可以在vim中使用系统剪切板了


<!-- more -->


### 我的.vimrc

```
" Plugin 
call plug#begin()
Plug 'tomasr/molokai' "主题
Plug 'ctrlpvim/ctrlp.vim' "快速查文件
Plug 'Shougo/neocomplete.vim' "代码实时提示
Plug 'majutsushi/tagbar' "tagbar
Plug 'scrooloose/nerdtree' "导航树
Plug 'jistr/vim-nerdtree-tabs' "导航树插件
Plug 'vim-airline/vim-airline' "状态栏
Plug 'ervandew/supertab' "supertab
Plug 'SirVer/ultisnips' "代码模板
Plug 'Valloric/YouCompleteMe' "table补全
Plug 'easymotion/vim-easymotion' "极速跳转
call plug#end()


" mapping
let mapleader=","
set mouse=a "可以用鼠标拖动
set clipboard=unnamed "鼠标选中y复制


" easymotion
let g:EasyMotion_smartcase = 1
let g:EasyMotion_startofline = 0 " keep cursor column when JK motion
nmap s <Plug>(easymotion-overwin-f)
nmap s <Plug>(easymotion-overwin-f2)
map  / <Plug>(easymotion-sn)
omap / <Plug>(easymotion-tn)
map  n <Plug>(easymotion-next)
map  N <Plug>(easymotion-prev)
map <Leader>j <Plug>(easymotion-j)
map <Leader>k <Plug>(easymotion-k)
map <Leader>w <Plug>(easymotion-w)
map <Leader>b <Plug>(easymotion-b)



" Color 
syntax enable
set t_Co=256
let g:rehash256 = 1
let g:molokai_original = 1
colorscheme molokai

" Ycm
"let g:ycm_server_use_vim_stdout = 1
"let g:ycm_server_log_level = 'debug'
" make YCM compatible with UltiSnips (using supertab)
let g:ycm_key_list_select_completion = ['<C-n>', '<Down>']
let g:ycm_key_list_previous_completion = ['<C-p>', '<Up>']
let g:SuperTabDefaultCompletionType = '<C-n>'
let g:UltiSnipsExpandTrigger = "<tab>"
let g:UltiSnipsJumpForwardTrigger = "<tab>"
let g:UltiSnipsJumpBackwardTrigger = "<s-tab>"


" Nerdtree
"autocmd vimenter * NERDTree "自动打开Tree
"autocmd vimenter * Tagbar "自动打开TagBar
nmap <F7> :NERDTreeToggle<CR>
nmap <F8> :TagbarToggle<CR>
let NERDTreeWinSize=20 "设置nerdtree宽度
let g:tagbar_width=20 "设置宽度，默认为40
"let g:nerdtree_tabs_open_on_console_startup=1 "启动打开nerdtree
let g:neocomplete#enable_at_startup = 1 "代码实时提示
"let NERDTreeQuitOnOpen=1 "打开文件后自动关闭窗口

" 打开NERDTree,定位到当前文件
noremap <silent> <Leader>f :NERDTreeFind<cr> 
"打开tagbar窗口,跳转后自动关闭,q不跳转直接关闭
noremap <silent> <Leader>g :TagbarOpen fjc<cr> 
" NERDTree tab switch
map  <C-l> :tabn<CR>
map  <C-h> :tabp<CR>
map  <C-o> :tabnew<CR>


" config
set tabstop=4                   " 设定tab长度为4
set shiftwidth=4                " 缩进的空格数为4

filetype off                    " Reset filetype detection first ...
filetype plugin indent on       " ... and enable filetype detection
set nocompatible                " Enables us Vim specific features
set ttyfast                     " Indicate fast terminal conn for faster 
set ttymouse=xterm2             " Indicate terminal type for mouse codes
set ttyscroll=3                 " Speedup scrolling
set laststatus=2                " Show status line always
set encoding=utf-8              " Set default encoding to UTF-8
set autoread                    " Automatically read changed files
set autoindent                  " Enabile Autoindent
set backspace=indent,eol,start  " Makes backspace key more powerful.
set incsearch                   " Shows the match while typing
set hlsearch                    " Highlight found searches
set noerrorbells                " No beeps
set number                      " Show line numbers
set showcmd                     " Show me what I'm typing
set noswapfile                  " Don't use swapfile
set nobackup                    " Don't create annoying backup files
set splitright                  " Vertical windows should be split to right
set splitbelow                  " Horizontal windows should split to bottom
set autowrite                   " Automatically save before :next, :make etc.
set hidden                      " Buffer still exist if window is closed
set fileformats=unix,dos,mac    " Prefer Unix over Windows over OS 9 formats
set noshowmatch                 " Do not show matching brackets by flickering
set noshowmode                  " We show the mode with airline or lightline
set ignorecase                  " Search case insensitive...
set smartcase                   " but not it begins with upper case
set completeopt=menu,menuone    " Show popup menu, even if there is one entry
set pumheight=10                " Completion window max size
set nocursorcolumn              " Dont highlight column
set nocursorline                " Dont highlight cursor
set lazyredraw                  " Wait to redraw
set cursorcolumn
```