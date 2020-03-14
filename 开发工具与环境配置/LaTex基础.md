## 概述

**TEX**：TEX 是高德纳 (Donald E. Knuth) 开发的、以排版文字和数学公式为目的的软件。1977 年，正在编写著作《计算机程序设计艺术》的高德纳，意图扭转排版质量每况愈下的状况，以免影响他的出书，于是开始开发 TEX，以发掘当时开始用于出版工业的数字印刷设备的潜力。TEX 排版引擎发布于 1982 年，在 1989 年又加以改进以更好地支持 8-bit 字符和多语言排版。TEX 以其卓越的稳定性、跨平台、几乎没有 bug 而著称。TEX 的版本号不断趋近于 *π*，当前为 3.141592653。 TEX 读作 “Tech” ，其中 “ch” 的发音类似于 “h” ，与汉字“泰赫”的发音相近。TEX 的拼写，来自希腊词语 τεχνική (technique，技术) 的开头几个字母。在 ASCII 字符环境，TEX 写作 TeX

**LaTEX**：LATEX 是一种格式（format）。为免误会，初次接触这一概念的读者可以粗略地将 LATEX 理解成是对 TEX 的一层封装。LATEX 使用 TEX 程序作为自己的排版引擎。LATEX 最初的设计目标 是分离内容与格式，以便作者能够无需关注版式设计，只需专注与内容创作就能得到高质量排版的作品。

### 使用示例

使用texstudio编辑器

```latex
\documentclass{article} 
\title{A Test for TeXstudio} 
\author{lsl} 
\begin{document} 
	\maketitle
	\tableofcontents 
	\section{Hello China} China is in East Asia. 
	\subsection{Hello Beijing} Beijing is the capital of China. 
	\subsubsection{Hello Dongcheng District} 
	\paragraph{Hello Tian'anmen Square}is in the center of Beijing 
	\subparagraph{Hello Chairman Mao} is in the center of Tian'anmen Square 
\end{document} 
```

这是一段代码，需要编译执行，执行之后(texstudio默认为F5)可以预览效果，编译之后就会生成pdf,toc等文件

###  LATEX 命令和环境

LATEX 中命令以反斜线 \ 开头，为以下两种形式之一： 

* 反斜线和后面的一串字母，如\LaTeX。它们以任意非字母符号（空格、数字、标点等）为界限。 
* 反斜线和后面的单个非字母符号，如\\$。  

一些 LATEX 命令可以接收一些参数，参数的内容会影响命令的效果。LATEX 的参数分为可选 参数和必选参数。可选参数以方括号 [ 和 ] 包裹；必选参数一般以花括号 { 和 } 包裹。还有些 命令可以带一个星号 *，带星号和不带星号的命令效果有一定差异。初次接触这些概念时，可以 粗略地把星号看作一种特殊的可选参数。 LATEX 中还包括环境，用以令一些效果在局部生效，或是生成特殊的文档元素。

### LATEX源代码结构 

LATEX 源代码以一个 \documentclass 命令作为开头，它指定了文档使用的文档类： \documentclass{...} \documentclass 命令有一个可选参数。document 环境当中的内容是文档正文。 

```
\begin{document} 
\section{...} 
正文内容…… 
\end{document} 
```

在 \documentclass 和 \begin{document} 之间的位置称为导言区。在导言区中一般会使用 \usepackage 调用宏包，以及会进行对文档的全局设置。

#### 文档类\documentclass

文档类规定了 LATEX 源代码所要生成的文档的性质——普通文章、书籍、演示文稿、个人简 历等等。LATEX 源代码的开头须用 \documentclass 指定文档类： 

```latex
\documentclass[⟨options⟩]{⟨class-name⟩} 
```

其中 *⟨**class-name**⟩* 为文档类的名称，如 LATEX 提供的 article, book, report

* article：文章格式的文档类，广泛用于科技论文、报告、说明文档等。 
* report：长篇报告格式的文档类，具有章节结构，用于综述、长篇论文、简单的书籍等。 
* book：书籍文档类，包含章节结构和前言、正文、后记等结构。 
* proc：基于 article 文档类的一个简单的学术文档模板。 
* slides：幻灯格式的文档类，使用无衬线字体。 
* minimal：一个极其精简的文档类，只设定了纸张大小和基本字号，用作代码测试的最小工作示例（Minimal Working Example）。

