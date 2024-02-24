---

layout: post
title: VScode学习笔记1---使用clangd插件格式化代码
date: 2022-12-28
tags: jekyll

---
 # webassembly-1.使用ollvm和wasi-sdk混淆

标签（空格分隔）： webassembly混淆

---
webassmebly作为新型平台越来越受到重视，其中WASM的混淆也尤为重要。本文介绍一种简单的混淆方法在ubuntu22.04下使用，并且使用wasi-sdk的类C库，混淆的WASM程序可以使用C库函数实现类似功能。
#### 1. 下载ollvm并编译安装
原始的ollvm版本已经年久失修，跟不上新版本的开发，如果使用会遇到很多bug，这里我采用一个修改版的ollvm，
github地址：[Pluto-Obfuscator][1]


  下载：
  git clone https://github.com/xubenji/Pluto-Obfuscator
  下载好以后需要编译安装，
  进入项目目录，直接执行
  `bash build.sh`
  就可以编译安装ollvm了(确保机器中有编译工具，cmake，ninja-build，g++等)，这里build.sh我已经修改好了，编译安装以后的ollvm支持编译C代码到WASM。
  ![截屏2024-02-24 下午3.30.25.png-19.9kB][2]


  在/Pluto-Obfuscator/build/bin文件夹下就是编译好的工具。
  进入此目录，使用编译好的clang将C文件编译为wasm，以下是编译命令：

    /root/Pluto-Obfuscator/build/bin/clang \
                 --target=wasm32 -nostdlib   -Wl,--no-entry\
        	-Wl,--export-all        -g -O0 loop.c -o  loop.wasm

  这个时候会出现以下错误：

    clang-12: error: unable to execute command: Executable "wasm-ld" doesn't exist!
    clang-12: error: linker command failed with exit code 1 (use -v to see invocation)

### 2. 拆解编译流程，使用wasi-sdk中的链接器
我们使用wasi-sdk中的clang编译此文件并且看看链接部分的命令是什么。
下载wasi-sdk:

    wget https://github.com/WebAssembly/wasi-sdk/releases/download/wasi-sdk-21/wasi-sdk-21.0-linux.tar.gz
    tar -xzvf wasi-sdk-21.0-linux.tar.gz

loop.c内容如下：

    int main(){
    	int n1 = 10;
    	int n2 = 20;
    	if (n1 > 0){
    		for(int i = 0; i < n2; i++){
    			n1 += n2;
    		}
    	}
    	return n1;
    }
（其实函数名是不是main没有关系）
使用wasi-sdk中的clang和相同的命令编译loop.c，但是加上-v选项。

       /root/wasi-sdk-21.0/bin/clang -v \
             --target=wasm32 -nostdlib   -Wl,--no-entry\
    	-Wl,--export-all        -g -O0 loop.c -o  loop.wasm

运行后得到如下输出：

    root@localhost:~# /root/wasi-sdk-21.0/bin/clang -v              --target=wasm32 -nostdlib   -Wl,--no-entry      -Wl,--export-all        -g -O0 loop.c -o  loop.wasm
    clang version 17.0.6
    Target: wasm32
    Thread model: posix
    InstalledDir: /root/wasi-sdk-21.0/bin
     "/root/wasi-sdk-21.0/bin/clang-17" -cc1 -triple wasm32 -emit-obj -mrelax-all -dumpdir loop.wasm- -disable-free -clear-ast-before-backend -disable-llvm-verifier -discard-value-names -main-file-name loop.c -mrelocation-model static -mframe-pointer=none -ffp-contract=on -fno-rounding-math -mconstructor-aliases -target-cpu generic -fvisibility=hidden -debug-info-kind=constructor -dwarf-version=4 -debugger-tuning=gdb -v -fcoverage-compilation-dir=/root -resource-dir /root/wasi-sdk-21.0/lib/clang/17 -isysroot /root/wasi-sdk-21.0/bin/../share/wasi-sysroot -internal-isystem /root/wasi-sdk-21.0/lib/clang/17/include -internal-isystem /root/wasi-sdk-21.0/bin/../share/wasi-sysroot/include -O0 -fdebug-compilation-dir=/root -ferror-limit 19 -fgnuc-version=4.2.1 -fcolor-diagnostics -o /tmp/loop-34b990.o -x c loop.c
    clang -cc1 version 17.0.6 based upon LLVM 17.0.6 default target wasm32-wasi
    #include "..." search starts here:
    #include <...> search starts here:
     /root/wasi-sdk-21.0/lib/clang/17/include
     /root/wasi-sdk-21.0/bin/../share/wasi-sysroot/include
    End of search list.
     "/root/wasi-sdk-21.0/bin/wasm-ld" -m wasm32 -L/root/wasi-sdk-21.0/bin/../share/wasi-sysroot/lib --no-entry --export-all /tmp/loop-34b990.o -o loop.wasm


 

