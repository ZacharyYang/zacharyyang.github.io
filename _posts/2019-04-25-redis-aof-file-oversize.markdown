---
layout: post
title: Redis aof文件过大处理办法
date: 2019-03-29 13:27:21 +0800
categories:
  - Redis(Codis-server)
---

- 目录
{:toc #markdown-toc}

Redis AOF持久化机制有点类似于MySQL的binlog，不断的讲执行的命令记录到AOF文件中。

# AOF持久化特点
    1. Redis会不断地将被执行的命令记录到AOF文件里面，所以随着Redis不断运行，AOF文件的体积也会不断增长。在极端情况下，体积不断增大的AOF文件甚至可能会用完硬盘的所有可用空间。
    2. Redis在重启之后需要通过重新执行AOF文件记录的所有写命令来还原数据集，所以如果AOF文件的体积非常大，那么还原操作执行的时间就可能会非常长。
# 解决AOF文件过大问题

为了解决AOF文件体积不断增大的问题，用户可以向Redis发送BGREWRITEAOF命令，这个命令会通过移除AOF文件中的冗余命令来重写（rewrite）AOF文件，使AOF文件的体积变得尽可能地小。BGREWRITEAOF的工作原理和BGSAVE创建快照的工作原理非常相似：Redis会创建一个子进程，然后由子进程负责对AOF文件进行重写。因为AOF文件重写也需要用到子进程，所以快照持久化因为创建子进程而导致的性能问题和内存占用问题，在AOF持久化中也同样存在。

跟快照持久化可以通过设置save选项来自动执行BGSAVE一样，AOF持久化也可以通过设置auto-aof-rewrite-percentage选项和auto-aof-rewrite-min-size选项来自动执行BGREWRITEAOF。举个例子，假设用户对Redis设置了配置选项auto-aof-rewrite-percentage 100和auto-aof-rewrite-min-size 64mb，并且启动了AOF持久化，那么当AOF文件的体积大于64MB，并且AOF文件的体积比上一次重写之后的体积大了至少一倍（100%）的时候，Redis将执行BGREWRITEAOF命令。如果AOF重写执行得过于频繁的话，用户可以考虑将auto-aof-rewrite-percentage选项的值设置为100以上，这种做法可以让Redis在AOF文件的体积变得更大之后才执行重写操作，不过也会让Redis在启动时还原数据集所需的时间变得更长。%
