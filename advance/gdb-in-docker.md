#### docker中运行 gdb 调试代码

默认情况下，在docker中启动gdb会报错

```SHELL
root@453c2beed95f:/data/php7/test# gdb /usr/local/php7/bin/php
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /usr/local/php7/bin/php...done.
(gdb) run test.php 
Starting program: /usr/local/php7/bin/php test.php
warning: Error disabling address space randomization: Operation not permitted
warning: Could not trace the inferior process.
Error: 
warning: ptrace: Operation not permitted
During startup program exited with code 127.
```

#### Error disabling address space randomization

禁用地址空间随机化失败（gdb 应该是需要这个特性来操作程序堆栈的地址），原因是容器内的linux系统默认开启了 seccomp （Secure computing mode）
通过 `--security-opt seccomp=unconfined` 选项修改来开启权限

[官方文档：Seccomp security profiles for Docker](https://docs.docker.com/engine/security/seccomp/)


#### ptrace: Operation not permitted


原因大概（猜测）是因为在容器环境中关闭了 ptrace 的功能，因此需要在启动容器时增加选项 `docker run --cap-add=SYS_PTRACE`，开启 ptrace 权限

[官方文档：搜索 SYS_PTRACE ](https://docs.docker.com/engine/reference/run/)
