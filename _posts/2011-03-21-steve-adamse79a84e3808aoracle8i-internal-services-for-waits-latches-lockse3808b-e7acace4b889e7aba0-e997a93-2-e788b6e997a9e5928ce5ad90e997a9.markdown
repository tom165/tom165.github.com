---
author: tom165
comments: true
date: 2011-03-21 12:58:33+00:00
layout: post
slug: steve-adams%e7%9a%84%e3%80%8aoracle8i-internal-services-for-waits-latches-locks%e3%80%8b-%e7%ac%ac%e4%b8%89%e7%ab%a0-%e9%97%a93-2-%e7%88%b6%e9%97%a9%e5%92%8c%e5%ad%90%e9%97%a9
title: Steve Adams的《oracle8i.internal.services.for.waits.latches.locks》-第三章 闩3.2 父闩和子闩
wordpress_id: 13
categories:
- Oracle翻译
---

3.2 父闩和子闩
大部分用闩保护的oracle内部数据结构只用一个闩来保护。但有时也用多个闩来保护。例如，有许多库缓存闩用来来保护库缓存中的不同组对象，不同的缓存链闩用来保护数据库的每个缓存散列链。

每当许多闩用来保护不同的结构部分或相同结构，这些闩叫子闩。每一相同类型的子闩集合有一个父闩。一般来说，父闩和子闩同时使用。但实际上，在父闩中你只愿意使用库缓存父闩，尽管如此，与它的子闩的活跃性相比，父闩极少发生。<!-- more -->

有点困惑的是，oracle总把没有后代的单独的闩叫做父闩。每个单独的闩在V$LATCH_PARENT视图中都有一行记录，每个真正的父闩也有一行记录。每个子闩在V$LATCH_CHILDREN中有一行记录，两个视图的结合包括了所有的闩。

不管是单独的闩或父闩和子闩的集合，oracle使用的闩的类型随oracle版本和操作系统端口的不同而变化。APT脚本 latch_types.sql能用来查看你的数据库使用的闩的类型，如果是父闩和子闩的集合，还显示出有多少子闩。例3.1显示了该脚本的一个输出摘 要：
例 3.1. latch_types.sql输出样式：
SQL> @latch_types
------ ------------------------------ ------ -------
0 latch wait list                     1       1
1 process allocation                  1
2 session allocation                  1
3 session switching                   1
4 session idle bit                    1       1
...








APT脚本和X$表
书中的许多APT脚本，如latch_types.sql, 直接使用X$表，而不是V$视图。这是因为V$视图不包含要求的信息，或因为查询V$视图会引起实例的不好的负荷。因为X$表只在SYS模式下可以查看，而且不必要的用SYS模式来工作是不好的，所以APT要求你给其他DBA模式创建暴露X$表的视图集合。这可以用 create_xview.sql脚本完成，它必须运行在SYS模式下。没有这些视图，所有的依赖X$表的APT脚本都会失效。

要注意X$表随数据库的版本变化而变化，所以APT脚本是与版本相关的，确保按你的oracle版本使用正确的脚本。




V$LATCH视图包含了按闩类型分组的统计汇总。当调查一个闩问题时，V$LATCH应当是你的参考起点。如果问题与相同类型的闩的集合相关，你应当用V$LACTH_CHILDREN查看在子闩中的活跃分布，用V$LACTH_PARENT来判断相对于父闩的活跃性。