可选参数 *⟨**options**⟩* 为文档类指定选项，以全局地规定一些排版的参数，如字号、纸张大小、 单双面等等。比如调用 article 文档类排版文章，指定纸张为 A4 大小，基本字号为 11pt，双面排版： 

```latex
\documentclass[11pt,twoside,a4paper]{article}
```

LATEX 的三个标准文档类可指定的选项包括： 

**10pt, 11pt, 12pt** ：指定文档的基本字号。默认为10pt。 

**a4paper, letterpaper, …** ：指定纸张大小，默认为美式信纸 letterpaper （8*.*5 *×* 11 英寸）。 可指定选项还包括 a5paper，b5paper，executivepaper 和 legalpaper。 

**twoside, oneside**：指定单面/双面排版。双面排版时，奇偶页的页眉页脚、页边距不同。article 和 report 默认为 oneside，book 默认为 twoside。 

**onecolumn, twocolumn**：指定单栏/双栏排版。默认为 onecolumn。 

**openright, openany** ：指定新的一章 \chapter 是在奇数页（右侧）开始，还是直接紧跟着上一页开始。report 默认为 openany，book 默认为 openright。对 article 无效。 

**landscape** ：指定横向排版。默认为纵向。 

**titlepage, notitlepage** 指定标题命令 \maketitle 是否生成单独的标题页。article 默认为notitlepage，report 和 book 默认为 titlepage。 

**fleqn** ：令行间公式左对齐。默认为居中对齐。 

**leqno** ：将公式编号放在左边。默认为右边。 

**draft, final** ：指定草稿/终稿模式。草稿模式下，断行不良的地方会在行尾添加一个黑色方块。默认为 final

#### 宏包\usepackage

在使用 LATEX 时，时常需要依赖一些扩展来增强或补充 LATEX 的功能，比如排版复杂的表格、插入图片、增加颜色甚至超链接等等（相当于引入类库）。这些扩展称为宏包。调用宏包的方法非常类似调用文档类的方法： 

```latex
\usepackage[⟨options⟩]{⟨package-name⟩} 
```

\usepackage 可以一次性调用多个宏包，在 *⟨**package-name**⟩* 中用逗号隔开。这种用法一般不要指定选项。

```latex
\usepackage{tabularx, makecell, multirow}
```

#### 文件组织方式\include

当编写长篇文档时，例如当编写书籍、毕业论文时，单个源文件会使修改、校对变得十分困 难。将源文件分割成若干个文件，例如将每章内容单独写在一个文件中，会大大简化修改和校对的工作。

LATEX 提供了命令 \include 用来在源代码里插入文件： 

```latex
\include{⟨fifilename⟩} 
```

*⟨**fifilename**⟩* 为文件名，如果和要编译的主文件不在一个目录中，则要加上相对或绝对路径，例如： 

\include{chapters/a.tex} % 相对路径 

\include{/home/Bob/file.tex} % Linux/macOS 绝对路径 

\include{D:/file.tex} % Windows 绝对路径，用正斜线 

值得注意的是 \include 在读入 *⟨**fifilename**⟩* 之前会另起一页。有的时候我们并不需要这样， 而是用 \input 命令，它纯粹是把文件里的内容插入： 

```latex
\input{⟨fifilename⟩}
```

#### 封面\maketitle

实际上是将\begin{}之上所定义的信息进行排版显示

```latex
\documentclass{article} 
\title{A Test for TeXstudio} 
\author{lsl} 
\begin{document} 
	\maketitle
\end{document} 
```

## 排版文字

### 字体大小

**全局模式****

```undefined
\documentclass[12pt]{article}
```

在文档的开头，有设置整个文章的字体大小，如：12pt。

**局部模式****

设置字体大小的命令从小到大为：

```undefined
\tiny
\scriptsize
\footnotesize
\small
\normalsize
\large
\Large
\LARGE
\huge
\Huge
```

注意：局部模式是相比于全局字体的基础上，来变大变小。例子：

```dart
\tiny Hello Latex.
```

### 字符

#### 空格

LATEX 源代码中，空格键和 Tab 键输入的空白字符视为“空格”。连续的若干个空白字符视为一个空格。一行开头的空格忽略不计。行末的回车视为一个空格；但连续两个回车，也就是空行，会将文字分段。多个空行被视为一个空行。也可以在行末使用 \par 命令分段。

