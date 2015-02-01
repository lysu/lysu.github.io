---
layout: post
title: "Tunning CMS GC"
date: 2014-11-23 21:56:35 +0800
comments: true
categories: jvm
---
### Introduce to CMS Collector
---
'CMS'(Concurrent Mark-Sweep) is a gc algorithm that widely used by 'reponse-time critical' application.

To young generation, minor gc, CMS will stop all application threads too, but 'ParNew' performs scan in multi-threads and scan young generation is quick, because it's small.

To old generation, instead of stopping the application threads during full gc, it take more background threads to periodically scan old generation. application threads only pause when it is realy nesessary. so its overall pause time is mush less than collector like 'thoughput collector'.

The trade-off is CMS will take more CPU usage to drive scan threads. In addition, background scan don't compact old generation, so there will be many fragments in old generation. When CPU is shortage or too fragemented to allocate new object, CMS will trigger full gc to resolve the problem.

CMS is enabled by `-XX:+UseConcMarkSweepGC -XX:+UseParNewGC`.

### Common attention
---

#### 1. Never specify a heap that is larger than physical memory

it will make system do swap to hard disk, it's very very slow.

#### 2. Set both initial and max heap size to same value

it will reduce the heap resize operation, although it will hold bigger heap for it's real needing.

#### 3. Make a right size Generations

Heap is divided to many regions as we know. The Larger young generation used, the less chances young gc will occur(although slower). The smaller old generation used more full gcs will take place.

#### 4. Make Perm Generation doesn't full

Resizing perm-gen will require a full-gc which is expensive.

#### 5. Control gc threads number

set flag `-XX:ParallelGCThreads=X` will effects the number of threads used for young generation collect and stop-world-phase of old generation. Using more threads will lead to shortern stop time, it's take more CPU usage in consequence.

### Attention for CMS
----

#### 1. minor gc

The bigger young generation, the less gc will occur. The bigger young generation, the longer GC cycles will be take.

Survivor size and tenuring

TLABS..todo.

#### 2. concurrent cycle detail

Concurrent cycle will be trigger when it's sufficiently full.

It starts with `initial mark phase`, which stop application threads to find GC root object in heap.

The next is `mark phase` that will not stop application and run conncurrent with application

Then `preclean phase` will concurrent running too, then `abortable preclean phase` will be taken to wait until the young generation is about 50% full to reduce the posibility for ygc occurs before remark.

Then `remark phase` will stop application threads again.

The `sweep phase` will swap concurrently.

Concurrent cycle doesn't collect young generation directly, but it will have at least one ygc because of abortable preclean phase.

#### 3. CMS failure

##### a. concurrent mode failure

When ygc occurs and no enough room in old generaition for promoted objects. CMS will trigger a full gc.

##### b. promotion fail for too fragements

When ygc occur, but old-gen is too fragements for promented objects. in the middle of ygc(ParNew) will collect and compact entrie old generation, It will take much time then concurrent mode failure, because compact is expensive.

##### c. full gc without concurrent failure

Normally, CMS will never meet full gc, expect concurrent failure or too fragements, but when perm-gen full, will take full gc, too. so take care of PerGen.

##### d. a race

When old-gen was filled for `CMSInitiatingOccupancyFraction`(default 70%) percent, concurrent cycle start a race, CMS must complete scan and free object befor remainder(100% - CMSInitiatingOccupancyFraction) be filled up. If not will meet a concurrent mode failure..

#### 4. Solve concurrent failure problem

##### a. run concurrent cycle more often

Set `-XX:+UseCMSInitiatingOccupancyOnly` and `-XX:CMSInitiatingOccupancyFraction=N` will control the time for cycle start.

Small CMSInitiatingOccupancyFraction value will let cycle start sooner. But too small value will let concurrent cycle consume much CPU time, and start time become uncontrol when CPU resource is shortage; and so much concurrent cycle may take overall pause time increased.

##### b. run concurrent cycle more quick

Set `-XX:ConcGCThreads=N` will control the threads to run cycle. the more threads the quick cycle will have, as well as more CPU usage.

If concurrent mode failure doesn't often take, set less value will save our CPU resouce.

#### 5. CMS PermGen

PermGen will not be collect by default when using CMS. 

By set `-XX:CMSPermGenSweepingEnable` and `-XX:CMSClassUnloadingEnable` will let cms threads to collect PermGen and free class metadata.

#### 6. Use G1

If u are using big heap and fragement problem disturb u, G1 is valuable to explore.







