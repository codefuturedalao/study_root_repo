# 南京大学  计算机系统基础

PA链接：https://nju-projectn.github.io/ics-pa-gitbook/ics2021/index.html#%E5%A6%82%E4%BD%95%E8%8E%B7%E5%BE%97%E5%B8%AE%E5%8A%A9

理解"程序如何在计算机上运行"的根本途径是从"零"开始实现一个完整的计算机系统. 南京大学计算机科学与技术系`计算机系统基础`课程的小型项目 (Programming Assignment, PA)将提出x86/mips32/riscv32(64)架构相应的教学版子集, 指导学生实现一个经过简化但功能完备的x86/mips32/riscv32(64)模拟器NEMU(NJU EMUlator), 最终在NEMU上运行游戏"仙剑奇侠传", 来让学生探究"程序在计算机上运行"的基本原理. NEMU受到了[QEMU](http://www.qemu.org/)的启发, 并去除了大量与课程内容差异较大的部分. PA包括一个准备实验(配置实验环境)以及5部分连贯的实验内容:

- 图灵机与简易调试器
- 冯诺依曼计算机系统
- 批处理系统
- 分时多任务
- 程序性能优化

