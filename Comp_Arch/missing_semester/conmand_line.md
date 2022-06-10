# Command-line Environment

## Job Control

在Linux中可以通过一些快捷键来发送信号，同时控制任务的执行与停止，详情可参考博客[linux中的ctrl+z和ctrl+c以及exit](https://blog.csdn.net/Castlehe/article/details/120289027)

* Ctrl-c（在较旧的 Unix 中，DEL）发送一个 INT 信号（“中断”，SIGINT）； 默认情况下，这会导致进程终止。
* Ctrl-z 发送 TSTP 信号（“终端停止”，SIGTSTP）； 默认情况下，这会导致进程暂停执行。 [3]
* Ctrl-\ 发送退出信号（SIGQUIT）； 默认情况下，这会导致进程终止并转储核心。
* Ctrl-t（并非所有 UNIX 都支持）发送 INFO 信号 (SIGINFO)； 默认情况下，如果命令支持，这会导致操作系统显示有关正在运行的命令的信息
* ctrl-s 中断控制台输出
* ctrl-q 恢复控制台输出
* ctrl-l 清屏
* jobs 显示当前暂停的进程
* bg %N 使第N个任务在后台运行(%前有空格)
* fg %N 使第N个任务在前台运行

控制字符都是可以通过stty命令更改的，可在终端中输入命令"stty -a"查看终端配置

```shell
$ stty -a
speed 38400 baud; rows 21; columns 166; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = M-^?; eol2 = M-^?; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V;
discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 hupcl -cstopb cread -clocal -crtscts
-ignbrk brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc ixany imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke -flusho -extproc
```

### Job Control Implementation

Typically, the shell keeps a list of jobs in a **job table**. Recall that a job corresponds to a process group, which consists of all the members of a [pipeline](https://en.wikipedia.org/wiki/Pipeline_(Unix)) and their descendants. The `jobs` command will list the background jobs existing in the job table, along with their job number and job state (stopped or running). When a [session](https://en.wikipedia.org/wiki/Login_session) ends when the user [logs out](https://en.wikipedia.org/wiki/Logout) (exits the shell, which terminates the *session leader* process), the shell process sends [SIGHUP](https://en.wikipedia.org/wiki/SIGHUP) to all jobs, and waits for the process groups to end before terminating itself.

The [`disown`](https://en.wikipedia.org/wiki/Disown_(Unix)) command can be used to remove jobs from the job table, so that when the session ends the child process groups are not sent SIGHUP, nor does the shell wait for them to terminate. They thus become [orphan processes](https://en.wikipedia.org/wiki/Orphan_process), and may be terminated by the operating system, though more often this is used so the processes are adopted by [init](https://en.wikipedia.org/wiki/Init) (the kernel sets their parent process to init) and continue executing as [daemons](https://en.wikipedia.org/wiki/Daemon_(computer_software)). Alternatives to prevent jobs from being terminated include [nohup](https://en.wikipedia.org/wiki/Nohup) and using a [terminal multiplexer](https://en.wikipedia.org/wiki/Terminal_multiplexer).

Session 中的每个进程组被称为一个 job，有一个 job 会成为 session 的前台 job(foreground)，其它的 job 则是后台 job(background)。

一般情况下 session 和终端是一对一的关系，当我们打开多个终端窗口时，实际上就创建了多个 session。

## Remote Machine

* `~/.ssh/authoried_keys`存放着合法用户的公钥
* `～/.ssh/config`存放远程主机的信息，可以用来边界登陆
* `～/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`分别是主机的私钥和公钥

### Key based authentication

`ssh` will look into `.ssh/authorized_keys` to determine which clients it should let in. To copy a public key over you can use:

```shell
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

A simpler solution can be achieved with `ssh-copy-id` where available:

```shell
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

### Copying files over SSH

There are many ways to copy files over ssh:

- `ssh+tee`, the simplest is to use `ssh` command execution and STDIN input by doing `cat localfile | ssh remote_server tee serverfile`. Recall that [`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) writes the output from STDIN into a file.
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) when copying large amounts of files/directories, the secure copy `scp` command is more convenient since it can easily recurse over paths. The syntax is `scp path/to/local_file remote_host:path/to/remote_file`
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) improves upon `scp` by detecting identical files in local and remote, and preventing copying them again. It also provides more fine grained control over symlinks, permissions and has extra features like the `--partial` flag that can resume from a previously interrupted copy. `rsync` has a similar syntax to `scp`.