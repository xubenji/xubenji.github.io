---

layout: post
title: LLVM学习笔记1之LLVM编译
date: 2022-06-11
tags: jekyll

---
llvm非常巨大，编译也是一件令人头疼的事情，我选择的编译平台是ubuntu x64。我没有选择MacOS是因为我不想把编译工作放在本地，用ubuntu的话我可以使用云主机编译，比本地更加方便。如果有什么问题我还可以推导从来。llvm改动非常频繁，之前有介绍llvm的书籍已经太老了，很多流程都已经改变了。所以我从官网和github上边把步骤记录了下来。
1，下载llvm源码。(在下载源码之前我已经安装了gcc, g++，llvm，cmake，make，git, opam, nijia-build）
    
```
git clone https://github.com/llvm/llvm-project.git

```
2，进入llvm-project文件夹以后设置编译选项。

```
cd llvm-project
cmake -S llvm -B build -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=release -DLLVM_TARGETS_TO_BUILD=X86 -DLLVM_USE_LINKER=gold
```
DCMAKE_BUILD_TYPE=release的意思是编译为release版本，如果想要debug的话这里等于debug即可，不过编译完成以后文件会变得很大。后边的意思是只编译x86平台的，最后一个参数是指令链接工具，这里更改链接工具原因是ld有点慢，我改这个似乎快点。
3, 进入build文件夹，因为刚刚指定了build文件夹。直接运行make

```
nohup make -j4 &
```
这里建议使用nohup因为我是使用的云主机编译，断线了又要重来，所以我就用了nohup。
注：如果发现配置错误等问题，建议直接删除llvm-project文件夹所有内容，然后重新git下载项目编译，我试了很多次，发生了错误即便是重新配置也没有用，硬要重新下载编译。很是蹊跷，希望有知道这些问题的小伙伴评论一下。
