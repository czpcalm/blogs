# Shell 脚本编程 <!-- omit in toc -->

> 本教程分为两篇：
>
> 1. 上篇《Shell 基础知识》：必看，每一个开发者都必须了解的命令行基础知识。
> 2. 下篇《Shell 脚本编程》：建议看，了解 shell 脚本编程基础，能够阅读和编写简单脚本工具。

这是下篇部分，许多内容与上篇相关，建议先阅读上篇。本文的目标是让读者对 shell 脚本有一个整体的认识，不会细致介绍命令的使用，如有需要请查阅命令手册。

## 1. Hello Shell

从一个简单的脚本开始。

**hello.sh**

```bash
#!/bin/bash

echo "Hello Shell!"
```

### 1.1. Shebang 指令

脚本的第一行以 `#!` 开始，称为 shebang 指令,用于指定运行该脚本的解析器。

当一个文件被执行时，计算机会先判断它是否是一个二进制文件，如果不是，就调用 shebang 指令指定的解析器执行该文件。

如果没有解析到 shebang，默认使用 `/bin/sh`。

如果是显式调用解析器执行脚本（见下），则 shebang 指令会被忽略不起作用。

Shebang 指令是所有脚本语言通用的。在 JS 与 Python 中，一般写成:

```bash
# JS 脚本中
#!/usr/bin/env node

# PY3脚本中
#!/usr/bin/env python3
```

因为这些解析器的安装环境不确定， `/usr/bin/env` 命令可以找到系统中的安装路径，是一种兼容性更佳的写法。

