# PA0 开发环境

1. Linux ：Ubuntu 22.04

2. Some Commands

   ```shell
   $ df -h
   ```

   df 是从总体上统计系统各磁盘的占用情况，不能统计具体的文件夹或文件的大小。
   du 既可以从总体上统计，又可以统计具体的某个文件夹或文件的大小。

   ```shell
   $ poweroff
   ```

   ```shell
   $ su -
   ```

   -表示是否切换环境变量

3. Install Tools

   1. 更新APT源

      ```shell
      bash -c 'echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" > /etc/apt/sources.list'
      ```

   2. 修改root密码

      Ubuntu的默认root密码是随机的，即每次开机都有一个新的root密码。可以在终端输入命令 `sudo passwd`，然后输入新密码即可。

   3. 下载所需库

      遇到版本不匹配的问题，参见以下教程[Debian: "apt install build-essential" fails because of unmet dependencies](https://serverfault.com/questions/993576/debian-apt-install-build-essential-fails-because-of-unmet-dependencies/1046470#1046470)，一开始可以解决问题，但是电脑重启后无法进入图形界面系统崩溃，采用中科大的源解决了问题。

4. Configure Vim

   由于我本身在github存有dotfile，通过rcm进行管理，因此可进行以下操作：

   1. 运行命令`git clone git@github.com:codefuturedalao/dotfiles.git`
   2. 运行命令`rcup -d [path]`

   3. 接下来会遇到一些vim插件的问题，运行`./startup.sh`命令，然后Launch `vim` 运行`:PluginInstall`

5. More Exploration

   1. gdb

      https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html

   2. makefile

      https://seisman.github.io/how-to-write-makefile/introduction.html

   3. tmux

      更新rc 文件

6. Getting Source Code for PAs

   1. 运行以下命令

      ```shell
      git clone https://github.com/NJU-ProjectN/ics-pa.git
      ```

   2. 修改init.sh中的git协议为http

   3. 运行以下命令

      ```shell
      bash init.sh nemu
      source ~/.bashrc
      bash init.sh abstract-machine
      source ~/.bashrc
      ```

   4. 编译nemu

      运行```make menu-config```和```make```，将未安装的库通过apt install下载下来。



