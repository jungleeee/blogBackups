---
layout  : post
title   : vim 常用插件配置
data    : 2017/9/12
tags    : VIM
comment : false
reward  : false
brief   :

---
### 前言

```
        学习、总结，交流，进步！
        插件与脚本使得VIM强大，毋庸置疑！
```

全程参考:[ 所需及所获:像使用IDE一样vim](https://github.com/yangyangwithgnu/use_vim_as_ide)
其余参考将会在文中相关部分列出。
文中内容如有错误，请指正！
原创，转载请注明出处！

---
### Ready

VIM的安转请自行进行。版本最好在7.5以上。
请检查用户目录下是否存在: `.vim/`文件夹和`.vimrc`用户配置文件，可以使用命令 `ls -l ~` 查看。
`.vim/`文件夹用于存放后续所有插件。
`.vimrc`用于配置VIM功能，包括基本语法、插件配置以及部分脚本等。
如果没有请自行创建。

<!-- more -->

---
### 基本配置

可以直接添加到`.vimrc`文件中:

```
" General set
    " .vimrc 文件保存后自动生效
    autocmd! bufwritepost .vimrc source %   " linux
    "let g:mapleader=";"            " vim 前缀键设置，默认为: \
    set autowrite                   " auto save file
    set nobackup                    " 不产生临时文件
    set noswapfile                  " 不产生临时文件
    "set t_ti= t_te=                 " 退出VIM时，将当前屏幕内容显示到终端屏幕上

    " 不使用鼠标，方向键。实测插入模式中还是可以使用
    "set mouse-=a
    "map <Left>  <Nop>
    "map <Right> <Nop>
    "map <Up>    <Nop>
    "map <Down>  <Nop>

    " 普通模式时使用ctrl+h/j/k/l 实现多文件编辑时窗口切换
    nnoremap <C-j> <C-W>j
    nnoremap <C-k> <C-W>k
    nnoremap <C-h> <C-W>h
    nnoremap <C-l> <C-W>l

    " 插入模式时使用ctrl+h/j/k/l 实现光标移动
    inoremap <C-j> <Down>
    inoremap <C-k> <Up>
    inoremap <C-h> <Left>
    inoremap <C-l> <Right>
```

---
### 插件管理

[Vundle](https://github.com/VundleVim/Vundle.vim)是一个行之有效的插件，接管`.vim/`文件夹，为每一个插件指定唯一目录。更清晰、明了。
[Vundle](https://github.com/VundleVim/Vundle.vim)会接管`.vim/`文件夹下的所有目录，所以先清空。在通过命令安装Vundle:

```
    git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

然后在`.vimrc`中增加配置:

```
" vundle set
    filetype off
    set rtp+=~/.vim/bundle/Vundle.vim
    call vundle#begin()
    Plugin 'VundleVim/Vundle.vim'
    Plugin 'altercation/vim-colors-solarized'
    Plugin 'tomasr/molokai'
    "Pluglin 'vim-scripts/phd'
    "Plugin 'Lokaltog/vim-powerline'
    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'
    "Plugin 'nathanaelkane/vim-indent-guides'
    Plugin 'Yggdroot/indentLine'
    Plugin 'derekwyatt/vim-fswitch'
    Plugin 'majutsushi/tagbar'
    Plugin 'vim-scripts/indexer.tar.gz'
    Plugin 'vim-scripts/DfrankUtil'
    Plugin 'vim-scripts/vimprj'
    Plugin 'easymotion/vim-easymotion'
    "Plugin 'mileszs/ack.vim'
    Plugin 'dyng/ctrlsf.vim'
    Plugin 'terryma/vim-multiple-cursors'
    Plugin 'Yggdroot/LeaderF'
    Plugin 'scrooloose/nerdcommenter'
    Plugin 'ShowTrailingWhiteSpace'
    Plugin 'SirVer/ultisnips'
    "Plugin 'honza/vim-snippets'
    Plugin 'Valloric/YouCompleteMe'
    Plugin 'scrooloose/nerdtree'
    Plugin 'fholgado/minibufexpl.vim'
    Plugin 'Tabular'                    " automatic alignment
    Plugin 'plasticboy/vim-markdown'    "
    Plugin 'iamcco/markdown-preview.vim'
    "Plugin 'iamcco/mathjax-support-for-mkdp'
    call vundle#end()

    filetype indent on              " different file smart indent
    filetype plugin on              " load the corrrsponding plug-in for different f
    filetype plugin indent on       " open antomatic completion
```

vundle管理的插件必须位于`call vundle#begin()` 和 `call vundle#end()`之间，其中每项

```
    Plugin 'VundleVim/Vundle.vim'
```

对应一个插件，后续如有新的插件，只需要追加到该列表中。安装新插件时需要在Github上找到插件地址，追加到列表中保存`.vimrc`文件，然后进入到VIM命令模式，执行:

```
    PluginInstall
```

便可以通过vundle自动安装插件。

需要卸载插件时，需要在`.vimrc`文件中注释该插件或者删除，保存`.vimrc`文件后，在命令模式执行:

```
    PluginClean
```

即可删除该插件。关于插件更新，直接在命令模式下，执行:

```
    PluginUpdate
```

即可更新整个插件列表。

关于插件: 多数插件在Github上都有不同的下载源，优先选取该插件的作者的下载源。

---
### UI Theme

界面优化，主题配置:

```
" Vim-UI set {
    " theme {很实用的两款:solarized、molokai
        set background=dark
        let g:solarized_termtrans=256
        colorscheme solarized
        "colorscheme molokai
        "set guifont =              " set default font
    " }
    " code fold {代码折叠的方式，method: manual, indent, expr, syntax, diff, marker
        "set foldenable             " 启动VIM时，折叠
        set nofoldenable            " 启动VIM时，不折叠
        set foldmethod=syntax       " 基于语法进行折叠
        "set foldmarker={,}
    " }
    "  search {搜索
        set incsearch               "
        set hlsearch                " 搜索结果高亮
        set ignorecase              " 忽略大小写
    "  }

    syntax enable                   " 语法高亮
    syntax on                       "

    set expandtab                   " 使用空格键代替TAB键
    set tabstop=4                   " TAB键的宽度
    set shiftwidth=4                " TAB键4个空格键
    "set softtabstop=4              " 把4个空格键自动识别为疑个TAB键

    set laststatus=2                " 总是显示状态栏
    set ruler                       " 打开状态标尺
    set number                      " 显示行号

    set nowrap                      "
    set autoindent                  " 自动缩进
    set cursorline                  " 高亮当前行
    "set cursorcolumn               " 高亮当前列
    set showmatch                   " highlighting the matching brack

    "set gcr=a:block-blinon0        " 禁止光标闪烁

    "set guioptions-=l              " ban on the scroll bar dispaly
    "set guioptions-=L
    "set guioptions-=r
    "set guioptions-=R

    "set guioptions-=m              " ban on the menu bar dispaly
    "set guioptions-=M
" }
```

###### 主题

通过前面插件列表下载了solarized、molokai两款主题，可以通过注释其中一款而是用另一款。
在实际使用中molokai可以直接使用，而主题solarized使用时，背景配色异常，最后通过以下方式正常使用（安装后能够正常使用的请忽略）。

`
    选取终端菜单栏中`Edit(编辑)`栏，最后一项`Profile Preference`，在弹出界面框中选中`color`项，如下图所示:
`

![solarized](/picture/img/vim-plug-in/1_solarized.png)

###### 代码折叠

文件代码量大时，可能会对分析代码产生干扰，或者通过折叠函数调用与定义之间的代码，可以更加直观的理解函数功能。
VIM自身支持多种折叠：手动折叠(manual)、基于缩进进行折叠(indent)、基于语义进行折叠(syntax)、未更改文本构成折叠等等。
其中indent、syntax较为适合编程。此段摘抄 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide) 4.3节。

`
    操作: za: 打开或关闭当前折叠；zM: 关闭多有折叠；zR: 打开多有折叠。
`
###### 其他

VIM基本配置项很多，本人并没有进行太多设置，具体设置项的作用或者功能，请参考其他`.vimrc`文件，或者对该项
Search，基本上都能够得到解释，在此不过多赘述。将会在文末贴上部分其他作者的`.vimrc`文件，以供参考。

---
### 插件详细

###### 状态栏

```
    set laststatus=2                " 总是显示状态栏

    "Plugin 'Lokaltog/vim-powerline'
    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'
```

[vim-powerline](https://github.com/Lokaltog/vim-powerline) 和 [vim-airline](https://github.com/vim-airline/vim-airline) 都是状态栏美化插件，选用其一，注释另外一个。[vim-airline-theme](https://github.com/vim-airline/vim-airline-theme) 提供 `vim-airline` 的主题，
在其路径 `vim-airline-theme/doc/` 下文件 `airline-themes.txt` 有对于可使用的主题的介绍。

状态栏配置，在`.vimrc`文件中添加以下内容:

```
    " set powerline theme style
    let g:Powerline_colorscheme='solarized256'
```

当选择`vim-powerline`作为状态栏主题时，添加上述内容；选择`vim-airline`时，添加以下内容:

```
    " set airling theme style
    let g:airline_theme='powerlineish'
```

###### 可视化缩进

```
    Plugin 'Yggdroot/indentLine'
    "Plugin 'nathanaelkane/vim-indent-guides'
```

很多代码规范中建议代码嵌套最多不能超过3层，但难免有更多层出现。[
vim-indent-guides ](https://github.com/nathanaelkane/vim-indent-guides)插件提供可视化缩进，将相同缩进的代码关联起来，`vim-indent-guides`通过识别制表符来绘制缩进连接线。在`.vimrc`文件中添加如下配置:

```
    " set indent guides
    let g:indent_guides_enable_on_vim_startup=1 " 打开VIM时跟随启动
    let g:indent_guides_start_level=2           " 从第二层开始可视化缩进
    let g:indent_guides_guide_sizw=1            " 色块宽度
```

本人配置完成后使用，发现色块宽度占了4个空格，原因未查。

另外一个插件[indentLine](https://github.com/Yggdroot/indentLine) ，在显示可视化缩进时，只有一根细线，甚和我意。推荐一哈。
后面还有一款他写的插件`LeaderF`，提供文件搜索功能，后面在介绍。`indentLine`基本上可以不用配置，但提供几个可选用配置项。

```
    " set indentLine
    "let g:indentLine_enable = 1                    " 默认就是开启的
    "let g:indentLine_char = '┆' '│'  '⎸' '▏' '¦'   " 配置可视化缩进时的显示符号，选其一
    "let g:indentLine_color_term = 239              " 参看作者的解释
    "map <C-i> :IndentLineToggle<CR>                " 映射快捷键用来开启/关闭可是话缩进
```

![indentLine](/picture/img/vim-plug-in/2_indentLine.png)

###### 文件切换

```
    Plugin 'derekwyatt/vim-fswitch'
```

[vim-fswitch](https://github.com/derekwyatt/vim-fswitch) 用来实现在源文件和头文件中快速切换，但需要保证文件名相同，比如`main.c`和`main.h`。需要在`.vimrc`文件中添加:

```
    " *.c** & *.h, fast switch
    nmap <M-t> :FSHere<CR>      " 切换文件 *.c/cpp/.. & *.h
```

我在这里使用了`Alt+t`组合键，`M`代表`Alt`键。除掉苹果键盘，`Alt`键就在大拇指下，个人觉得这个键，使用很方便。插播一下`Alt`键说明:

```
    我选择这个键只有一个原因，就是觉得的不用起来，好浪费。但是实际使用起来bug有比较多，所以。。。我选择了将就。
```

###### 标签系统

代码中的变量、宏、结构、枚举、类、函数、对象等等这些统称为标识符，每个标识符的定义、所在文件的行位置、所在文件的路径等信息就是标签（tag)。
此段摘抄 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  4.6节。

`Exuberant Ctags`就是一款经典的用于生成代码标签信息的工具，支持数十种语言。请在终端中执行命令安装。安装完成后，在代码目录中执行:

```
    ctags -R
```

将会在当前目录下生成`tag`文件，包含了当前目录下所有文件的标识符集合。

标签除了可以很直观展示当前文件中的各种信息，还能提供强大的导航和补全功能，接下来将会一一介绍。

在生成`tag`文件后，可以通过插件 [tagbar](https://github.com/majutsushi/tagbar) 把从当前代码文件中提取出的所有标识符放在一个侧边子窗口中，并且能够按语法规则将标识符进行归类。
`tagbar`安装完成后在`.vimrc`文件中添加如下配置:

```
    " set tagbar
    map <F8> :TagbarToggle<CR>  " 通过快捷键F8实现tagbar打开/关闭
    let tagbar_left=1           " 设置tagbar子窗口的位置出现在主编辑的左边
    let tagbar_width=32         " 设置窗口宽度
    let g:tagbar_compact=1      " 子窗口不显示冗余帮助信息
    let g:tagbar_sort=0         " 设置默认排序方式为在文中出现的位置
    let g:tagbar_type_cpp = {   " 设置对哪些代码标识符生成标签
            \'kinds' : [
                \ 'c:classes:0:0',
                \ 'd:macros:0:0',
                \ 'e:enumerators:0:0',
                \ 'f:functions:0:0',
                \ 'g:enumeration:0:0',
                \ 'l:local:0:0',
                \ 'm:members:0:0',
                \ 'n:namespaces:0:0',
                \ 'p:functions_prototypes:0:0',
                \ 's:structs:0:0',
                \ 't:typedefs:0:0',
                \ 'u:unions:0:0',
                \ 'v:global:0:0',
                \ 'x:wxternal:0:0'
            \ ],
            \ 'sro'        : '::',
            \ 'kind2scope' : {
                \ 'g' : 'enum',
                \ 'n' : 'namespaces',
                \ 'c' : 'class',
                \ 's' : 'struct',
                \ 'u' : 'union'
            \ },
            \ 'scope2kind' : {
                \ 'enum'      : 'g',
                \ 'namespace' : 'n',
                \ 'class'     : 'c',
                \ 'struct'    : 's',
                \ 'union'     : 'u'
                \ }
            \ }
```

进入之前生成`tag`文件的目录下，打开某一源文件后，通过快捷键`F8` 打开`tagbar`子窗口，可以通过快捷键`ctrl+w+w`快速切换子窗口，类似`alt+tab`的功能，或者使用前面定义的`ctrl+h/i/j/k`在`normal`模式下进行窗口切换，进入到`tagbar`窗口中。

在标识符列表中选中对应标识符后即可跳转至源码中对应位置;在源码中停顿几秒，`tagbar`将高亮对应标识符。`tagbar`有两种排序方式，一是按标签字母先后顺序；一是按标签在源码中出现的先后顺序。此段摘抄 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide) 4.7节。

注释掉`let g:tagbar_sort=0`，将会使用字母顺序排序。

开发时代码不停在变更，如果每次都需要手动执行命令生成新的标签文件，太麻烦。通过引入插件 [indexer](https://github.com/vim-scripts/indexer.tar.gz) 实现自动生成标签。并引入。
`indexer` 依赖 [DfrankUtil](https://github.com/vim-scripts/DfrankUtil)、[vimprj](https://github.com/vim-scripts/vimpr) 两个插件，请一并安装。 此段摘抄[所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_id) 4.7节。

```
    Plugin 'vim-scripts/indexer.tar.gz'
    Plugin 'vim-scripts/DfrankUtil'
    Plugin 'vim-scripts/vimprj'
```

并在`.vimrc`文件中添加:

```
    " 设置插件 indexer 调用 ctags 的参数
    " 默认 --c++-kinds=+p+l，重新设置为 --c++-kinds=+p+l+x+c+d+e+f+g+m+n+s+t+u+v
    " 默认 --fields=+iaS 不满足 YCM 要求，需改为 --fields=+iaSl
    let g:indexer_ctagsCommandLineOptions="--c++-kinds=+p+l+x+c+d+e+f+g+m+n+s+t+u+v --fields=+iaSl --extra=+q"
```

另外，`indexer` 还有个自己的配置文件，用于设定各个工程的根目录路径，配置文件位于`~/.indexer_files`，内容格式为:

```
    [project_name]
    /project/code
```

方括号内为自动生成的标签文件名字，一般更工程一样。路径为工程的代码目录，不要包含构建目录，文档目录等，以免产生非代码文件的标签信息。
当打开以上目录任何代码文件时，插件`indexer`便对整个目录创建标签文件，当代码有更新，文件保存时，`indexer`将自动调用`ctags`更新标签文件。
`indexer`生成的标签文件以方括号内的名字命名，位于目录: `~/.indexer_files_tags/` 下，并自动引入`Vim`中。此处3段摘抄 [所需及所获:像使用IDE一样使用vim](https://github./yangyangwithgnu/use_vim_as_ide) 4.7节。

###### 声明/定义跳转

主要两类导航: 基于标签的跳转和基于语义的跳转。

既然`Vim`可以引入标签，说明`Vim`能识别标签。虽然标签文件中并无行号，但已经有标签所在文件以及标签所在行的完整内容，`Vim`只需切换至对应文件，再在文件内做内容查找即可找到对应行。
换言之，只要有对应的标签文件，`Vim`就能根据标签跳转至标签定义处。

在前述已生成标签文件的项目目录中打开任意文件，将光标定位到某一函数名或变量上，键入快捷键`g]`，`Vim`将罗列处该函数或变量名所有的标签候选列表，按需要键入编号后按`Enter`键即可跳转进入。

基于语义的跳转是在后面介绍补全时的一款插件`YCM`的功能。此处直接给出快捷键设置，在`.vimrc`文件中添加配置:

```
    "nnoremap <M-=> :YcmCompleter GoToDefinition<CR>                " jump to define
    "nnoremap <M-=> :YcmCompleter GoToDeclaration<CR>               " jump to declaration
    nnoremap <M-=> :YcmCompleter GoToDefinitionElseDeclaration<CR>  " jump to define or declaration
```

`YCM`提供这三个快捷键定义，意思很明了，可以根据自己情况进行配置，当然需要配置不同的快捷键。

但是个人感觉，这两个插件的跳转功能都不够好用。

写到这儿，有点不想写了。很多东西在我参考的文章中都很详细，很多地方都是摘抄过来的，所以后续我回直接贴出我所介绍的插件在原文章中的章节位置，然后主要介绍我所遇到的一些小问题和解决办法，以及我的快捷键配置，提供十分有限的参考价值，谢谢。

###### 快速移动

[vim-easymotion](https://github.com/easymotion/vim-easymotion) 更多请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  8.3节。

`.vimrc`文件配置如下:

```
    Plugin 'easymotion/vim-easymotion'
```

```
    " set easymotion
    let g:EasyMotion_leader_key='f'
```

设置启用`easymotion`的前缀键`<Leader>`为`f`，在`normal`模式下，双击`f`键后输入需要跳转到的位置的字母，然后根据快速跳转的字符，快速定位到目标位置。

###### 内容查找

更多请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  4.8节。

[ctrlsf.vim](https://github.com/dyng/ctrlsf.vim) 后端调用`ack`一定记得安装。`.vimrc`文件中添加配置如下:

```
    Plugin 'dyng/ctrlsf.vim'
```

```
" set ctrlsf
    let g:ctrlsf_ackprg='ack'              " set default
    nnoremap <M-f> :CtrlSF<CR>             " 键入快捷键alt+f，搜索光标当前字符串的匹配项
```

`ctrlsf`通过`p`键可以定位到匹配项的完整代码，类似跳转功能，键入`q`退出插件。

###### 内容替换

更多请参考 [所需及所获:像使用IDE一样使用vim](https:/github.com/yangyangwithgnu/use_vim_as_ide) 4.9节。

[vim-multiple-cursors](https://github.com/terryma/vim-multiple-cursors) 配合`ctrlsf`插件，实现多个不同位置的字符串替换。`.vimrc`文件中添加如下配置:

```
    Plugin 'terryma/vim-multiple-cursors'
```

```
    " set multiple-sursors
    let g:multi_cursor_use_default_mapping=0
    let g:multi_cursor_next_key='<S-n>'
    let g:multi_cursor_prev_key='<S-p>'
    let g:multi_cursor_skip_key='<S-s>'
    let g:multi_cursor_quit_key='<ESC>'
```

`S`表示`Shift`键。在通过`ctrlsf`插件列举处匹配项后，则可以通过快捷键`S-n`选中当前匹配项并跳到下一个匹配项，键入`S-s`则可以跳过当前项而选中下一个，可以通过`S-p`回退到前一个匹配项。键入`Esc`推出插件。

`Vim`自带很强大的替换功能，这里就不介绍了。建议了解一哈，一般会看这篇文档的，都有可能在面试的时候被提问。-_-

###### 文件搜索

```
    Plugin 'Yggdroot/LeaderF'
```

[ctrlP](https://github.com/kien/ctrlp.vim) 和[LeaderF](https://github.com/Yggdroot/LeaderF) 插件都能实现文件搜索功能，`ctrlP`在网上搜索就能找到配置方法，这里不介绍。这里使用的`LeaderF`在网上资料不多，看作者的介绍操作，暂时还不够熟悉，后续补充。

###### 快速注释

更多请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  5.1节。

[NERD Commenter](https://github.com/scrooloose/nerdcommenter) 能够快速注释/取消注释，并且提供不同的注释方式。在`.vimrc`文件中添加配置:

```
    Plugin 'scrooloose/nerdcommenter'
```

```
    " set nerdcommenter
    map <M-c> <Leader>cc                   " add comment
    map <M-u> <Leader>cu                   " cancel comment
    map <M-a> <Leader>ca                   " switch /*  */ & //
    let g:NERDSpaceDelims=1                " 强制在注释符旁边增加一个空格
```

在`Vim` `normal`模式下，键入`v`或者`V`进入可视化模式，通过`h/j/k/l`选中需要注释的部分，键入快捷键`alt+c`则注释当前选中代码。
反之，选中需要取消注释的部分，键入快捷键`alt+u`取消注释。`alt+a`可以切换注释方式:`/* */或者//`，但这个快捷键定义跟某些有冲突，将就可用，也可以自定义其他。

###### 行尾空格

这个插件是我在知乎的一个回答中了解到的，有实用性，提供参考。贴上这个知乎的帖子:https://www.zhihu.com/question/19989337
写程序时，虽然行尾的空格都看不见，遇上了的话，有点强迫症的肯定受不了。为了能够显示行尾的空格，需要[ShowTrailingWhiteSpace](https://github.com/vim-scripts/ShowTrailingWhitespace)这个插件。在`.vimrc`中添加如下内容:

```
    Plugin 'vim-scripts/ShowTrailingWhiteSpace'
```

```
    " set ShowTrailingWhitespace
    let g:ShoeTrailingWhitespace=1                          " default ON; 1 on, 0 off
    highlight ShowTrailingWhitespace ctermbg=Red guibg=Red  " 可以理解为高亮行尾空格
```

帖子中的回答者，还提供了一个快捷键，自动清理文件中所有行尾的空格:

```
    "strip all trailing whitespace in the current file
    nnoremap <leader>w :%s/\s\+$//<cr>:let @/=''<CR>
```

我自己改变了一哈，每次保存文件(所有格式的文件)的时候，自动清除当前文件中所有行尾的空格:

```
    "autocmd BufWritePre *.c,*.cpp,*.h,*.sh, :%s/\s\+$//e
    autocmd BufWritePre * :%s/\s\+$//e
```

注释掉的配置，表示只在这几个类型的文件执行保存文件命令时自动清除行尾空格，文件类型自己根据需要添加。

###### 补全

主要介绍三种补全:模板补全，基于标签的补全，基于语义的补全。

模板补全: 更多请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  5.2节。

[ultisnips](https://github.com/SirVer/ultisnhips) 提供相同代码段的快速输入，比如`if-else`、`switch-case`语句。在`.vimrc`文件中添加如下配置:

```
    Plugin 'SirVer/ultisnips'
    "Plugin 'honza/vim-snippets'
```

`ultisnips`补全时用于选中的键为`tab`，与后面将要提到的插件`YCM`的快捷键有冲突，所以提供改建的配置方式。个人选择改`YCM`的快捷键。

```
    " set ultisnips
    "let g:UltiSnipsExpandTrigger="<Leader><tab>"
    "let g:UltiSnipsJumpForwardTrigger="<M-n>"
    "let g:UltiSnipsJumpBackwardTrigger="<M-m>"
```

另外需要说明的一点是，下载`ultisnips`插件后，默认是没有提供模板补全用的文件的。需要自己下载。
一种方式时安装插件[vim-snippets]( https://github.com/honza/vim-snippets)，复制`~/.vim/bundle/vim-snippets/UltiSnips`整个文件夹到`~/.vim/bundle/ultisnips`目录下面即可。
另一种是自己到`github`上去下载， https://github.com/honza/vim-snippets ，其实内容都是一样的。我认为这个插件最好的点是支持自己定义，很强大。

基于标签的补全: 详情请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  5.3节部分。

基于语义的补全: 更多请参考 [所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  5.3节部分。

[YouCompleteMe](https://github.com/Valloric/YouCompleteMe) 插件在安装过程中，参看文章的作者以及很多其他介绍`YCM`插件使用的作者都有说到，`YCM`原作者建议下载`LLVM`官网提供的预编译二进制文件，以避免各种问题。
但我按照教程介绍，最后编译生成始终只有一个`ycm_core.so`文件，也没有查找到原因，待优化。
然后后面我是按照`YCM`原作者Git项目中`README.md`文件中`Ubuntu Linux x64`部分实现安装的，不需要手动下载库，按作者的顺序执行即可，需要比较长的时间。需要确认已安装`Cmake`、`python-dev`、`python3-dev`，没有的话请自行安装。
更多请自行阅读原作者[ README.md ](https://github.com/Valloric/YouCompleteMe)。

`YCM`插件需要在`.vimrc`文件中添加配置:

```
    Plugin 'Valloric/YouCompleteMe'
```

```
    " set YouComplete
    set runtimepath+=~/.vim/bundle/YouCompleteMe
    let g:ycm_collect_identifiers_from_tags_files=1                             " 开启YCM标签补全
    let g:ycm_complete_in_comments=1                                            " 补全功能在注释中同样有效
    let g:ycm_seed_identfiers_with_syntax=1                                     " 语法关键字补全
    let g:ycm_confirm_extra_conf=0                                              " 允许VIM加载.ycm_extra_conf.py 文件，不再提示
    let g:ycm_key_list_select_completion=['<M-n>', "<Down>"]                    " 补全列表中 向下选中
    let g:ycm_key_list_previous_completion=['<M-m>', "<Up>"]                    " 补全列表中 向上选中
    let g:ycm_min_num_of_chars_for_completion=2                                 " 从第2个键入字符就开始罗列匹配项
    let g:ycm_cache_comnifunc=0                                                 " 禁止缓存匹配项，每次都重新生成匹配项
    " 加载.ycm_extra_conf.py 文件的路径，可以不用每个工程目录下都去添加这个文件
    let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_part/ycmd/cpp/ycm/.ycm_extra_conf.py'
    "nnoremap <M-=> :YcmCompleter GoToDefinition<CR>                            " 跳转到声明/定义的位置，在前面跳转部分有介绍
    "nnoremap <c-j> :YcmCompleter GoToDefinitionElseDeclaration<CR>
    nnoremap <M-=> :YcmCompleter GoToDefinitionElseDeclaration<CR>
```

另外还需要配置`.ycm_extra_conf.py`文件以及引入标签补全，这里就不做详述，请自行参考介绍。

###### 工程文件管理

[NERDtree](`https://github.com/scrooloose/nerdtree)可以查看文件列表，要打开哪个文件，光标选中后回车即可在新`buffer`中打开。在`.vimrc`文件中添加如下配置:

```
    Plugin 'scrooloose/nerdtree'
```

```
    " set NERDTree
    nmap <F9> :NERDTreeToggle<CR>
    let NERDTreeShowBookmarks=1             " set NERDTree book marks default on
    let NERDTreeIgnore=['\.py[cd]$', '\~$', '\.swo$', '\.swp$', '^\.git$', '^\hg$', '^\.svn$', '\.bzr$']
    let NERDTreeWinSize=32                  " set width
    let NERDTreeWinPos="right"              " set position
    let NERDTreeShoeHidden=1                " show the hidden files
    let NERDTreeMinimalUI=1                 " the window tags not show help info
    let NERDTreeAutoDeleteBuffer=1          " Automatically deletes files when deletes files buffer
```

更多请参考[ 所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  6.1节部分。

###### 多文件编辑

`Vim`中每打开一个文件`Vim`就对应创建一个`buffer`，多个文件就有多个`buffer`，但默认你只看得到最后`buffer`对应的Window，通过插件[MiniBufExplorer](https://github.com/fholgado/minibufexpl.vim)可以切换到不同的`buffer`及达到编辑多个文件的功能。在`.vimrc`文件中添加配置:

```
    Plugin 'fholgado/minibufexpl.vim'
```

```
    " set minibuffexploer
    map <M-q> :bn<CR>                           " 切换多个buffer时的快捷键
    map <M-w> :bp<CR>
    " 这个设置，网上我查到两种说法，一个是在gvim下时防止出现多窗口，一种是自动打开。实际情况看，无论是0/1，在只有一个文件时都会显示nimibuffer
    let g:let g:miniBufExplorerMoreThanOne=1
```

`buffer`快捷切换时，只需要在当前编辑文件中`normal`下按快捷键切换(需要存在多个buffer)，但是删除`buffer`时，需要将光标定位到`buffer`标签上。键入`d`删除光标所在`buffer`。

更多请参考[ 所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)  6.2节部分。

###### 自动对齐

```
    Plugin 'godlygeek/tabular'               " 自动对齐
```

后续补充。可自行搜索。

###### Markdown

`Markdown`就不多解释了，在`Vim`编辑过程中最大的需求就是实时预览，在这里介绍插件[markdown-preview](https://github.com/iamcco/markdown-preview.vim)，能够实现在浏览器中实时预览，默认是谷歌浏览器`Chorme`，自行下载。
需要说明的是，这款插件在Github上比较多，比如[markdown-preview](https://github.com/atom/markdown-preview)，不需要浏览器支持即可实时预览，供君选择。
[vim-markdown](https://github.com/plasticboy/vim-markdown) 是一个语法高亮插件，根据实际情况使用。

在`.vimrc`文件中添加配置:

```
    Plugin 'plasticboy/vim-markdown'
    Plugin 'iamcco/markdown-preview.vim'
```

```
    " set markdown
    let g:mkdp_auto_start=0                 " set 1, the vim will auto open preview window once enter markdown buffer
    let g:mkdp_auto_open=0                  " set 1, the vim will auto open preciew window when you edit the markdown file
    let g:mkdp_auto_close=1                 " set 1, the vim will auto close current preview window when change from markdown buffer to another buf
    let g:mkdp_refresh_slow=0               " set 1, the vim will just refresh markdown when save the buffer or leave frim insert mode,default 0 is
    let g:mkdp_command_for_global=0         " set 1, the markdown preview command can be use for all files, by default it just can be use in markdo
    let g:mkdp_path_to_chrome="google-chrome"
    map <F6> <Plug>MarkdownPreview
    map <F7> <Plug>StopMarkdownPreview
```

基本上都是默认配置，除了打开/关闭的快捷键映射。更多的话，可以自己到Github看作者介绍。

### 其他配置

###### 文件模板

新建源文件时，一般都会添加一些文件信息在文件开始的位置处。常见比如文件名、作者、创建时间、文件简介等等，如果每次都手动添加，就会很麻烦。所以可以通过添加脚本函数的方式，实现文件新建时自动添加。
在这里贴上一个`.vimrc`文件的配置，可以参考他的实现方式: https://github.com/taizilongxu/dotfiles/blob/master/vimrc 。文件中其他也可以参考。
注意一点，如果需要判断是否为`*.h`文件，头文件，请使用`if expand("%:e")=='h'`进行判断。

###### 自动添加时间

需要的话也是通过脚本函数实现。`.vimrc`文件中添加配置:

```
    autocmd BufWritePre,FileWritePre *.c,*.h,*.sh,*.py ks|call LastChanged()|'s
    func! LastChanged()                          " auto add last changed date
        if line ("$") > 15
            let l = 15
        else
            let l = line("$")
        endif
        exe "1,".l."g/Last Changed : /s/Last Changed : .*/Last Changed : ".strftime("%c")
    endfunc
```

关于这一段的解释和详细介绍请参考: http://www.cnblogs.com/soli/archive/2009/10/21/885044.html 。

### 结束语

第一次的环境配置就先这样了，后续有其他好用插件，会再推荐。文中还有没写完整的，会尽快补充。
没有特别认真的写，比较水，希望不要介意，能够提供一丁点的帮助，我觉得也是好的。谢谢。

### 参考

1、[所需及所获:像使用IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide)
2、[.vimrc](https://github.com/wklken/k-vim/blob/master/vimrc)
3、[.vimrc](https://github.com/taizilongxu/dotfiles/blob/master/vimrc)
4、[.vimrc](https://github.com/humiaozuzu/dot-vimrc/blob/master/vimrc)
5、[vim配置技巧——在vim中使用autocmd命令在保存文件时自动插入最后修改日期和时间](http://www.cnblogs.com/soli/archive/2009/10/21/885044.html)
6、[Vim与Markdown共舞](https://segmentfault.com/a/1190000008321057)
7、[NERD_commenter安装和使用](http://blog.csdn.net/zcube/article/details/42298419)
