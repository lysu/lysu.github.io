---
layout: post
title: "How to deal with non-heap or native memory leak"
date: 2015-02-02 00:07:15 +0800
comments: true
categories: jvm 
---

### when question occur out of heap
---
This week, I meet two very strange memory problems. Althought they are in different system, they have the same appearance --- Java process use much more memory over that JVM option setted.(e.g. Jvm option use 1.5G, but in `top` command, java consume 2.9G memory).

### direct buffer memory
---

First system was a data-transfer one. It uses Netty heavily. When get the Heap dump we see many DirectByteBuffer objects, it's one of signals that we have direct buffer memory problem...then we use java.nio.Bits([see this](http://stackoverflow.com/questions/3908520/looking-up-how-much-direct-buffer-memory-is-available-to-java)) to confirm that direct buffer memory increase quickly when many request came.

At last, we found that netty sender product much requst that over receiver's ability. So, we recheck our netty configuaration, and set `WATER_MARK` option and check `channel.isWritable` before write data to confirm that Netty has avalible to handle that request, if not we will try it later. Then direct buffer memory is under our control.

### native memory
---

The other system was a Lib-scan one. It will scan many `.jar` file every day.

At first, we try to dump heap many times, but we can not found any footmark in heap dump.

So, we change tool and use `pmap -x [pid]` and `cat /proc/[pid]/smaps` to see java memory usage.

First, we see find big `anon` blocks using 65500+ memory, like this..

    00007f8b00000000   65512   40148   40148 rwx--    [ anon ]
    00007f8b03ffa000      24       0       0 -----    [ anon ]
    00007f8b04000000   65520   59816   59816 rwx--    [ anon ]
    00007f8b07ffc000      16       0       0 -----    [ anon ]
    
   
Then we using `gdb` to watch the content of these blocks

    gdp -pid [pid]
    
    dump memory mem.bin 0x00007f8b00000000 0x00007f8b00000000+65512
    
Then let it visuable

    cat mem.bin | strings
    
we found the content is `MANIFEST.MD` file.. at this time, we start to suspect jar..


    pmap -x [pid] | grep '.jar' | sort -k6 | uniq -c
    
After do some statistics,  we found that many scanned jar file using memory more then once and never give back it...It's problem.

Then open JVM code -- `jdk/src/share/native/java/util/zip/zip_util.c` and `ZipFile.c` will found ZipFile/JarFile will use `mmap` and only free then when `close()` be called..

So, after recheck and fix unclosed `ZipFile/JarFile`.. The problem was solved.


### Summary
---

Using heap dump, we can find java heap problem, but Java and Java-Libary also have chance to non-heap memory leaked. 

When use nio-libary, we must take care the direct memory usage, using libary carefully, and better let `java.nio.Bits` be monited(also this way is trick.)

In addition to `nio` problem, mis-used JNI api also lead the non-heap memory leak, too. We should try `pmap or /proc/[pid]/smaps` to found which jar/lib/stack/file might using much memory..then use `gdb` to watch content of blocks to help our analysis.