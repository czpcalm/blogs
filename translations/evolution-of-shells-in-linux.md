# Linux shell 的演进史

> 这是一篇译文。
> 原文地址：[Evolution of shells in Linux](https://developer.ibm.com/tutorials/l-linux-shells/)。

Shell 与编辑器有个类似之处：每个人都有自己心仪的一款，而且非常排斥别的选择（还会告诉别人为什么应该换掉）。确实，不同的 shell 能提供不同的特性，不过它们都实现了数十年前发展出的核心理念。

我是在 80 年代第一次接触现代 shell 的，当时我正在开发一个 SunOS 上的软件。当了解到能够把一个程序的输出作为另一个程序的输入（甚至能用链式写法多次实现），我能轻易且高效地实现过滤器和转化。利用这一特性，能够开发出简单灵活的工具，与其它工具有效结合，发挥作用。从这点看，shell 不仅提供了与内核和设备交互的方法，还集成了一些今天软件开发中通用的设计模式（比如管道和过滤器）。

让我们从现代 shell 的一段简要历史开始，探索如今 Linux 上的一些实用且新奇的 shell。

## Shell 的一段历史

Shell，或者称命令行解析器，有一段漫长的历史，不过这里的讨论从第一个 UNIX shell 开始。Ken Thompson（来自贝尔实验室）在 1971 年为 UNIX 开发了第一个 shell，称为 V6 shell。与 [Multics](https://en.wikipedia.org/wiki/Multics) 上的类似，这个 shell (/bin/sh) 在内核外运行，是一个独立的用户程序。其他的概念是通过外部程序实现的，像 [glob](<https://en.wikipedia.org/wiki/Glob_(programming)>) （用于参数展开的模式匹配，如 `*.txt`）是在一个叫 _glob_ 的工具包中实现的，执行条件表达式的 `if` 也是如此。这种分离使得 shell 本身很小，C 语言源码不超过 900 行。

## 1977 年后的 UNIX shell

Thompson shell 之后，我们来看 1977 年出现的现代 shell -- Bourne shell。Bourne shell 是 Stephen Bourne 在 AT&T 贝尔实验室 为 V7 UNIX 所开发的，并且作为实用的 shell 保留至今（在某些情况下，作为默认 shell）。作者在继 ALGOL68 编译器之后开发了 Bourne shell，所以你会发现它的语法比起其他 shell 更像算法语言。即使源码本身用了 C 语言，还是利用宏增添了一种 ALGOL68 的味道。

Bourne shell 有两个主要的目标：作为一个命令解析器去交互式地为操作系统执行命令和脚本化编程（编写可重用的 shell 可运行脚本）。除了替代 Thompson shell，Bourne shell 还有几个优点，把控制流程，循环，变量引入了脚本，为系统操作（交互和非交互上）提供了一种更具功能性的语言。Bourne shell 还允许你把 shell 脚本作为过滤器，这集成对了信号处理的支持，但缺少函数定义的功能。最后，它集成了许多今天使用的特性，包括命令替换（使用反引号）和在脚本内嵌入原始字符串的 here 文档。

Bourne shell 不仅是一个重要的进步，现在，那些主流 Linux 系统使用的 shell，许多都以它为锚点。下图展示了一些主要的 shell 的家族关系。Bourne shell 导致了 Korn shell (ksh)，Almquist shell (ash)，Bourne Again Shell (bash) 的诞生。C shell (csh) 在 Bourne shell 发布时处于开发阶段。下图展示了主要的类系，但并非全部影响，有些未提及的 shell 也起到了重要作用。

![20220109230631](https://raw.githubusercontent.com/czpcalm/ImageHosting/master/image/20220109230631.png)

我们会在后面探讨其中的一些 shell，看到一些推动它们进步的语言和特性的例子。

## Shell 基本架构

一个假想的 shell 的基础架构是很简单的（由 Bourne shell 可见）。 从下图可以看出，这个架构类似一个流水线，在里面进行输入分析和解析，符号拓展（使用各种方法，比如大括号 `{}` 、波浪符 `~` 、变量和参数的展开/替换、文件名展开），并最终执行命令（通过 shell 内置命令或外部命令）。

![20220110123828](https://raw.githubusercontent.com/czpcalm/ImageHosting/master/image/20220110123828.png)

[这里](https://www.aosabook.org/en/bash.html) 可以了解开源的 Bash shell 的架构。

## 探索 Linux shell

现在让我们来探索一些 Linux shell，回顾它们做出的贡献，并在每小节看一个脚本例子。

### Tenex C shell

C shell 是 Bill Joy 在 1978 年为伯克利 UNIX (BSD UNIX) 开发的，他当时是加州大学伯克利分校的一名研究生。五年后，C shell 引入了 Tenex 系统（流行于 DEC PDP 系统）的功能。Tenex 增加了命令行编辑特性，还带来了文件名和命令的自动补全。Tenex C shell (tcsh) 向后兼容了 csh 并全面提升了它的交互特性。tcsh 是由 Ken Greer 在卡内基梅隆大学开发的。

C shell 的一个设计目标是开发一个与 C 语言类似的脚本语言。因为 C 语言使用广泛（而且很多操作系统是 C 语言开发的），这是一个很实用的目标。

命令历史是 Bill Joy 在 C shell 引入的一个实用特性。这一特性保留了之前执行过的命令，用户能够查看并快速选择历史命令去执行。比如，输入命令 `history` 会展示之前执行的命令，使用上下方向键选择历史命令，上一条命令能通过 `!!` 执行。甚至可以指代上一条命令的参数，比如 `!*` 代表上一个命令的所有参数，`!$` 代表上一个命令的最后一个参数。

看一个简短的 tcsh 脚本的例子。

```tcsh

#!/bin/tcsh
#找出所有可执行文件

set count=0

#参数校验
if ($#argv != 1) then
  echo "Usage is $0 <dir>"
  exit 1
endif

#确保参数是一个目录
if (! ‑d  $1) then
  echo "$1 is not a directory."
  exit 1
endif

#遍历目录，输出可执行文件
foreach filename ($1/∗)
  if (‑x $filename) then
    echo $filename
    @ count = $count + 1
  endif
end

echo
echo "$count executable files found."

exit 0
```

这个脚本接受一个参数（一个目录名），会列出该目录下所有的可执行文件并打印数量。我将在后面的例子中重用这个脚本，以展示它们的不同之处。

这个 tcsh 脚本被分为三个主要部分。首先，注意我使用了 shebang，或者叫 hashbang 符号，表明这个文件需要用指定的 shell 执行（在这个例子中，是 tcsh）。这样我能把这个文件当做普通的可执行文件，而不需要在文件前写上解析器。脚本维护了一个发现的可执行文件数量，所以我把它初始化成 0。

脚本第一部分用于验证用户传递的参数。`#argv` 变量指传递的参数数量（不包括命令名称）。你能通过下标获取到这些参数，比如，`#1` 指第一个参数（`argv[1]` 的缩写）。脚本期望接受一个参数，如果没有找到，它会报出一个错误消息，错误消息中使用了 `$0` 指代控制台输入的命令名称（`argv[0]`）。

第二部分确保传递的参数是一个目录。`-d` 运算符会在参数是目录的时候返回 `true`。不过，注意我前面使用了逻辑非运算符 `!`。这样，整个表达式会在参数不是一个目录时打印错误消息。

最后一部分遍历了目录内的所有文件，判断它们是否可执行。我使用了便利的 `foreach` 迭代器，它会循环遍历括号内（这里是目录）的所有条目，然后作为循环体的一部分执行。这个步骤使用了 `-x` 操作符来判断文件是否可执行，如果是，则输出文件，并增加计数。脚本最后输出了可执行文件的数量。

### Korn shell

David Korn 设计开发的 Korn shell (ksh) 几乎与 Tenex C shell 同时出现。ksh 引入了一个重要的特性，在向后兼容原来的 Bourne shell 的同时，还能作为脚本语言使用。

Korn shell 之前一直是专利软件，直到 2020 年才开源（以 CPL 协议）。除了对 Bourne shell 的良好兼容，Korn shell 也采用了其它 shell 的特性（如 csh 的历史功能）。Korn shell 还提供了几个与现代脚本语言 Ruby 和 Python 类似的高级功能。比如，关联数组和浮点运算。Korn shell 支持很多操作系统，包括 IBM AIX 和 HP-UX，并且力求支持 POSIX shell 语言标准。

Korn shell 是 Bourne shell 的衍生物，比起 C shell ，它与 Bourne shell 和 Bash 更像。让我们看一个用 Korn shell 查找可执行文件的例子。

```ksh

#!/usr/bin/ksh
#找出所有的可执行文件

count=0

#参数校验
if [ $#‑ne 1 ] ; then
  echo "Usage is $0 <dir>"
  exit 1
fi

#确保参数是一个目录
if [ ! ‑d  "$1" ] ; then
  echo "$1 is not a directory."
  exit 1
fi

#遍历目录，输出可执行文件
for filename in "$1"/∗
do
  if [ ‑x "$filename" ] ; then
    echo $filename
    count=$((count+1))
  fi
done

echo
echo "$count executable files found."

exit 0
```

你一眼就会发现这段代码跟上一个例子很像。结构上，它们几乎完全一样，关键区别在条件判断、表达式、迭代执行的写法。ksh 采用了 Bourne 风格的运算符，而不是 C 语言风格。

Korn shell 在迭代方面也有不同。在 Korn shell 上，使用了 `for...in` 结构，通过命令替换的方式，获取命令 `ls $1/*` 输出到标准输出的文件列表，来指代目标目录的内容。

### Bourne-Again Shell

Bourne-Again shell, 也叫 Bash，是开源项目 GNU 为了取代 Bourne shell 开发的。Bash 由 Brian Fox 开发，并成为了使用最广泛，平台支持性最好（Linux，Darwin，Windows，Cygwin，Novell，Haiku 等）的 shell 之一。就像它的名字一样，它是 Bourne shell 的超集，可以直接执行大部分 Bourne shell 脚本。

Bash 在兼容 Bourne shell 脚本编程的同时，集成了 Korn shell 和 C shell 的功能，包括命令历史，命令行编辑，目录堆栈（`pushd` 和 `popd`），一些实用环境变量，命令自动补全等。

Bash 在持续发展中引入了一些新功能，包括正则表达式（类似 Perl）和 关联数组。尽管其它脚本语言可能不支持这些特性，但可以通过其他方式兼容。正因为这点，下面的范例脚本中，除了 shebang（`/bin/bash`），其他部分与 Korn shell 中的例子一模一样。

```bash

#!/bin/bash
#查找所有可执行文件

count=0

#参数校验
if [ $#‑ne 1 ] ; then
  echo "Usage is $0 <dir>"
  exit 1
fi

#确保参数是一个目录
if [ ! ‑d  "$1" ] ; then
  echo "$1 is not a directory."
  exit 1
fi

#遍历目录，打印可执行文件
for filename in "$1"/∗
do
  if [ ‑x "$filename" ] ; then
    echo $filename
    count=$((count+1))
  fi
done

echo
echo "$count executable files found."

exit 0
```

这些 shell 的一个关键不同点是它们的软件发布协议。GNU 项目的 Bash，如你所想，是 GPL 协议的。但 csh、tcsh、zsh、ash 和 scsh 都是以 BSD 或 类 BSD 协议发布的。Korn shell 则属于 CPL 协议。

## 一些新奇的 shell

对一些喜欢尝鲜的人，你可以自己的喜好选择可替代的 shell。Scheme shell (scsh) 使用 Scheme （Lisp 语言的衍生物）提供了一套脚本环境。Pyshell 尝试使用 Python 语言来实现类似的脚本功能。最后，在嵌入式系统上，有 BusyBox，它把 shell 和所有的命令集成到一个二进制执行文件里，以此来简化它的发行和管理。

下面是查找可执行文件范例的 Scheme shell (scsh) 脚本。

```scsh

#!/usr/bin/scsh ‑s
!#

(define argc
        (length command‑line‑arguments))

(define (write‑ln x)
        (display x)
)

(define (showfiles dir)
        (for‑each write‑ln
                (with‑cwd dir
                        (filter file‑executable? (directory‑files "." #t)))))

(if (not (= argc 1))
        (write‑ln "Usage is fae.scsh dir")
        (showfiles (argv 1)))
```

这个脚本看起来有点陌生，它在功能上与上面的脚本例子是一样的。这个脚本含有三个函数，和一段用于判断参数数量的直接运行代码（在最后部分）。这个脚本的主要部分在 `showfiles` 函数内，它遍历了一个数组（在 `with-cwd` 命令后面），对数组内每一个元素，调用 `write-ln` 。这个数组是遍历目标目录并过滤出其中的可执行文件得到的。

## 结论

很多来自早期的 shell 的思想和接口保留了将近 35 年，这些都是那些作者的证明。shell 在不断地刷新提升自己，不过没有什么本质上的改变。尽管不断有新的尝试去创造一些特别的 shell，但 Bourne shell 及其衍生物一直都是使用最广泛的。
