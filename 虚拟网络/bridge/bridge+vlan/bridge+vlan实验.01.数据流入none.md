# bridge+vlan实验.01.数据流入none

恢复引言中的实验网络, 开启 bridge 的 vlan 过滤功能.

```
ip netns exec ns03 ip link set dev mybr0 type bridge vlan_filtering 1
```

## 不带 vlan tag 的数据包将被丢弃

在`ns03`执行如下命令

```
$ bridge vlan del dev veth31 vid 1
$ bridge vlan show
port	vlan ids
veth31	None
veth32	 1 PVID Egress Untagged
mybr0	 1 PVID Egress Untagged
```

此时网络拓扑不变, 只是 bridge 上的接口标记发生的变动, `ns01`与`ns02`是不通的.

从`ns01`中ping 10.1.1.4, 数据包流向: veth11 -> veth31 -🚫> mybr0 -> veth32 -> veth22

在mybr0上抓包, 没有数据, 说明被 bridge 设备丢弃了.

## 带有 vlan tag 的数据包也被丢弃

从 veth11 接口创建 vid 1 的 vlan 设备, 并将 veth11 的 IP 移交给该 vlan 设备.

```
ip netns exec ns01 ip link add link veth11 name veth11.1 type vlan id 1
ip netns exec ns01 ip addr del 10.1.1.3/24 dev veth11
ip netns exec ns01 ip r flush dev veth11
ip netns exec ns01 ip addr add 10.1.1.3/24 dev veth11.1
ip netns exec ns01 ip link set veth11.1 up
```

这样发出的数据包就会带有 vid 1 的 vlan tag 了.

此时网络拓扑如下

```
+-------------------------------+-------------------------------------------------------+
|              netns1           |                  netns3                 |   netns2    |
|  10.1.1.1/24                  |                                         | 10.1.1.2/24 |
|  +----------+      +-------+  |  +-------+     +-------+     +-------+  |  +-------+  |
|  | veth11.1 ├──────┤ veth11|  |  |veth31 | <-> | mybr0 | <-> | veth32|  |  | veth22|  |
|  +----------+      +---↑---+  |  +---↑---+     +-------+     +----↑--+  |  +--↑----+  |
|      vlan              └─────────────┘                            └───────────┘       |
+-------------------------------+-----------------------------------------+-------------+
```

但仍然是不通的, mybr0上抓包, 还是没数据.

注意: 本次实验只讨论数据包流入 None 类型接口的情况, 所以此时从 10.1.1.4 处反向 ping 10.1.1.3 的数据包流向未作讨论.
