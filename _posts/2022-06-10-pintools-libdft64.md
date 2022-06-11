---
layout: post
title: 调试pintools之libdft6
date: 2022-06-10
tags: jekyll 
---

intel pin的官方手册中并没有给出调试pintools的方法，只列出了调试待插桩程序的方法。后来俺又找了好久的资料终于找到调试pintools的方法了，本文以track.cpp（libdft的一个示例程序）为例。
 trak.cpp其实是一个pintools。为了调试pintools我们需要在pin启动的时候加一个参数
   以前我们启动pintools是这样启动的
   
```
../../../pin -t ./obj-intel64/trace.so -logfile trace.out -- ../../../bin/mytest
```
   mytest是你需要插桩的进程，trace.so是intel pin给的简单例子。
   调试pintools我们需要在启动pintools的时候加一个参数 -pause_tool
   
```
../../..pin -pause_tool 25 -t obj-intel64/track.so  -- obj-intel64/mini_test.exe cur_input
```
   25是需要暂停的秒数，意思是在pin启动以后暂停多少秒等待gdb连接，cur_input是mini_test.exe需要打开的文件，同学们如果说调试其他pintool没有需要打开的文件就不需要加这个
   因为libdft64是污点追踪程序。当mini_test.exe打开文件的时候就会激发hook
   我们启动一下试试
   
```

   Pausing for 25 seconds to attach to process with pid 4961
   To load the debug info to gdb use:
 *****************************************************************
   set sysroot /not/existing/dir
   file
   add-symbol-file /root/libdft64/tools/obj-intel64/track.so 0x7f0d92a34430 -s .data 0x7f0d92e458c0 -s .bss 0x7f0d92e4cf40
   *****************************************************************
```
   
   我们看到屏幕上输出了这样的信息
   我们在另外一个端口中打开gdb
   再输入attach 4961


```

   (gdb) attach 4961
   Attaching to process 4961
   Reading symbols from /root/libdft64/tools/obj-intel64/mini_test.exe...done.
   Reading symbols from /lib64/ld-linux-x86-64.so.2...Reading symbols from /usr/lib/debug//lib/x86_64-linux-gnu/ld-2.27.so...done.
   done.
   0x00007f0d96ef4a57 in ?? ()
   (gdb)
```
   
   再输入set sysroot /not/existing/dir

```
   (gdb) set sysroot /not/existing/dir
   warning: Unable to find dynamic linker breakpoint function.
   GDB will be unable to debug shared library initializers
   and track explicitly loaded dynamic code.
   (gdb)
```
   
   再输入add-symbol-file /root/libdft64/tools/obj-intel64/track.so 0x7f0d92a34430 -s .data 0x7f0d92e458c0 -s .bss 0x7f0d92e4cf40
  
```
 (gdb)  add-symbol-file /root/libdft64/tools/obj-intel64/track.so 0x7f0d92a34430 -s .data 0x7f0d92e458c0 -s .bss 0x7f0d92e4cf40
   add symbol table from file "/root/libdft64/tools/obj-intel64/track.so" at
           .text_addr = 0x7f0d92a34430
           .data_addr = 0x7f0d92e458c0
           .bss_addr = 0x7f0d92e4cf40
   (y or n) y
   Reading symbols from /root/libdft64/tools/obj-intel64/track.so...done.
   (gdb)
```
   
   现在调试信息已经被加载进来了，我们打个断点，然后c执行
 ![](upload/attach/202205/950919_BZQJXETH5SZMMMH.png)

gdb即便是-tui我也用不习惯，还是想给他套个界面，vscode+gdb并不支持attach类型的调试。。。。呃，其实还是支持的，不过launch.json文件中必须加入待调试程序的二进制文件，但是这里并没有（这里是gdb启动以后再attach，最后加载.so），所以vscode+gdb这个还是用不了。
欢迎感兴趣的同学加入秋秋裙一起研究～ 375678777


