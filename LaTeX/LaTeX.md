# 用法杂记

## 查看宏包文档

```bash
texdoc xxx
```

而`texdoc lshort-zh`可以查看中文教程文档。

## 不标号

在标题标签后加上`*`可以隐藏标题序号（前缀），公式也能用它隐藏标号。

# 模板

```latex
\documentclass[11pt]{paper}

\usepackage{ctex}
\usepackage[colorlinks,linkcolor=black,anchorcolor=black,citecolor=black]{hyperref}
\usepackage{setspace}

\title{xxx}
\author{aaa}
\date{2023/5/11}
  
\begin{document}
%下方居中显示页码
\pagestyle{plain}
  
\begin{titlepage}
    \begin{center}%开始插题目
        \vspace{4mm}  %插入间距
        \begin{spacing}{2}%行距2
            Lab Report of "Fourier Analysis and Synthesis of Waveforms" experiment
        \end{spacing}
        \vspace{4mm}  %插入间距
        \large{       %调大以下内容的字号
            \begin{tabular}{ll} %用表格的方式插入学生信息
                \textbf{Name:}           & Fisher' C        \\
                \textbf{Student number:} & 666666           \\
                \textbf{College email:}  & kekeke@gmail.com \\
            \end{tabular}
        }
    \end{center}
\end{titlepage}
  
\newpage
\pagenumbering{roman}
  
\vspace{10mm}
\setcounter{tocdepth}{2}%只显示到2级标题
\tableofcontents
  
\newpage
\pagenumbering{arabic}
  
\section{Objective}
\heiti
你好

\end{document}
```

# 基本语法

## HelloWorld

```latex
\documentclass{article}

\usepackage{ctex}

\begin{document}
 你好
\end{document}
```

## 基本结构

```latex
%导言区
\documentclass{article}

\author{abc}
\date{2023/5/11}

%正文区（文稿区）
\begin{document}
\maketitle
hello
\end{document}
```

`$ xxxx $`内的是数学模式，外部是文本模式。

`$ xxxx $`为行内公式，`$$ xxxx $$`为行间公式（会另起一行）。

## 定义新命令

在导言区使用`\newcommand\xxx{abc}`即可定义新命令`\xxx`。类似于函数，比如可以封装字体。

## 带编号的行间公式

```latex
\begin{equation}
xxxxxxxxxxx
\end{equation}
```

## 变量

```latex
\setlength{\baselineskip}{20pt}
```

# 字体

```latex
%在紧跟的括号里设置字体族
\textrm{Roman Family}
\textsf{Sans Serif Family}
\texttt{Typewriter Family}

%在接下来的作用域内设置字体族
{\rmfamily Roman Family}
{\sffamily Sans Serif Family}
{\ttfamily Typewriter Family}
```

```latex
%中文的粗体就是黑体，斜体就是楷书。
\emph{强调}
\textbf{粗体}
\textit{斜体}
\underline{下划线}
```

![](assets/uTools_1699850246663.png)

![](assets/uTools_1699850328974.png)

# 段落

## 设置固定行间距

```latex
\setlength{\baselineskip}{20pt}
```

# 配置

## vscode配置
设置的json添加：
```json
"latex-workshop.latex.autoBuild.run": "never",
"latex-workshop.showContextMenu": true,
"latex-workshop.intellisense.package.enabled": true,
"latex-workshop.message.error.show": true,
"latex-workshop.message.warning.show": true,
"latex-workshop.latex.tools": [
  {
	  "name": "xelatex",
	  "command": "xelatex",
	  "args": [
		  "-synctex=1",
		  "-interaction=nonstopmode",
		  "-file-line-error",
		  "%DOCFILE%"
	  ]
  },
  {
	  "name": "pdflatex",
	  "command": "pdflatex",
	  "args": [
		  "-synctex=1",
		  "-interaction=nonstopmode",
		  "-file-line-error",
		  "%DOCFILE%"
	  ]
  },
  {
	  "name": "latexmk",
	  "command": "latexmk",
	  "args": [
		  "-synctex=1",
		  "-interaction=nonstopmode",
		  "-file-line-error",
		  "-pdf",
		  "-outdir=%OUTDIR%",
		  "%DOCFILE%"
	  ]
  },
  {
	  "name": "bibtex",
	  "command": "bibtex",
	  "args": [
		  "%DOCFILE%"
	  ]
  },
  {
	"name": "biber",
	"command": "biber",
	"args": [
	  "%DOCFILE%"
	]
  }
],
"latex-workshop.latex.recipes": [
  {
	  "name": "XeLaTeX",
	  "tools": [
		  "xelatex"
	  ]
  },
  {
	  "name": "PDFLaTeX",
	  "tools": [
		  "pdflatex"
	  ]
  },
  {
	  "name": "BibTeX",
	  "tools": [
		  "bibtex"
	  ]
  },
  {
	  "name": "LaTeXmk",
	  "tools": [
		  "latexmk"
	  ]
  },
  {
	"name": "xe->biber->xe->xe",
	"tools": [
	  "xelatex",
	  "biber",
	  "xelatex",
	  "xelatex"
	]
  },
  {
	  "name": "xelatex -> bibtex -> xelatex*2",
	  "tools": [
		  "xelatex",
		  "bibtex",
		  "xelatex",
		  "xelatex"
	  ]
  },
  {
	  "name": "pdflatex -> bibtex -> pdflatex*2",
	  "tools": [
		  "pdflatex",
		  "bibtex",
		  "pdflatex",
		  "pdflatex"
	  ]
  },
],
"latex-workshop.latex.clean.fileTypes": [
  "*.aux",
  "*.bbl",
  "*.blg",
  "*.idx",
  "*.ind",
  "*.lof",
  "*.lot",
  "*.out",
  "*.toc",
  "*.acn",
  "*.acr",
  "*.alg",
  "*.glg",
  "*.glo",
  "*.gls",
  "*.ist",
  "*.fls",
  "*.log",
  "*.fdb_latexmk"
],
"latex-workshop.latex.autoClean.run": "onFailed",
"latex-workshop.latex.recipe.default": "lastUsed",
"latex-workshop.view.pdf.internal.synctex.keybinding": "double-click",
```




