> 💡 参看 [What the #! shebang really does](https://dev.to/meleu/what-the-shebang-really-does-and-why-it-s-so-important-in-your-shell-scripts-2755)

### 1.2. 执行脚本

#### 1.2.1. a) 作为可执行文件执行

以文件名的方式直接运行文件，需要先添加执行权限：

```bash
# 给所有用户添加执行权限
chmod a+x hello.sh     # 同 chmod 755 hello.sh
./hello.sh
```

注意，这里的路径前缀 `./` 是必须的。缺少路径前缀，会被识别为命令，然后根据 `PATH` 变量中的路径查找。但 `PATH` 变量不包含当前目录，这是出于安全性的考虑，具体可以参考 [What's wrong with having '.' in your $PATH?](https://cets.seas.upenn.edu/answers/dot-path.html) 。

#### 1.2.2. b) 指定解析器执行

调用 Bash 执行脚本，这种情况下文件不需要有可执行权限，路径前缀也不是必须的，shebang 指令会被忽略。

```bash
bash hello.sh
```

#### 1.2.3. c) 作为命令执行

如果你有一个经常需要执行的脚本，希望能像命令一样去执行它。

很简单！根据上面的说法，只要你把文件移动到 PATH 变量下的任意目录即可，推荐 `/usr/local/bin`。

```bash
# 基于上面 hello.sh 例子
# 把后缀去掉，这样更像命令
mv hello.sh /usr/local/bin/hello
hello
# Hello Shell!
```

或者创建一个文件软链，这样就不用移动文件。

```bash
ln -s ./hello.sh /usr/local/bin/hello
```

不过，更推荐使用别名（见上篇）。

```bash
# 不要使用相对路径，这样可以在任何目录下调用
alias hello="bash ~/hello.sh"
hello
# Hello Shell!
```

## 2. 变量

见上篇。

## 3. 输入输出

三个内置命令，简单说下。

**输出**:

- `echo`: 打印内容到标准输出。
- `printf`: 格式化输出，类似 C 的 printf 函数。
  ```bash
  printf "保留2位小数：Pi= %1.2f" $PI
  # 3.14
  ```

**输入**:

- `read`: 从标准输入读取用户输入

  ```bash
  # 展示输入提示，用户输入值被保存到变量name中
  read -p "请输入任意值" name

  # 利用重定向，从文件中读取输入值
  read name1 name2 < file.txt
  ```

## 4. 流程控制

### 4.1. 顺序结构

命令默认以从上到下顺序执行，除非一些特殊的运算符和复合命令作用。

即使某条命令执行异常，也**会**继续执行后面的命令。这点与大部分编程语言不同。

分号 `;` 是命令的结束标识，标志着命令的结束，或者复合命令中的一部分结束。**换行符前面的分号是可以省略的**。常见的风格是使用换行，省略分号，即一条（简单）命令占一行。

在命令行中，回车用于表示执行命令，用分号可以做到一行中键入多个命令，或者执行复合命令。

```bash
# 一次性输入多个命令
command1; command2; command3

# 执行 if 命令
if true; then echo "true";fi
```

### 4.2. 条件结构

#### 4.2.1. `&&` 和 `||` 运算符

上篇提到,

```bash
test-command && command1 || command2
```

是一种简单实用的条件执行写法。如果 `test-command` 为真（状态码为 0）就执行 `command1`，否则，执行 `command2`。

#### 4.2.2. `if` 命令

```bash
if test-commands; then
    branch-commands;
[elif more-test-commands; then
    more-brach-commands;
]
[else
    alternative-commands;
]
fi
```

> 💡 为了强调语法，我会在这些复合语句中使用了显式的分号 `;` 。记住，一行中最后的分号是可以省略的，你可以调整分号和换行，选择不同的风格写法。下同。

- `[ ]` 表示 `elif` 和 `else` 分支是可选的部分,不是语法的一部分。
- 单词 `commands` 用了复数，代表这部分可以是多个命令。
- `if ... fi` 用关键字的镜像作为该结构的起止标志符，是一种常见做法。

每个 `if/elif` 分支，先执行 `test-commands` ，如果最后退出码为 0，即条件为 true，执行对应的分支命令，否则进入下一分支。

Shell 没有表达式语法，使用命令执行后的退出码进行条件判断，退出码为 0 表示 true，非 0 表示 false。

`test` 命令是一个专门用于条件判断的内置命令，它支持一系列条件表达式，当条件成立时退出码为 0，不成立时为 1。

```bash
test expr

# 或者使用简写
[ expr ]

# 还有一种拓展的test，在原有的基础上支持了正则匹配
[[ expr ]]
```

这里不展开 `test` 命令的具体使用，可以参考 [Bash test and comparison functions - IBM Developer](https://developer.ibm.com/tutorials/l-bash-test/)，或者使用 `help test` 查看帮助手册。

当判断条件为算术运算时，也经常使用算术表达式 `(( expr ))`（见下）。

下面是一个简单例子。

**file-exist.sh**

```bash
#!/bin/bash

filename=$1

if [ -e $filename ]; then
    echo "文件${filename}存在"
    if [ -d $filename ]; then
        echo "这是一个文件目录"
    elif [ -b $filename ]; then
        echo "这是一个块文件"
    elif [ -c $filename ]; then
        echo "这是一个字符文件"
    else
        ls -l $filename
    fi
else
    echo "文件${filename}不存在"
fi
```

```bash
bash file-exist.sh shell-basis.md
# 文件 shell-basis.md 存在
# -rw-r--r--  1 me  staff  23040 Jun 16 18:11 shell-basis.md
```

#### 4.2.3. `case` 命令

```bash
case word in
    [ [(] glob-pattern  ) commands ;;]…
esac
```

- 使用 glob 模式匹配，不是正则。
- 模式用括号包裹，括号左边经常省略，右括号不能省略。
- 子句必须用 `;;`,`;&` 或 `;;&` 结尾（不可省略），它们会影响执行该子句之后的行为。
  - `;;` 同 break，退出 case 结构，不再往下执行。
  - `;&` 表示直接执行下一个子句的命令（不管是否匹配），与其它语言漏掉 `break` 类似。
  - `;;&` 表示接着进入下一子句的匹配，好像没有子句命中过一样。
- 最后的 `...` 表示可以出现多个 case 子句。
- 可以在最后一个子句中使用模式 `*` 作为 `default` 分支。

同样，看一个例子。

```bash
#!/bin/bash

cat  <<TIP
你最喜欢的编程语言是？
  1) C++
  2) Java
  3) Python
请输入对应的数字：
TIP

read input_num

case $input_num in
  1 )
    lang="C++"
    echo "C++ 性能优越。"
    ;;
  2 )
  lang="Java"
  echo "Java 神通广大。"
  ;;
  3 )
  lang="Python"
  echo "Python 简单高效。"
  ;;
  * )
  echo "无效输入"
  ;;
esac
```

### 4.3. 循环结构

#### 4.3.1. `while` 命令

```bash
while test-commands;
do
  consequent-commands;
done
```

当型循环，当 `test-commands` 成立时，执行 `consequent-commands`。

#### 4.3.2. `until` 命令

```bash
until test-commands;
do
  consequent-commands;
done
```

直到型循环，执行 `consequent-commands` 直到 `test-commands` 成立，退出循环。

#### 4.3.3. `for` 命令

`for` 命令有两种语法格式：传统的 `for...in` 格式和 C 风格的 `for(( expr1; expr2; expr3 ))` 格式。

##### 4.3.3.1. `for...in`

```bash
for variable [in words];
do
  commands;
done
```

可以理解成对 `in` 后面的字符串以词（word）为单位做遍历，每次循环，把本次循环的值保存在 `variable` 变量中，可以在 `commands` 中使用该变量。

`in words` 部分内容可以省略，如果省略，默认为 `in "$@"`, 即对所有的命令参数做遍历。

举个例子：

```bash
for option in A B C D; do echo -n $option; done
# ABCD
```

`words` 部分会执行展开，可以利用这种特性快速生成遍历内容，看两个简单例子:

1. 利用大括号展开重写上例：
   ```bash
   for option in {A..D}; do echo -n $option; done
   ```
2. 利用文件名展开，遍历当前目录下所有的 `.txt` 文件：
   ```bash
   for txt_file in *.txt; do echo $txt_file; done
   ```

##### 4.3.3.2. `for(( expr1; expr2; expr3 ))`

```bash
for(( expr1; expr2; expr3 ));
do
  commands
done
```

`expr1`, `expr2`, `expr3` 都是算术表达式。注意这里使用双括号，与算术表达式语法相同。

执行结构与 C 语言 for 语句一样。等效于：

```bash
(( expr1 ))
while (( expr2 )); do
  commands
  (( expr3 ))
done
```

看个例子：

```bash
for(( option=1; option<1+4; option++ ));do echo -n $i; done
# 1234
```

## 5. 正则表达式

### 5.1. BRE 和 ERE

必须留意一点，Bash 的正则表达式语法有两种，基础正则表达式（Basic Extended Regular, BRE）和拓展正则表达式（Extended Extended Regular,ERE），并且很多地方**默认使用的是 BRE**。

它们的区别在于以下几个字符：

```txt
( ) { }  ? + |
```

在 BRE 语法下，它们是普通字符，用 `\` 转义后，才具有元字符的含义； 相反，ERE 语法下，它们属于元字符，在转义后成为普通字符。

### 5.2. 匹配模式

Shell 的正则表达式和大部分语言类似。总结如下：

**字符匹配**
| 模式 | 含义 | 示例 |
| --- | --- | --- |
| 普通字符 | 精确匹配，匹配对应的字符| `abc` 只匹配 `abc` |
| `.` | 匹配任意一个字符 | `js.` 匹配 `jsp`，`jst`，但不匹配 `json`|
| `[abc]` | 范围匹配，匹配 `abc` 中的任意一个字符 | `[ab]\.txt` 匹配 `a.txt`, `b.txt` 但不能匹配 `d.txt`, `ab.txt` </br> `[a-zA-Z]` 匹配任意一个大小写字母。
| `[^abc]` | 范围排除匹配，匹配除了 `abc` 以外的任意一个字符，用法类似 `[abc]` | `[^0-9]` 匹配任意一个非数字 |

位置匹配
| 模式 | 含义 | 示例 |
| --- | --- | --- |
| `^` | 匹配开始位置 | `^test` 匹配以 `test` 开头的字符串
| `$` | 匹配结束位置 | `\.js$` 匹配以 `.js` 结尾的字符串

数量修饰符
| 修饰符 | 含义 | 示例 |
| --- | --- | --- |
| `?` | 其前面的模式出现 1 次或 0 次 | `colou?r` 只匹配 `color` 或 `colour` |
| `+` | 其前面的模式出现 1 次以上 | `id-[0-9]+` 匹配 `id-0`，`id-44` 等，不匹配 `id-` , `id-2k` |
| `*` | 其前面的模式出现任意次 | `ahh*` 匹配 `ah`,`ahhhhh`
| `{n,m}` | 其前面的模式出现 n 次到 m 次 | `[a-z]{5,7}` 匹配 5 到 7 个小写字母 |
| `{n,}` | 其前面的模式出现 n 次及以上 | `[a-z]{5,}` 匹配 5 个以上小写字母 |
| `{,m}` | 其前面的模式出现 不超过 m 次 | `[a-z]{,7}` 匹配不超过 7 个小写字母 |

其它特殊字符
| 符号 | 含义 | 示例 |
| --- | --- | --- |
| `()` | 改变优先级，视为整体 | `py(thon)?` 匹配 `py` 和 `python` |
| `\|` | 匹配其左右任意一个模式即可（低优先级） | `py\|python` 匹配 `py` 和 `python`

### 5.3. `grep` 命令

`grep` 是一个常用的模式匹配文本处理工具。对给定的文件，它会以行为单位，打印出能匹配模式的整行文本内容。

```bash
grep [options] [pattern] [file ...]
```

```bash
# 在这篇文档中检索 shell ,-i 忽略大小写。
grep -i shell shell-scripting.md
```

如果不输入文件，会从标准输入中读取内容。利用这点，`grep` 经常被应用在流水线中，作为过滤工具，匹配出含有特定模式的内容。

```bash
history | grep "echo"
```

## 6. 算术表达式

### 6.1. 算术运算

Shell 采用和 C 语言相同的运算符和运算优先级。支持的运算包括：

- 自增、自减（包括前置后置）。即 `i++`,`i--`,`++i`,`--i`.
- 正负。即`+i`， `-i` .
- 基本算术运算：`+ - \* / %`
- 比较：`== != > < <= >=`
- 位运算：`~ & | ^ << >>`
- 逻辑运算：`! && ||`
- 三元运算：`expr1?expr2:expr3`
- 赋值：`= += -=` ...

在逻辑运算和比较运算时，用 0 表示真，1 表示假。

### 6.2. 支持算术表达式的几个命令

不能像这样直接使用算术表达式：

```bash
# 错误用法
sum=1+1; echo $sum    # 1+1
```

只有在使用特定的命令时，才会以算术表达式的形式解析。主要有：

- 算术展开 `$(( expr ))`
  ```bash
  sum=$((expr)); echo $sum    #2
  ```
- `let` 命令
  ```bash
  let sum=1+1; echo $sum     #2
  ```
- `let` 简写形式 `(( expr ))`
  ```bash
  (( sum=1+1 )); echo $sum    # 2
  ```
- `declare -i `
  ```bash
  declare -i sum=1+1; echo $sum    #2
  ```

`(( expr ))` 和 `let expr` 命令会在表达式结果非 0 时，设置退出码为 0，反之，退出码为 1。因此，它们也经常作为 `if` 命令的判断。

## 7. 函数

与其它语言一样，编写函数能有效提升代码的组织结构和代码复用。

### 7.1. 函数的定义

函数的语法可以表示成：

```bash
fname(){
  commands
  return
}
```

还有一种不推荐的写法 `function fname(){}`，目前[已经废弃](https://wiki.bash-hackers.org/scripting/obsolete)。

和其他语言类似，内置命令 `return` 用于退出函数,函数体最后一行的 `return` 可省略。

函数的定义本身也是一个命令（关键字）,它在执行环境中创建一个函数名到函数体的引用。除非发生语法错误，函数定义的退出码总是为 0。

根据脚本的顺序执行特点，函数的定义必须位于其使用之前。

### 7.2. 函数的使用

Shell 函数是可以看成命令，执行函数和执行其它命令是一样的。

```bash
fname [arguments...]
```

### 7.3. 函数内的位置变量

执行函数时，位置变量 `$N`(N>0) 和 特殊变量 `$#`, `$@`, `$*` 会被赋值成调用函数时的参数对应值，执行完成后再恢复。

**function-position-parameters.sh**

```bash
#!/bin/bash

func(){
    # $0 仍然指向脚本文件名称
    echo "\$0 = $0"
    # 其它位置参数被更新成函数调用时的参数
    echo  "参数个数 $#, 分别为 $@"
    # 函数的名称存储在环境变量 FUNCNAME 中
    echo $FUNCNAME
}

# 给这个函数传参执行
func 1 2 3
```

执行该文件后输出：

```txt
$0 = function-position-parameters.sh
参数个数 3, 分别为 1 2 3
func
```

### 7.4. 局部变量

在函数内，可以使用 `local` 命令声明局部变量。

**variable-scope.sh**

```bash
#!/bin/bash

foo(){
    local var="var in foo"
    bar
}

bar(){
    echo $var
    var="var in bar"
    echo $var
    }

var="var in global"
foo
echo $var
```

输出：

```txt
var in foo
var in bar
var in global
```

`foo` 函数中，使用 `local` 声明了一个局部变量 `var`，覆盖了同名的全局变量。
`foo` 函数调用了 `bar` 函数，`bar` 函数内向上查找到了 `var`，并且修改了局部变量。
最后，当退出函数时，局部变量释放，`var` 的值为全局变了。

如果不使用 `local`，变量默认具有全局作用域。也就是说，如果存在同名全局变量，就修改它，如果不存在，就创建一个全局变量。

为了不污染全局作用域，如果一个变量只在函数内使用，建议声明为本地变量。

## 8. 处理命令行参数

上篇说到，特殊变量与位置变量可以获取命令行参数，这里我们说一下对参数的基本处理。

### 8.1. `shift` 命令

`shift` 命令用于从左边删掉 n 个位置参数：

```bash
shift [n] # n 默认等于1
```

当我们从左到右依次处理参数时，可以使用这个命令，去掉已经处理过的参数。比如，考虑一个支持子命令的工具，对应的子命令放在其 /bin 目录下，可以这样处理工具入口：

```bash
$sub_cmd=$1;
shift
/utility_path/bin/$sub_cmd "$@"
```

### 8.2. 选项解析

上篇提到，shell 语法上并不区分参数中的选项和非选项，这些是在脚本内去解析的，一般有 3 种方式：

- 手动解析：参数复杂的时候，解析成本高。
- 内置命令 [getopts](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/getopts.html)：推荐，遵循 POSIX 规范，不过不支持长选项。
  :
- 外部命令 [getopt](https://www.tutorialspoint.com/unix_commands/getopt.htm)
  ：linux 命令，能够解析长选项。

这里以 `getopts` 为例，看看如何解析脚本的参数。

```bash
getops optstring name [arg ...]
```

其中，`optstring` 是一个选项描述，`"ab:c"` 表示期望 `"-a -b bvalue -c"` 的形式个选项，`b` 后面的冒号表示 `b` 选项后面紧跟着一个选项值。

`getopts` 应该在 `while` 循环中使用, 它每次解析一个选项，如果解析到一个选项，选项会被保存到变量 `$name` 中， 退出码为 0，进入 while 循环体；如果解析到不存在的选项，`$name` 的值为`?`，退出码依然为 0；当解析到第一个非选项的时候，退出码为 1，结束循环。

命令使用了两个隐式变量`$OPTIND` (OPTion INDex) 和 `$OPTARG` (OPTion ARGument)。`$OPTIND` 记录了下次解析的位置（从 1 开始），在每次执行脚本时被设置为 1，并在解析后累加。`$OPTARG` 记录了当前选项对应的值（如果存在）。

下面是一个脚本例子。

**logrm.sh**

```bash
#!/bin/bash

# 用法：
#*******************************#
# rmfile options files ...      #
#*******************************#
# -c 二次确认
# -m message  必须，删除备注，保存在操作日志中

confirm=0 # 默认不需要二次确认

while getopts "cm:" option
do
    case option in
        c)  confirm=1 ;;
        m)  message="$OPTARG" ;;
        ?)  echo "参数错误"; exit 1 ;;
    esac
done

shift $(($OPTIND-1))    # 去掉参数中被解析的选项部分

files="$@"
default_message="删除文件 $files"

if (( $confirm ));then
    read -p "确定删除文件？输入y确定:" input
    if [[ $input == [^yY] ]]; then
        exit 1;
    fi
fi

if rm $files; then
echo  "log: ${message:-$default_message} " > log.txt
fi
```

## 9. 数组

Bash 虽然没有数据结构，但确实有一维数组，可以方便地处理一列数据，关于数组的内容，请移步[我的另一篇文章](https://juejin.cn/post/7120225811050790919)
。

## 10. 更多资料

- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/index.html)：一份不错的 shell 脚本教程。
- [ShellCheck](https://www.shellcheck.net/)：shell 脚本检查工具。
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html): Google shell 风格指南。
