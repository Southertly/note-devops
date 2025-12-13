# ip link set master报错RTNETLINK answers：Exchange full

## 问题描述

```bash
$ ip link add test_01 type veth peer name test_10
$ ip link set test_01 master mybr
RTNETLINK answers: Exchange full
```

## 原因分析

这是因为 mybr 设备中已经接入了太多网线, 满了.

当前 mybr 中接入了 1023 个设备, 没法再接入第 1024 个.
