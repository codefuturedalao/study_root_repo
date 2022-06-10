# Data Wrangling

本节课关于数据处理，会涉及到管道和shell中的一些小工具的使用

```shell
ssh myserver journalctl | grep sshd
```

* `sed` is a “stream editor” that builds on top of the old `ed` editor.

* Linux paste 命令用于合并文件的列。paste 指令会把每个文件以列对列的方式，一列列地加以合并。

* `awk` is a programming language that just happens to be really good at processing text streams. 

  `$0` is set to the entire line’s contents, and `$1` through `$n` are set to the `n`th *field* of that line, when separated by the `awk` field separator (whitespace by default, change with `-F`).
