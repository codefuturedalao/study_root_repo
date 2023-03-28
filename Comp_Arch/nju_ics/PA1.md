# PA1 Simplest Machine

## Before the travel

### 检查画面，按键和声音

* 声音

  虚拟机的声音需要在Vmware works的settings中选中声卡和已连接选项。

* 按键

  克隆一个新的子项目`am-kernels`, 里面包含了一些测试程序:

  ```bash
  cd ics2022
  bash init.sh am-kernels
  ```

  然后运行其中的按键测试程序:

  ```bash
  cd am-kernels/tests/am-tests
  make ARCH=native mainargs=k run
  ```

  运行后会弹出一个新窗口, 在新窗口中按下按键, 你将会看到程序在终端输出相应的按键信息, 包括按键名, 键盘码, 以及按键状态. 

  ![image-20230327145815603](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20230327145815603.png)

* 画面

  玩一下超级玛丽，ROM[链接](http://jyywiki.cn/ICS/2021/labs/PA1)，根据fceux中的README.md运行

  ![image-20230327145037853](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20230327145037853.png)

### 使用ccache

```shell
sudo apt install ccache
```

将下面内容写入bashrc

```shell
export PATH=/usr/lib/ccache:$PATH
```

### 选择ISA

> ISA的本质就是类似这样的规范. 所以ISA的存在形式既不是硬件电路, 也不是软件代码, 而是一本规范手册.

选择riscv64，通过make menuconfig进行更改。

## 开天辟地的篇章

### 程序是个状态机

> 计算机是个状态机
>
> 既然计算机是一个数组逻辑电路, 那么我们可以把计算机划分成两部分, 一部分由所有时序逻辑部件(存储器, 计数器, 寄存器)构成, 另一部分则是剩余的组合逻辑部件(如加法器等). 这样以后, 我们就可以从状态机模型的视角来理解计算机的工作过程了: 在每个时钟周期到来的时候, 计算机根据当前时序逻辑部件的状态, 在组合逻辑部件的作用下, 计算出并转移到下一时钟周期的新状态.

程序是状态机的一个子集。

我们其实可以从两个互补的视角来看待同一个程序:

- 一个是以代码(或指令序列)为表现形式的静态视角, 大家经常说的"写程序"/"看代码", 其实说的都是这个静态视角. 这个视角的一个好处是描述精简, 分支, 循环和函数调用的组合使得我们可以通过少量代码实现出很复杂的功能. 但这也可能会使得我们对程序行为的理解造成困难.
- 另一个是以状态机的状态转移为运行效果的动态视角, 它直接刻画了"程序在计算机上运行"的本质. 但这一视角的状态数量非常巨大, 程序代码中的所有循环和函数调用都以指令的粒度被完全展开, 使得我们难以掌握程序的整体语义. 但对于程序的局部行为, 尤其是从静态视角来看难以理解的行为, 状态机视角可以让我们清楚地了解相应的细节.

## RTFSC

### 配置系统和项目构建

在真正开始阅读代码之前, 我们先来简单介绍一下NEMU项目中的配置系统和项目构建.

#### 配置系统kconfig

在一个有一定规模的项目中, 可配置选项的数量可能会非常多, 而且配置选项之间可能会存在关联, 比如打开配置选项A之后, 配置选项B就必须是某个值. 直接让开发者去管理这些配置选项是很容易出错的, 比如修改选项A之后, 可能会忘记修改和选项A有关联的选项B. 配置系统的出现则是为了解决这个问题.

NEMU中的配置系统位于`nemu/tools/kconfig`, 它来源于GNU/Linux项目中的kconfig, 我们进行了少量简化. kconfig定义了一套简单的语言, 开发者可以使用这套语言来编写"配置描述文件". 在"配置描述文件"中, 开发者可以描述:

- 配置选项的属性, 包括类型, 默认值等
- 不同配置选项之间的关系
- 配置选项的层次关系

在NEMU项目中, "配置描述文件"的文件名都为`Kconfig`, 如`nemu/Kconfig`. 当你键入`make menuconfig`的时候, 背后其实发生了如下事件:

- 检查`nemu/tools/kconfig/build/mconf`程序是否存在, 若不存在, 则编译并生成`mconf`

- 检查`nemu/tools/kconfig/build/conf`程序是否存在, 若不存在, 则编译并生成`conf`

- 运行命令`mconf nemu/Kconfig`, 此时`mconf`将会解析`nemu/Kconfig`中的描述, 以菜单树的形式展示各种配置选项, 供开发者进行选择

- 退出菜单时, `mconf`会把开发者选择的结果记录到`nemu/.config`文件中

- 运行命令

  ```
  conf --syncconfig nemu/Kconfig
  ```

  , 此时

  ```
  conf
  ```

  将会解析

  ```
  nemu/Kconfig
  ```

  中的描述, 并读取选择结果

  ```
  nemu/.config
  ```

  , 结合两者来生成如下文件:

  - 可以被包含到C代码中的宏定义(`nemu/include/generated/autoconf.h`), 这些宏的名称都是形如`CONFIG_xxx`的形式
  - 可以被包含到Makefile中的变量定义(`nemu/include/config/auto.conf`)
  - 可以被包含到Makefile中的, 和"配置描述文件"相关的依赖规则(`nemu/include/config/auto.conf.cmd`), 为了阅读代码, 我们可以不必关心它
  - 通过时间戳来维护配置选项变化的目录树`nemu/include/config/`, 它会配合另一个工具`nemu/tools/fixdep`来使用, 用于在更新配置选项后节省不必要的文件编译, 为了阅读代码, 我们可以不必关心它

所以, 目前我们只需要关心配置系统生成的如下文件:

- `nemu/include/generated/autoconf.h`, 阅读C代码时使用
- `nemu/include/config/auto.conf`, 阅读Makefile时使用

#### 项目构建和Makefiles

关于Makefile中$@和$<

```
all: library.cpp main.cpp
```

In this case:

- `$@` evaluates to `all`
- `$<` evaluates to `library.cpp`
- `$^` evaluates to `library.cpp main.cpp`

### ctags

```shell
ctags -R *
```

| Keyboard command  | Action                                         |
| ----------------- | ---------------------------------------------- |
| `Ctrl-]`          | Jump to the tag underneath the cursor          |
| `:ts <tag> <RET>` | Search for a particular tag                    |
| `:tn`             | Go to the next definition for the last tag     |
| `:tp`             | Go to the previous definition for the last tag |
| `:ts`             | List all of the definitions of the last tag    |
| `Ctrl-t`          | Jump back up in the tag stack                  |

使用ctags中遇到了怎么也没办法解析函数名的问题，后来发现在根目录生成tags文件后，需要在根目录进行vim，在子目录vim是没办法解析到的。

## The structure of code

* FCEUX: 红白机模拟项目
* NEMU：模拟器
  * monitor：不属于计算机的必要组成部分，引入方便调试。
  * CPU
    * cpu-exec.c：程序执行的主循环
  * memory
  * device
  * engine
    * interpreter：解释器的实现
* abstract-machine：抽象计算机
* am-kernels：基于抽象计算机开发的应用程序

为了支持不同的ISA, 框架代码把NEMU分成两部分: ISA无关的基本框架和ISA相关的具体实现. NEMU把ISA相关的代码专门放在`nemu/src/isa/`目录下, 并通过`nemu/include/isa.h`提供ISA相关API的声明. 这样以后, `nemu/src/isa/`之外的其它代码就展示了NEMU的基本框架. 这样做有两点好处:

- 有助于我们认识不同ISA的共同点: 无论是哪种ISA的客户计算机, **它们都具有相同的基本框架**
- 体现抽象的思想: 框架代码将ISA之间的差异抽象成API, 基本框架会调用这些API, 从而无需关心ISA的具体细节. 如果你将来打算选择一个不同的ISA来进行二周目的攻略, 你就能明显体会到抽象的好处了: 基本框架的代码完全不用修改!

API参照网站：[这个页面](https://ysyx.oscc.cc/docs/ics-pa/nemu-isa-api.html)



1. am_init_monitor or init_monitor:

   1. parse_args

   2. **init_rand**

   3. init_log

   4. **init_mem**

   5. **init_devices**

   6. **init_isa**

      ![image-20230327172115592](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20230327172115592.png)

      1. 将客户程序读入内存

      2. 初始化寄存器PC和gpr

          初始化寄存器的一个重要工作就是设置`cpu.pc`的初值, 我们需要将它设置成刚才加载客户程序的内存位置, 这样就可以让CPU从我们约定的内存位置开始执行客户程序了. 对于mips32和riscv32, 它们的0号寄存器总是存放`0`, 因此我们也需要对其进行初始化.

   7. **load_img**

   8. init_difftest

   9. init_sdb

2. engine_start

   1. cpu_exec
   2. sdb_mainloop

在真实的计算机系统中, 计算机启动后首先会把控制权交给BIOS, BIOS经过一系列初始化工作之后, 再从磁盘中将有意义的程序读入内存中执行. 对这个过程的模拟需要了解很多超出本课程范围的细节, 我们在PA中做了简化: 采取约定的方式让CPU直接从约定的内存位置开始执行.

### debug about error when typing q to exit

当键入q直接退出时，会出现Error，但如果输入c，再输入q则不会出现错误。

![image-20230328100601702](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20230328100601702.png)

1. make run -n查看可运行文件名字

2. 通过cgdb运行该文件

3. 通过调试，发现如果运行c命令，会运行默认的指令，由于默认指令有ebreak，ebreak被定义为set_nemu_state，将nemu_state.state设置为NEMU_END（2），从而在运行q时，执行is_exit_status_bad中good为1，返回值为0，并不会报错

   ![image-20230328101011140](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20230328101011140.png)

   而直接运行q，由于并没有设置系统状态为NEMU_QUIT，所以good为0，返回值为1，导致Makefile文件中产生错误。

## 基础设施

