# Shell

shell是用户和computer交互的一种方式，称之为CLI（Command Line Interface），另一种方式就是大众普遍采用的GUI（Graphical User Interface），其中终端、shell、控制台是命令行界面CLI系统中极重要的几个概念，建议阅读[一文搞清控制台、终端、shell概念](https://zhuanlan.zhihu.com/p/61369678)和[What is the exact difference between a 'terminal', a 'shell', a 'tty' and a 'console'?](https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con)

简单来说：

* 终端和tty同义，都是用来处理出入输出
* shell接受终端或终端模拟器处理过的输入，进行命令的解释，然后将结果反馈给终端
* 常见的shell有很多：
  * sh（Bourne Shell），1977年由Steve Bourne开发，成为各种UNIX的标配，兼容POSIX的sh在1992年开发；
  * csh（C Shell），1978年由Bill Joy开发，随BSD UNIX发布，语法和C语言相似；
  * **bash（Bourne Again Shell）**，1989年由GNU项目开发，与POSIX标准保持一致，同时兼容sh，是Cygwin、Linux系统的标配，这些系统上sh实际是指向bash的符号链接；
  * zsh（Z Shell），1990年由Paul Falstad开发，在兼容bash基础上引入ksh、tcsh的多种特性，如自动补全、智能提示等，配置高度自由，被称为终极shell；
  * fish（Friendly Interactive Shell），2005年开发，目标是易于使用、记忆命令，和zsh是对手。

## tools

shell中会自带些二进制程序或内置命令（通过`type`查看），常见的如下：

* date：查看日期
* echo：显示后续内容，常用来查看环境变量
* which：查找命令的所在地
* pwd：查看当前工作目录
* cd：切换目录，`.`表示当前目录，`..`表示上一级目录，`～`表示家目录（直接`cd`也可以切换到家目录）
* ls：查看目录下文件
* mv：移动或重命名文件

shell中的每个程序通常都有三个标准文件：标准输入（stdin）、标准输出（stdout）和标准错误（stderr），通常情况下可以根据需求对其重定向。

The simplest form of redirection is `< file` and `> file`. These let you rewire the input and output streams of a program to a file respectively:

```shell
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

Demonstrated in the example above, `cat` is a program that con`cat`enates files. When given file names as arguments, it prints the contents of each of the files in sequence to its output stream. But when `cat` is not given any arguments, it prints contents from its input stream to its output stream (like in the third example above).当使用adb连接android设备时，通常里面没有编辑工具，就只能使用cat命令加上ctrl+D来创建文件并写入。

管道可以将前一个命令的标准输出连接到下一个命令的标准输入，可以用来实现命令的组合，非常有用。关于重定向还有一个有意思的命令为xargs，xargs的出现是为了一些命令只接受参数而不处理标准输入如ls、echo和touch等，因此xargs用来将标准输入转化为命令的参数，可参见[xargs命令详解，xargs与管道的区别](https://blog.csdn.net/tjcwt2011/article/details/120778078)，里面还有-作为标准输入的用法和一般程序读命令行参数和标准输入的优先级问题。

## Shell Scripting

```markdown
#!读法及名称由来
#和!都来自于Unix术语，#通常被称为sharp、hash或mesh，而!则被称为bang。
所以#!连在一起写作“sha-bang”、“sh-bang”或者“shebang”（最常用）
长期以来，shebang 都没有正式的中文名称。
Linux 中国翻译组的 GOLinux 将其翻译为：释伴，
即解释伴随行的简称，同时又是 shebang 的音译。
```

- `$0` - Name of the script
- `$1` to `$9` - Arguments to the script. `$1` is the first argument and so on.
- `$@` - All the arguments
- `$#` - Number of arguments
- `$?` - Return code of the previous command
- `$$` - Process identification number (PID) for the current script
- `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with sudo by doing `sudo !!`
- `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing `Esc` followed by `.` or `Alt+.`
- `$!` contains the process ID of the most recently executed background pipeline.

$(CMD)执行命令，反馈结果

<(CMD)执行命令，将输出放入文件中

## vi binding

## Experiment

```shell
cat /sys/class/power_supply/BAT0/capacity
```

查看电池状态
