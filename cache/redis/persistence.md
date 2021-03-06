---
title: Redis 持久化 persistence
date: 2016-04-11 13:00:00
updated:
comments: true
tags:
- Redis
categories:
- Cache
- Redis
---

本文介绍 Redis 持久化。

* https://redis.io/topics/persistence

* http://www.redis.cn/topics/persistence.html

<!--more-->

# 对比

* `文件体积` RDB 小，单一文件

* `安全性` AOF 安全性高，最多丢 1s 数据

* `恢复速度` RDB 快

# RDB 快照 (Redis DataBase)

该方式为默认方式。

`RDB` 方式的持久化是通过快照（snapshotting）完成的，当符合一定条件时 Redis 会自动将内存中的 **所有** 数据进行快照并存储在硬盘上。进行快照的条件可以由用户在配置文件中自定义，由两个参数构成：`时间` 和 `改动的键的个数`。当在指定的时间内被更改的键的个数大于指定的数值时就会进行快照。配置文件中已经预置了3个条件：

```bash
save 900 1    # 900秒内有至少1个键被更改则进行快照
save 300 10   # 300秒内有至少10个键被更改则进行快照
save 60 10000 # 60秒内有至少10000个键被更改则进行快照

save "" # 禁用该功能
```

可以存在多个条件，条件之间是「或」的关系，只要满足其中一个条件，就会进行快照。如果想要禁用自动快照，只需要将所有的 `save` 参数删除即可。

Redis 默认会将快照文件存储在当前目录（可以自定义，在客户端使用 `CONFIG GET dir` 查看）的 `dump.rdb` 文件（可以自定义，在客户端使用 `CONFIG GET dbfilename` 查看）中。

配置 `dir` 和 `dbfilename` 两个参数可以分别指定快照文件的存储路径和文件名。

父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作

也可以手动执行 `SAVE` 命令

```bash
redis 127.0.0.1:6379> SAVE    # 该命令将在 redis 备份目录中创建dump.rdb文件。
```

```bash
$cat dump.rdb
REDIS0008    redis-ver4.0.1
redis-bits@ctimeYused-mem
                                aof-preamblerepl-id(484f9d49a700c4b9b136f0fd40d2d6e5a8460438
                                                                                               repl-offa;^foobarfoobar^KJ_U
```

# AOF 日志 (Append-only file)

`append only file`

将 操作 + 数据 以格式化指令的方式追加到操作日志文件的尾部

```bash
appendonly yes

appendfilename appendonly.aof

# 当目前的 AOF 文件大小超过上一次重写时的 AOF 文件大小的百分之多少时会再次进行重写，如果之前没有重写过，则以启动时的 AOF 文件大小为依据

auto-aof-rewrite-percentage 100

auto-aof-rewrite-min-size 64mb   # 允许重写的最小AOF文件大小

# appendfsync always             # 每次执行写入都会执行同步，最安全也最慢
appendfsync everysec             # 每秒执行一次同步操作
# appendfsync no                 # 不主动进行同步操作，而是完全交由操作系统来做（即每30秒一次），最快也最不安全
```

Redis 允许同时开启 `AOF` 和 `RDB`。

```bash
$cat appendonly.aof
*2
$6
SELECT
$1
0
*3
$3
set
$3
```

# 混合 RDB-AOF 持久化格式 4.0+

先以 RDB 格式写入全量数据再追加增量日志

* https://yq.aliyun.com/articles/193034

```bash
aof-use-rdb-preamble yes
```

```bash
$cat appendonly.aof
REDIS0008    redis-ver4.0.1
redis-bits@ctimeYused-memP
                                  aof-preamblerepl-id(484f9d49a700c4b9b136f0fd40d2d6e5a8460438
                                                                                                 repl-offsetfoobar?I    Y*2
$6
SELECT
$1
0
*3
$3
set
$3
foo
$3
bar
```

前半段是 RDB 格式的全量数据后半段是 redis 命令格式的增量数据。

# 恢复

如果需要恢复数据，只需将备份文件 `dump.rdb` 或 `appendonly.aof` 移动到启动配置文件中设置的 `dir` 目录并启动服务即可。

注意：  

* 当配置文件启用 `appendonly` 时，redis 默认寻找 `appendonly.aof` 恢复数据，如果没有 `aof` 文件，则 `redis` 数据为空。

* 当需要使用 `rdb` 文件恢复数据时，启动配置文件需注释掉 `#appendonly yes` 参数（或者将参数值改为 `no`）。
