---
layout: post
title: "Introduce to gdb"
date: 2015-03-14 22:01:22 +0800
comments: true
categories: os, debug
---

### 0. Purpose

I'm a freshman to use gdb or c, because I mainly use java, but there is some situation for java programer to use gdb tools to solve problem. So I 'mark' this article~


### 1. Debug Target

gdb could debug many kinds of programming languages, like c, c++, objective-c, and so on. It doesn't like debug tools in java, we often use it's command-line interface instead of GUI.

Before we start to debug, we should take code compile with debug information. 

for example, using `gcc` we should add `-g` flag

    $ gcc example.c -g -o example
    
This is the reason why if we want to debug jvm, we must build it with debug info first.


### 2. Run as debug

using
    
    $ gdb ./example
        
then we enter the gdb-commandline, and we can use `tab` key to complete or list all command

there are some basic command

| command|description
|--------|------------------
| dir [directories]| set source code folder 
| r | run it 
| b [where] | add a break, where is an express like 'xx_method' or linenum 
| b [where] if [condition] | break when condition 
| l | list code 
|step | step into a method 
|c| continue 
|finish| finish current method 
|p [var name]|watch a variable value 
|x/? [address] | watch the memory address 
|dump memory [file] [sadd] [eadd] | dump memory between to address into file 
|info thread | list threads 
|info break | list all break 
|thread [tid]| switch thread 
|bt | show stacktrace 
|[enter]| redo last command 

and so on...


### 2. Attach to running process

using 

    $ gdb ./example [pid]
    
to attach to pid process..


### 3. Run with coredump file

using 

    $ gdb ./example [coredump file path]
   
the [previous blog](http://lysu.github.io/blog/2015/03/07/Troubleshooting-tools/) tell some detail of debug coredump file.


Also see

- [GDB调试程序](http://blog.csdn.net/haoel/article/details/2879)











        
        