注意到这条命令就是链接命令：

     "/root/wasi-sdk-21.0/bin/wasm-ld" -m wasm32 -L/root/wasi-sdk-21.0/bin/../share/wasi-sysroot/lib --no-entry --export-all /tmp/loop-34b990.o -o loop.wasm


因为编译安装的clang中没有设置wasm-ld的地址，所以clang找不到链接器，一种办法是设置环境变量，一种办法是直接使用wasi-sdk中的链接器，有一些版本的clang可能会提示找不到wasm-ld-14，wasm-ld-13等等，考虑到这种情况设置环境变量可能不太好，直接独立使用sdk中的wasm-ld会更好。
所以我们修改一下链接命令得到如下：

     "/root/wasi-sdk-21.0/bin/wasm-ld" -m wasm32 -L/root/wasi-sdk-21.0/bin/../share/wasi-sysroot/lib --no-entry --export-all /root/loop.o -o /root/loop.wasm


编译命令也要先编译为.o文件才行：
这是修改后的编译命令：

    /root/Pluto-Obfuscator/build/bin/clang \
                 --target=wasm32 -nostdlib   -Wl,--no-entry\
        	-Wl,--export-all        -g -O0 loop.c -c -o  loop.o

整合到一起就能编译loop.c到loop.wasm了。
使用runtime运行loop.wasm, 得不到任何结果，因为我们没有一些输出信息, 但是加入标准库以后clang会提示找不到c库。

    root@localhost:~/wasmtime-v17.0.1-x86_64-linux# ./wasmtime /root/loop.wasm 
    root@localhost:~/wasmtime-v17.0.1-x86_64-linux# 

### 3. 使用wasi-sdk中模拟的标准C库输出信息，并开启混淆
在loop.c中加入打印代码：

    #include <stdio.h>
    int main(){
        printf("hello world\n");
    	int n1 = 10;
    	int n2 = 20;
    	if (n1 > 0){
    		for(int i = 0; i < n2; i++){
    			n1 += n2;
    		}
    	}
    	return 0;
    }

编译这个C文件需要指定C库的地址，使用wasi-sdk中的clang编译文件命令如下：

    root@localhost:~/wasi-sdk-21.0/bin# ./clang -O0  --target=wasm32-wasi --sysroot=/root/wasi-sdk-21.0/share/wasi-sysroot/ /root/loop.c  -o /root/loop.wasm

