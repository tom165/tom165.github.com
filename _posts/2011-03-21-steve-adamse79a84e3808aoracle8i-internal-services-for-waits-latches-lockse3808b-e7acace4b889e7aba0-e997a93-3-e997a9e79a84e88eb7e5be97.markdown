---
author: tom165
comments: true
date: 2011-03-21 12:59:35+00:00
layout: post
slug: steve-adams%e7%9a%84%e3%80%8aoracle8i-internal-services-for-waits-latches-locks%e3%80%8b-%e7%ac%ac%e4%b8%89%e7%ab%a0-%e9%97%a93-3-%e9%97%a9%e7%9a%84%e8%8e%b7%e5%be%97
title: Steve Adams的《oracle8i.internal.services.for.waits.latches.locks》-第三章 闩3.3 闩的获得
wordpress_id: 15
categories:
- Oracle翻译
---

3.3 闩的获得
当oracle进程需要读写被闩保护的数据结构时，有两种模式去要求获得闩，分别是愿意等待模式或不等待模式（也叫立即模式）<!-- more -->

3.3.1愿意等待模式
oracle期待闩被断时间歇的持有，所以如果一个进程试图通过愿意等待模式获得闩，并发现闩得不到，将短时自旋并重试。进程
自旋时，作为等待方式，在重试前多次执行一系列简单指令。虽然是在真正的等待，从操作系统的角度看还在使用CPU周期，所
以有时又叫做活跃等待。

在重试获得闩之前，进程占用的CPU时间数是很小的，固定的（但在oracle7里可以用_LATCH_SPIN_COUNT参数调节）。如果
下一次没有获得闩，这个过程将重复，直到达到_SPIN_COUNT参数指定的次数。在多处理器环境下这个参数通常缺省是2000次
。

3.3.1.1 为何自旋？
自旋的概念是在其他CPU上执行的其他进程可能会释放出闩来，因而允许自旋进程继续执行。当然，在单CPU机器上自旋是没有意义的，oracle也不会那样做。

自旋的替换是让出CPU，允许其他进程使用。初看起来这是个好主意。然而，要一个CPU停止一个进程并开始执行其他进程，它必须进行上下文切换。就是说， 它必须保存第一个进程的上下文，确定接下来调度的进程，恢复下个进程的上下文。一个进程的上下文本质上就是描述进程精确状态的CPU寄存器值的一个集合。

上下文切换的实现是高度依赖机器的。实际上，它是典型的用汇编语言编写的。系统提供商用尽所有办法来减少上下文的数据大小，用各种方法来优化上下文切换， 如用重影射内存地址而不是拷贝数据。尽管如此，由于要对许多内核数据结构进行查找和更新，上下文切换仍然是一种昂贵的操作。读取这些数据结构是用自旋保护 的，相当于操作系统的闩。在一个大而繁忙的系统中，上下文切换通常消耗1%到3%的CPU时间。所以如果能用短时的自旋来替换上下文切换，有些CPU时间 就会节约下来，获得闩的等待时间会被最小化。从这个原因来说，系统更倾向于短时自旋而不是立即让出CPU。

3.3.1.2 理解自旋统计值
V$LACTH类的视图中的闩统计值记录了进程在愿意等待模式下获得闩的次数。如果进程没有自旋没有获得闩，一个闩缺失会被记录。如果闩在一个或多个自旋 重复后被获得，一个闩获得会被记录。如果自旋时闩不能获得，进程会让出CPU并进入睡眠。不管进程后来唤醒，自旋，又睡眠多少次，不管将来记录的是闩的获 得还是缺失，如果闩最终通过自旋获得，只有一个闩获得被记录。所以，没有自旋获得闩的次数根本是缺失次数。我叫做简单获得次数APT脚本 latch_gets.sql显示了按简单获得次数、自旋获得次数，睡眠获得次数（睡眠获得的）的闩获得统计分析。例3.2显示了一些输出样式。
例3.2. latch_gets.sql脚本输出样式：
SQL> @latch_gets
------------------------------ ------------------ -------------- --------------
archive control                       228 100.00%       0  0.00%   0  0.00%
cache buffer handles                67399 100.00%       0  0.00% 0  0.00%
cache buffers chains           2948282897 100.00%   11811  0.00%   35999  0.00%
cache buffers lru chain          56863812  99.60%   44364  0.08%  182480  0.32%
dml lock allocation               2047579  99.99%      36  0.00%     199  0.01%
enqueue hash chains              14960087  99.95%    1139  0.01% 6603  0.04%
enqueues                         24759299 100.00%     165  0.00%     861  0.00%
...

也许更有趣的是，APT脚本lacth_spins.sql在例3.3中显示了自旋对每种闩的效果。
例3.3. latch_spins.sql脚本输出样式：

SQL> @latch_spins

LATCH TYPE                   SPIN    GETS SLEEP GETS SPIN HIT RATE
_________________________________________________________________

cache buffers lru chain             44752     182595        19.68%
redo allocation                     29218      66781        30.44%
library cache                       18997      43535        30.38%
cache buffers chains                11812      36001        24.70%
redo copy                             606      18245         3.21%
messages                             3968       8315        32.30%
enqueue hash chains                  1139       6603        14.71%
system commit number                 2312       5548        29.41%
undo global data                      252       1327        15.96%
session idle bit                      256       1198        17.61%
enqueues                              165        861        16.08%
transaction allocation                 80        535        13.01%
list of block allocation               47        353        11.75%
shared pool                           272        295        47.97%
dml lock allocation                    36        199        15.32%
global tx hash mapping                 36        184        16.36%
latch wait list                        27         95        22.13%
session allocation                     13         78        14.29%
row cache objects                      89         76        53.94%

ALL LATCHES                        114080     372833        23.43%
