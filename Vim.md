
# 概念

[Missing Semesters Vim](https://missing-semester-cn.github.io/2020/editors/)

- **正常模式**：在文件中四处移动光标进行修改
- **插入模式**：插入文本
- **替换模式**：替换文本
- **可视化模式**（一般，行，块）：选中文本块
- **命令模式**：用于执行命令

>Vim 会维护一系列打开的文件，称为“缓存”。一个 Vim 会话包含一系列标签页，每个标签页包含 一系列窗口（分隔面板）。每个窗口显示一个缓存。跟网页浏览器等其他你熟悉的程序不一样的是， 缓存和窗口不是一一对应的关系；窗口只是视角。一个缓存可以在_多个_窗口打开，甚至在同一 个标签页内的多个窗口打开。这个功能其实很好用，比如在查看同一个文件的不同部分的时候。

# 命令

operator *number* motion

## 基础
### 移动 motion

多数时候你会在正常模式下，使用移动命令在缓存中导航。在 Vim 里面移动也被称为 “**名词**”， 因为它们指向文字块。

- 基本移动: `hjkl` （左， 下， 上， 右）
- 词： `w` （word，下一个词）， `b` （词初）， `e` （end，词尾）
- 行： `0` （行初）， `^` （第一个非空格字符）， `$` （行尾）
- 屏幕： `H` （屏幕首行）， `M` （屏幕中间）， `L` （屏幕底部）
- 翻页： `Ctrl-u` （上翻）， `Ctrl-d` （下翻）
- 文件： `gg` （文件头）， `G` （文件尾）
- 行数： `:{行数}<CR>` 或者 `{行数}G` ({行数}为行数)
- 杂项： `%` （找到配对，比如括号或者 /* */ 之类的注释对）
- 查找： `f{字符}`， `t{字符}`， `F{字符}`， `T{字符}`
    - 查找/到 向前/向后 在本行的{字符}
    - `,` / `;` 用于导航匹配
- 搜索: `/{正则表达式}`, `n` / `N` 用于导航匹配

### 选择

可视化模式:

- 可视化：`v`
- 可视化行： `V`
- 可视化块：`Ctrl+v`

可以用移动命令来选中。

### 编辑

所有你需要用鼠标做的事， 你现在都可以用键盘：采用编辑命令和移动命令的组合来完成。 这就是 Vim 的界面开始看起来像一个程序语言的时候。Vim 的编辑命令也被称为 “**动词**”， 因为动词可以施动于名词。

- `i` 进入插入模式
    - 但是对于操纵/编辑文本，不单想用退格键完成
- `O` / `o` 在之上/之下插入行
- `d{移动命令}` 删除 {移动命令}
    - 例如，`dw` 删除词, `d$` 删除到行尾, `d0` 删除到行头。
- `c{移动命令}` 改变 {移动命令}
    - 例如，`cw` 改变词
    - 比如 `d{移动命令}` 再 `i`
- `x` 删除字符（等同于 `dl`）
- `s` 替换字符（等同于 `xi`）
- 可视化模式 + 操作
    - 选中文字, `d` 删除 或者 `c` 改变
- `u` 撤销, `<C-r>` 重做
- `y` 复制 / “yank” （其他一些命令比如 `d` 也会复制）
- `p` 粘贴
- 更多值得学习的: 比如 `~` 改变字符的大小写

### 计数

你可以用一个计数来结合“名词”和“动词”，这会执行指定操作若干次。

- `3w` 向前移动三个词
- `5j` 向下移动5行
- `7dw` 删除7个词

### 修饰语

你可以用修饰语改变“名词”的意义。修饰语有 `i`，表示“内部”或者“在内“，和 `a`， 表示”周围“。

- `ci(` 改变当前括号内的内容
- `ci[` 改变当前方括号内的内容
- `da'` 删除一个单引号字符串， 包括周围的单引号

## 进阶

### 搜索和替换

`:s` （替换）命令（[文档](http://vim.wikia.com/wiki/Search_and_replace)）。

- `%s/foo/bar/g`
    - 在整个文件中将 foo 全局替换成 bar
- `%s/\[.*\](\(.*\))/\1/g`
    - 将有命名的 Markdown 链接替换成简单 URLs

### 多窗口

- 用 `:sp` / `:vsp` 来分割窗口
- 同一个缓存可以在多个窗口中显示。
- 使用`Ctrl+w w`可以切换窗口

### 宏

- `q{字符}` 来开始在寄存器`{字符}`中录制宏
- `q`停止录制
- `@{字符}` 重放宏
- 宏的执行遇错误会停止
- `{计数}@{字符}`执行一个宏{计数}次
- 宏可以递归
    - 首先用`q{字符}q`清除宏
    - 录制该宏，用 `@{字符}` 来递归调用该宏 （在录制完成之前不会有任何操作）
- 例子：将 xml 转成 json ([file](https://missing-semester-cn.github.io/2020/files/example-data.xml))
    - 一个有 “name” / “email” 键对象的数组
    - 用一个 Python 程序？
    - 用 sed / 正则表达式
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - …
    - Vim 命令 / 宏
        - `Gdd`, `ggdd` 删除第一行和最后一行
        - 格式化最后一个元素的宏 （寄存器 `e`）
            - 跳转到有 `<name>` 的行
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - 格式化一个的宏
            - 跳转到有 `<person>` 的行
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - 格式化一个标签然后转到另外一个的宏
            - 跳转到有 `<person>` 的行
            - `qq@pjq`
        - 执行宏到文件尾
            - `999@q`
        - 手动移除最后的 `,` 然后加上 `[` 和 `]` 分隔符


## 更多窍门

- 标记 - 在vim里你可以使用 `m<X>` 为字母 `X` 做标记，之后你可以通过 `'<X>` 回到标记位置。这可以让你快速定位到文件内或文件间的特定位置。
- 导航 - `Ctrl+O` 和 `Ctrl+I` 命令可以使你在最近访问位置前后移动。
- 撤销树 - vim 有不错的更改跟踪机制，不同于其他的编辑器，vim存储变更树，因此即使你撤销后做了一些修改，你仍然可以通过撤销树的导航回到初始状态。一些插件比如 [gundo.vim](https://github.com/sjl/gundo.vim) 和 [undotree](https://github.com/mbbill/undotree) 通过图形化来展示撤销树。
- 时间撤销 - `:earlier` 和 `:later` 命令使得你可以用时间而非某一时刻的更改来定位文件。
- [持续撤销](https://vim.fandom.com/wiki/Using_undo_branches#Persistent_undo) - 是一个默认未被开启的vim的内置功能，它在vim启动之间保存撤销历史，需要配置在 `.vimrc` 目录下的`undofile` 和 `undodir`，vim会保存每个文件的修改历史。
- 热键（Leader Key） - 热键是一个用于用户自定义配置命令的特殊按键。这种模式通常是按下后释放这个按键（通常是空格键）并与其他的按键组合去实现一个特殊的命令。插件也会用这些按键增加它们的功能，例如，插件UndoTree使用 `<Leader> U` 去打开撤销树。
- 高级文本对象 - 文本对象比如搜索也可以用vim命令构成。例如，`d/<pattern>` 会删除下一处匹配 pattern 的字符串，`cgn` 可以用于更改上次搜索的关键字。

# 个性化

`~/.vimrc`如下：

```
setlocal noswapfile " 不要生成swap文件                                       
set bufhidden=hide " 当buffer被丢弃的时候隐藏它
colorscheme evening " 设定配色方案
set number " 显示行号
set cursorline " 突出显示当前行
set ruler " 打开状态栏标尺
set shiftwidth=2 " 设定 << 和 >> 命令移动时的宽度为 2
set softtabstop=2 " 使得按退格键时可以一次删掉 2 个空格
set tabstop=2 " 设定 tab 长度为 2
set nobackup " 覆盖文件时不备份
set autochdir " 自动切换当前目录为当前文件所在的目录
set backupcopy=yes " 设置备份时的行为为覆盖
set hlsearch " 搜索时高亮显示被找到的文本
set noerrorbells " 关闭错误信息响铃
set novisualbell " 关闭使用可视响铃代替呼叫
set t_vb= " 置空错误铃声的终端代码
set matchtime=2 " 短暂跳转到匹配括号的时间
set magic " 设置魔术
set smartindent " 开启新行时使用智能自动缩进
set backspace=indent,eol,start " 不设定在插入状态无法用退格键和 Delete 键删除回车符
set cmdheight=1 " 设定命令行的行数为 1
set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ Ln\ %l,\ Col\ %c/%L%) " 设置在状态行显示的信息
"set foldenable " 开始折叠
"set foldmethod=syntax " 设置语法折叠
"set foldcolumn=0 " 设置折叠区域的宽度
"setlocal foldlevel=1 " 设置折叠层数为 1
"nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR> " 用空格键来开关折叠
"let g:molokai_original = 1
colorscheme molokai
```

`colorscheme molokai`表示使用安装在`~/.vim/colors`里的molokai主题。


















