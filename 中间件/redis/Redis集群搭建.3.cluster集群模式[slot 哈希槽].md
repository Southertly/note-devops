# redis集群搭建

参考文章

1. [Redis-Cluster集群](https://www.jianshu.com/p/813a79ddf932)
2. [官网 cluster模式](http://redis.io/topics/cluster-tutorial)
    - 对集群模式做了基础但详细的介绍, 可以参考.

redis3.0之前没有集群化的解决方案, 至多只有哨兵的存在. 3.0之后redis原生带有cluster的支持. 现在部署redis集群大多推荐使用cluster模式.

cluster 集群至少包含3个节点. 如果只有三个节点的话, 它们都应该作为主节点. 可以在之后为主节点添加从节点, 也可以继续添加新的主节点, 以提高可用性与负载均衡. 从节点不会直接对外服务, 而是实现对其本身的主节点的数据同步. 当主节点挂掉, 从节点可以接替主节点的位置.

cluster集群作为一个整体对外服务, 集群将所有请求分成16384份(对应文档中`slot`的概念, 实际上比这个解释要复杂的多, 可以搜索`一致性哈希`深入研究), 每个主节点可以指定其所划分的slot个数及序号. 最初创建3主节点集群或是按照官网及大部分个人博客上的3主3从的教程来说, 三个主节点平分这16384份请求.

cluster的建立流程, 先讲述3主节点的情况.

```log
redis-trib create --replicas 1 \
192.168.0.101:6379 \
192.168.0.102:6379 \
192.168.0.103:6379 \
192.168.0.104:6379 \
192.168.0.105:6379 \
192.168.0.106:6379
```

> `--replicas 1`: 创建的集群中为每个主节点分配一个从节点, 上面的6个节点可以达到3主3从.

使用`redis-cli`到任意节点上都可以执行命令, 以下是一些比较常用的命令.

- `cluster info`: 查看集群信息, 包括集群状态(ok, fail), 节点数量等.
- `cluster nodes`: 查看集群中节点列表, 包含各节点角色(master/slave).
- `role`: 查看当前节点的角色, IP地址和端口信息, 以及在集群中的Id值

但是在写入数据的时候可能会出现如下错误

```log
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set key1 val1
(error) MOVED 9189 10.254.2.10:6379
```

...好吧, 在写入操作时需要启动cluster模式, 在`redis-cli`命令中加上`-c`选项才行.

```log
127.0.0.1:6379> set key1 val1
-> Redirected to slot [9189] located at 10.254.2.10:6379
OK
10.254.2.10:6379> keys *
1) "key1"
```

可以了.

但是, 进入任意实例的命令行, 执行`keys *`命令可能获取不到全部值, 但是使用`get key1`却可以被重定向然后得到目标值.

```log
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> get key1
-> Redirected to slot [9189] located at 10.254.2.10:6379
"val1"
```

> `keys *`只能查看当前节点上的数据, 所以显示是不全的.