| 两个quad空格 | a \qquad b | ![a \qquad b](http://upload.wikimedia.org/math/e/5/0/e505263bc9c94f673c580f3a36a7f08a.png) | 两个*m*的宽度  |
| ------------ | ---------- | ------------------------------------------------------------ | -------------- |
| quad空格     | a \quad b  | ![a \quad b](http://upload.wikimedia.org/math/d/a/8/da8c1d9effa4501fd80c054e59ad917d.png) | 一个*m*的宽度  |
| 大空格       | a\ b       | ![a\ b](http://upload.wikimedia.org/math/6/9/2/692d4bffca8e84ffb45cf9d5facf31d6.png) | 1/3*m*宽度     |
| 中等空格     | a\;b       | ![a\;b](http://upload.wikimedia.org/math/b/5/a/b5ade5d5393fd7727bf77fa44ec8b564.png) | 2/7*m*宽度     |
| 小空格       | a\,b       | ![a\,b](http://upload.wikimedia.org/math/7/b/e/7bea99aed60ba5e1fe8a134ab43fa85f.png) | 1/6*m*宽度     |
| 没有空格     | ab         | ![ab\,](http://upload.wikimedia.org/math/b/6/b/b6bd9dba2ebfca24731ae6dc3913e625.png) |                |
| 紧贴         | a\!b       | ![a\!b](http://upload.wikimedia.org/math/0/f/b/0fbcad5fadb912e8afa6d113a75c83e4.png) | 缩进1/6*m*宽度 |

#### 注释%

注释为%

#### 特殊符号

LATEX 预定义了其它一些文本模式的符号

<img src="https://s2.ax1x.com/2020/02/19/3EPVUK.png" alt="3EPVUK.png" style="zoom:50%;" />

### 分段

 \par：分段。 

### 强调

#### 下划线\underline

LATEX 定义了 \underline 命令用来为文字添加下划线

```latex
An \underline{underlined} text.
```

\underline 命令生成下划线的样式比较机械，不同的单词可能生成高低各异的下划线，并且无法换行。ulem 宏包解决了这一问题，它提供的 \uline 命令能够轻松生成自动换行的下划线：

```latex
An example of \uline{some
long and underlined words.}
```

#### 斜体\emph 

\emph 命令用来将文字变为斜体以示强调。如果在本身已经用 \emph 命令强调的文字内部嵌套使用 \emph 命令，内部则使用直立体文字

```latex
Some \emph{emphasized words,
including \emph{double-emphasized}
words}, are shown here
```

#### 断行\newline

在绝大多数时候，我们无需自己操心断行和断页。但偶尔会遇到需要手工调整的地方。断行可使用两个命令 

```latex
\\[⟨length⟩]
\newline
```

\\\ 可以带可选参数 *⟨**length**⟩*，用于在换行处向下增加垂直间距，而 \newline 不带可选参数；

\ \\ 也在表格、公式等地方用于分行，而 \newline只用于文本段落中。

#### 断页\newpage

断页的命令有两个： \newpage or \clearpage 

通常情况下两个命令都能起到另起一页的作用，但有一些区别：一是在双栏排版中 \newpage 只起到另起一栏的作用；二是涉及到浮动体的排版上行为不同。

#### 断词 

如果 LATEX 遇到了很长的英文单词，仅在单词之间的位置断行无法生成宽度匀称的行时，就要考虑从单词中间断开。对于绝大部分单词，LATEX 能够找到合适的断词位置，在断开的行尾加上连字符 -。  

## 文档元素

#### 章节标题

文档的标题层次结构为：

```latex
\chapter{⟨title⟩} \section{⟨title⟩} \subsection{⟨title⟩}
\subsubsection{⟨title⟩} \paragraph{⟨title⟩} \subparagraph{⟨title⟩}
```

其中 \chapter 只在 **book** 和 **report** 文档类有定义。这些命令生成章节标题，并能够自动编号。

article 文档类带编号的层级为 \section / \subsection / \subsubsection 三级； 

report/book 文档类带编号的层级为 \chapter / \section / \subsection 三级。

#### 生成目录\tableofcontents

#### 文档结构的划分 

所有标准文档类都提供了一个 \appendix 命令将正文和附录分开2，使用 \appendix 后，最高一级章节改为使用拉丁字母编号，从 A 开始。 book 文档类还提供了前言、正文、后记结构的划分命令： 

**\frontmatter** 前言部分，页码为小写罗马字母格式；其后的 \chapter 不编号。 

**\mainmatter** 正文部分，页码为阿拉伯数字格式，从 1 开始计数；其后的章节编号正常。 

**\backmatter** 后记部分，页码格式不变，继续正常计数；其后的 \chapter 不编号。 

#### 标题/作者/日期

```latex
\title{⟨title⟩} \author{⟨author⟩} \date{⟨date⟩}
```

### 环境

#### 列表enumerate/itemize

LATEX 提供了基本的有序和无序列表环境 enumerate 和 itemize，两者的用法很类似，都用 \item 标明每个列表项。enumerate 环境会自动对列表项编号。 

```latex
\begin{enumerate}
\item …
\end{enumerate}
```

其中 \item 可带一个可选参数，将有序列表的计数或者无序列表的符号替换成自定义的符号。列表可以嵌套使用，最多嵌套四层。

```latex
\begin{enumerate}
\item An item.
\begin{enumerate}
\item A nested item.
\item[*] A starred item.
\end{enumerate}
\item Reference().
\end{enumerate}
```

```latex
\begin{itemize}
\item An item.
\begin{itemize}
\item A nested item.
\item[+] A `plus' item.
\item Another item.
\end{itemize}
\item Go back to upper level.
\end{itemize}
```

#### 文本对齐

center、flushleft 和 flushright 环境分别用于生成居中、左对齐和右对齐的文本环境。

```latex
\begin{center} … \end{center}
\begin{flushleft} … \end{flushleft}
\begin{flushright} … \end{flushright}
```

除此之外，还可以用以下命令直接改变文字的对齐方式： 

```latex
\centering \raggedright \raggedleft
```

```latex
\centering
Centered text paragraph.
\raggedright
Left-aligned text paragraph.
\raggedleft
Right-aligned text paragraph.
```

#### 引用quote/quotation

LATEX 提供了两种引用的环境：quote 用于引用较短的文字，首行不缩进；quotation 用于引用若干段文字，首行缩进。引用环境较一般文字有额外的左右缩进。

LATEX 提供了两种引用的环境：quote 用于引用较短的文字，首行不缩进；quotation 用于引用若干段文字，首行缩进。引用环境较一般文字有额外的左右缩进。 

```latex
Francis Bacon says:
\begin{quote}
Knowledge is power.
\end{quote}
```

```latex
《木兰诗》：
\begin{quotation}
万里赴戎机，关山度若飞。
朔气传金柝，寒光照铁衣。
将军百战死，壮士十年归。
归来见天子，天子坐明堂。
策勋十二转，赏赐百千强。……
\end{quotation}
```

#### 代码块verbatim

```latex
\begin{verbatim}
#include <iostream>
int main()
{
std::cout << "Hello, world!"
<< std::endl;
return 0;
}
\end{verbatim}
```

### 其它效果

#### 表格tabular

基本表格

```latex
\documentclass{article}
\begin{document}
\begin{tabular}{cc}%一个c表示有一列，格式c为居中显示(center)，表示有两列，都居中显示
(1,1)&(1,2)\\%第一行第一列和第二列  中间用&连接
(2,1)&(2,2)\\%第二行第一列和第二列  中间用&连接
\end{tabular}
\end{document}
```

横线与竖线

```latex
\documentclass{article}

\begin{document}

\begin{tabular}{|c|c|}% 通过添加 | 来表示是否需要绘制竖线
\hline  % 在表格最上方绘制横线
(1,1)&(1,2)\\
\hline  %在第一行和第二行之间绘制横线
(2,1)&(2,2)\\
\hline % 在表格最下方绘制横线
\end{tabular}

\end{document}
```

表格头部粗

```latex
\documentclass[UTF8]{ctexart}
\usepackage{booktabs} %需要加载宏包{booktabs}
\begin{document}

\begin{tabular}{ccc}
\toprule  %添加表格头部粗线
姓名& 学号& 性别\\
\midrule  %添加表格中横线
Steve Jobs& 001& Male\\
Bill Gates& 002& Female\\
\bottomrule %添加表格底部粗线
\end{tabular}

\end{document

```

**单元格合并**

```latex
\documentclass[UTF8]{ctexart}
\begin{document}

\begin{table}[!htbp]
\centering
\begin{tabular}{|c|c|c|}
\hline
\multicolumn{3}{|c|}{学生信息}\\ % 用\multicolumn{3}表示横向合并三列 
                        % |c|表示居中并且单元格两侧添加竖线 最后是文本
\hline
姓名&学号&性别\\
\hline
Jack& 001& Male\\
\hline
Angela& 002& Female\\
\hline
\end{tabular}
\caption{这是一张三线表}
\end{table}

\end{document}
```





#### 图片

LATEX 本身不支持插图功能，需要由 graphicx 宏包辅助支持。使用 latex + dvipdfmx 编译命令时，调用 graphicx 宏包时要指定 dvipdfmx 选项；而使用 pdflatex 或 xelatex 命令编译时不需要。

在调用了 graphicx 宏包以后，就可以使用 \includegraphics 命令加载图片了：\includegraphics[*⟨**options**⟩*]{*⟨**fifilename**⟩*}其中 *⟨**fifilename**⟩* 为图片文件名，可选参数为

| 参数                  | 含义                                    |
| --------------------- | --------------------------------------- |
| width=*⟨**width**⟩*   | 将图片缩放到宽度为 *⟨**width**⟩*        |
| height=*⟨**height**⟩* | 将图片缩放到高度为 *⟨**height**⟩*       |
| scale=*⟨**scale**⟩*   | 将图片相对于原尺寸缩放 *⟨**scale**⟩* 倍 |
| angle=*⟨**angle**⟩*   | 令图片逆时针旋转 *⟨**angle**⟩* 度       |

## 封面



## 常见问题

### TexStudio不显示中文

使用xelatex编译器，然后插入代码

```latex
\usepackage[UTF8]{ctex}
```

即可，texstudio设置编译器方法 选项→配置→构建→默认编译器 选择xelatex

### 粘贴时保持缩进

texstudio设置： 选项→配置→编辑器→保持缩进

## 实战

```latex
\documentclass[12pt,a4paper,UTF8,twoside,fancyhdr]{ctexart}

\usepackage{geometry}   %%修改页边距用
\geometry{left=2.8cm, right=2.8cm, top=3.8cm, bottom=3.8cm}  
\usepackage{indentfirst}
\setlength{\parindent}{2em}

\usepackage{graphicx}  % 并排放置多个图，其中subfigure 用于子图
\usepackage{tikz}    %% 图形
\usetikzlibrary{shapes,arrows,automata,calc,patterns}
\usepackage{subfig}  %% 图形中子图

\usepackage{hyperref}
\usepackage{xcolor}


\usepackage{color}  %带颜色的文本
\usepackage{amsmath}
\usepackage{amsfonts,amssymb,amsthm}

\usepackage{milstd}
\usepackage{textcomp}

\usepackage{enumitem}
\AddEnumerateCounter{\chinese}{\chinese}{}
\renewcommand{\labelenumi}{ \chinese{enumi}、 }
\renewcommand{\labelenumii}{ \arabic{enumii}. }


\pagestyle{fancy}
\usepackage{lastpage}
\fancyhead{\empty}
\lhead{《软件方法与过程》实验报告}
\chead{}
\rhead{学号: 20172430412\qquad 姓名: 梁松林}
\lfoot{}
\cfoot{共\quad\pageref{LastPage}\quad 页\qquad 第\quad\thepage\quad 页}
\rfoot{}
\renewcommand{\headrulewidth}{0.2pt} 

\begin{document}
	
	\begin{titlepage}
		\centering
		\thispagestyle{empty} %第一页不要页眉页脚
		\begin{minipage}{4cm}
			\includegraphics[width=1.3in,scale = 0.1]{zzu.png}
		\end{minipage}
		\vskip 1\baselineskip
		
		%	{\heiti\zihao{1} $\llangle$ 软件方法与过程 $\rrangle$ 		实验报告 }
		
		{\heiti\zihao{1} 《软件方法与过程》实验报告 }
		
		\vskip3\baselineskip
		
		\begin{center}
			{\zihao{1}\kaishu 个体软件过程实践(1)  \\[15pt]   }
		\end{center}
		
		\vskip3\baselineskip
		
		{{\zihao{3}\kaishu
				\hskip3cm 姓 \hskip8ex  名：\hbox to 6cm{ 李晨曦 \hfill}  \\
				\hskip3cm 学 \hskip8ex  号：\hbox to 6cm{ 20172430212  \hfill}  \\
				\hskip3cm 专 \hskip8ex  业：\hbox to 6cm{ 软件工程(卓越工程师) \hfill}  \\
				\hskip3cm 班 \hskip8ex  级：\hbox to 6cm{ 2017级1班 \hfill}  \\
				\hskip3cm 指\hskip1ex 导\hskip1ex 教\hskip1ex 师：\hbox to 6cm{葛伟力 \hfill}  \\
		}}
		\vskip3\baselineskip
		\begin{center}
			\zihao{3}\kaishu 
			信\hskip2ex息\hskip2ex工\hskip2ex程\hskip2ex学\hskip2ex院  \\
			计算机与人工智能学院    \\
			2020年3月5日    \\
		\end{center}
		
		
	\end{titlepage}
	\newpage
	\thispagestyle{empty} %第一页不要页眉页脚
	
	\begin{center}
		{\zihao{2}实验报告使用说明}
	\end{center}
	
	
	软件方法与过程是一门理论和实际应用密切结合的课程，为了使学生能够深入理解软件过程的发展和改进之路，掌握课程中重点讲解的技术和方法的应用而开设了本课程实验。其目的是使学生对软件方法与过程在理论和实践上有一个全面的认识，能够运用课程中讲述的方法解决开发中的实际问题，为今后的软件工程综合实验和实际项目开发打下坚实基础。
	
	%实验主要分为两部分，第一部分是个体软件过程实践，要求学生理解个人过程改进的方式方法，养成良好的工程开发习惯；第二部分是分组项目开发实践，要求学生通过分组开发理解RUP和AP过程模式的主要思想，通过实践掌握RUP和AP的主要方法。
	
	本实验报告主要用于第一部分实验——个体软件过程实践，请在规定学时内将实验报告中内容完成。
	
	本课程实验报告采用电子档，
	建议以PDF文件提交，
	推荐使用
	\href{https://miktex.org/}{Miktex}+
	\href{http://texstudio.sourceforge.net/}{TexStudio}生成。
	
	\newpage
	
	\setcounter{page}{1}  % 从此页开始显示阿拉伯页码
	\begin{center}
		{\zihao{2}实验一\quad 流程图使用与PSP开发过程记录}
	\end{center}
	
	\begin{enumerate}[fullwidth,itemindent=2em]
		\item 实验目的
		\begin{enumerate}[fullwidth,itemindent=3em]
			\item 复习流程图的应用;
			\item 了解PSP的过程记录方法;
			\item 了解PSP的过程记录方法;
			\item 掌握PSP的时间记录日志、作业编号日志和缺陷记录日志的使用。	
		\end{enumerate}
		
		
		\item 实验内容
		
		\qquad 设计一个函数，完成把包含两位小数的数字转换成中文金额，如输入``$1034.50$",
		输出为``$\otimes$壹仟零叁拾肆元伍角整"或``人民币壹仟零叁拾肆元伍角整"。
		\begin{enumerate}[fullwidth,itemindent=3em]
			\item 使用高级程序设计语言(C、C++、Java等)完成函数的编写;
			\item 可以将部分功能编写成函数供调用;
			\item 编写程序进行测试;
			\item 鼓励将函数封装成DLL文件，并进行跨语言的测试;
		\end{enumerate}
		
		\item 实验要求
		\begin{enumerate}[fullwidth,itemindent=3em]
			\item 个人独立完成程序，测试至没有缺陷;
			\item 在编码前做简单设计，并将设计表达记录在实验报告中;
			\item 简单设计后画出程序流程图;
			\item 注意采用良好的程序风格;
			\item 设计测试用例，并记录测试结果;
			\item 将程序开发时间在时间记录日志中记录，并在作业编号日志中总结;
			\item 将程序开发中出现的缺陷记录在缺陷记录日志中。
		\end{enumerate}
		
		
		
		
	\end{enumerate}	
	
	
	
\end{document}

```