同上我们使用-v选项查看整个链接过程，得到如下信息：

     root@localhost:~/wasi-sdk-21.0/bin#  ./clang -v -O0 --target=wasm32-wasi --sysroot=/root/wasi-sdk-21.0/share/wasi-sysroot/ /root/loop.c  -o /root/loop.wasm
    clang version 17.0.6
    Target: wasm32-unknown-wasi
    Thread model: posix
    InstalledDir: /root/wasi-sdk-21.0/bin/.
     "/root/wasi-sdk-21.0/bin/clang-17" -cc1 -triple wasm32-unknown-wasi -emit-obj -mrelax-all -dumpdir /root/loop.wasm- -disable-free -clear-ast-before-backend -disable-llvm-verifier -discard-value-names -main-file-name loop.c -mrelocation-model static -mframe-pointer=none -ffp-contract=on -fno-rounding-math -mconstructor-aliases -target-cpu generic -fvisibility=hidden -debugger-tuning=gdb -v -fcoverage-compilation-dir=/root/wasi-sdk-21.0/bin -resource-dir /root/wasi-sdk-21.0/lib/clang/17 -isysroot /root/wasi-sdk-21.0/share/wasi-sysroot/ -internal-isystem /root/wasi-sdk-21.0/lib/clang/17/include -internal-isystem /root/wasi-sdk-21.0/share/wasi-sysroot//include/wasm32-wasi -internal-isystem /root/wasi-sdk-21.0/share/wasi-sysroot//include -fdebug-compilation-dir=/root/wasi-sdk-21.0/bin -ferror-limit 19 -fgnuc-version=4.2.1 -fcolor-diagnostics -o /tmp/loop-c1b686.o -x c /root/loop.c
    clang -cc1 version 17.0.6 based upon LLVM 17.0.6 default target wasm32-wasi
    ignoring nonexistent directory "/root/wasi-sdk-21.0/share/wasi-sysroot//include/wasm32-wasi"
    #include "..." search starts here:
    #include <...> search starts here:
     /root/wasi-sdk-21.0/lib/clang/17/include
     /root/wasi-sdk-21.0/share/wasi-sysroot//include
    End of search list.
     "/root/wasi-sdk-21.0/bin/./wasm-ld" -m wasm32 -L/root/wasi-sdk-21.0/share/wasi-sysroot//lib/wasm32-wasi /root/wasi-sdk-21.0/share/wasi-sysroot//lib/wasm32-wasi/crt1-command.o /tmp/loop-c1b686.o -lc /root/wasi-sdk-21.0/lib/clang/17/lib/wasi/libclang_rt.builtins-wasm32.a -o /root/loop.wasm

使用ollvm中的clang先将文件编译为。o文件，然后链接为wasm文件。

    /root/Pluto-Obfuscator/build/bin/clang -O0 --target=wasm32-wasi --sysroot=/root/wasi-sdk-21.0/share/wasi-sysroot/ loop.c -c -o loop.o
    
    "/root/wasi-sdk-21.0/bin/./wasm-ld" -m wasm32 -L/root/wasi-sdk-21.0/share/wasi-sysroot//lib/wasm32-wasi /root/wasi-sdk-21.0/share/wasi-sysroot//lib/wasm32-wasi/crt1-command.o /root/loop.o -lc /root/wasi-sdk-21.0/lib/clang/17/lib/wasi/libclang_rt.builtins-wasm32.a -o /root/loop.wasm

使用wasmtime运行如下：

    root@localhost:~/wasmtime-v17.0.1-x86_64-linux# ./wasmtime /root/loop.wasm 
    hello world

这样我们就可以开启ollvm flattning混淆了，命令如下：

    /root/Pluto-Obfuscator/build/bin/clang -mllvm -fla -O0 --target=wasm32-wasi --sysroot=/root/wasi-sdk-21.0/share/wasi-sysroot/ loop.c -c -o loop.o

混淆前的loop.wasm和混淆后的loop_fla.wasm在ghidra中对比效果：
loop.wasm（未混淆）：
![截屏2024-02-24 下午5.58.23.png-108.7kB][3]
loop_fla.wasm（已混淆）：
![截屏2024-02-24 下午5.56.11.png-283.4kB][4]

ollvm中有许多其他混淆选项也可以试试～
  [1]: https://github.com/xubenji/Pluto-Obfuscator
  [2]: https://static.zybuluo.com/benjixu/zo7r6sj7grrncnqtufsvgmuf/%E6%88%AA%E5%B1%8F2024-02-24%20%E4%B8%8B%E5%8D%883.30.25.png
  [3]: https://static.zybuluo.com/benjixu/5mlvw3c003p254wpcnpf7sao/%E6%88%AA%E5%B1%8F2024-02-24%20%E4%B8%8B%E5%8D%885.58.23.png
  [4]: https://static.zybuluo.com/benjixu/z5tpifosjfkt8y9qogl0aeux/%E6%88%AA%E5%B1%8F2024-02-24%20%E4%B8%8B%E5%8D%885.56.11.png
