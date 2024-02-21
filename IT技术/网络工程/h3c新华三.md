## 命令行

### 命令行视图

- 用户视图 `<h3c>`，只能查看配置
- 系统视图`[h3c]`,可以查看和修改全局配置
- 接口视图`[h3c-GigabitEthernet0/0]`,可以修改接口配置

### 视图切换

- system-view 从用户视图进入系统视图
- inferface g0/0 从系统视图进入接口视图
- quit 返回上层视图
- return 直接返回到用户试图，快捷键 Ctrl+Z

### 常用查看命令

- display current-configuration 查看当期所有配置
- display ip interface brief 查看所有三层接口的摘要信息，接口的ip地址，接口的连通状态
- display interface g0/0 查看g0/0接口的详细信息
- display version 查看设备硬件版本信息
- display this 查看当前试图下配置了哪些参数

### 设备操作命令

- sysname R1 更改设备名称为R1 系统视图
- reboot 重启设备 用户视图
- save 保存当前配置 系统视图
- reset saved-configuration 清空保存的配置 恢复出厂设置，重启后生效
- undo 在命令前加undo代表该命令的反向操作（撤销）
- shutdown 手动关闭接口 接口视图
- undo shutdown 手动开启接口

### 命令行帮助

- 命令简写
- ?键命令提示，s? 查看以s开头的命令
- Tab键自动补齐命令

## 设备文件管理

### 设备存储器

- ROM 只读存储器，存储了Bootrom程序，在Bootrom模式下可以破解密码
- RAM 内存，存储当前正在运行的数据，断电数据会消失
- Flash 内存，永久存储操作系统文件，配置文件等数据

### 设备的配置文件

- 当前配置
- 起始配置
- 起始文件的备份

## 网络设备调试

### Debug

- 开启debug命令 ,用户视图
	- terminal monitor
	- terminal debugging
	- debugging 具体的协议
	
- 结束debug
	- undo debugging all
	- Ctrl+O

## 以太网交换机工作原理

### 以太网

- 定义 传输标准Ethernet II类型帧的网络
- 特征 多路访问，广播式的网络 （不清楚对方在哪，需要广播)
- Mac地址 
	- 每台网络设备生产时就写入一个全球唯一的物理地址
	- 48位长度，16进制格式地址
	- 前24位为厂商标志（oui）,后24位是设备标识
- 以太网帧格式
	- 目的Mac地址
	- 源Mac地址
	- 服务和类型
	- DATA
	- 帧校验序列（checksum）

## VLAN

虚拟局域网

### VLAN工作原理

- 端口类型
	-  Access 必须加入到一个vlan,只能加入到一个vlan；收到帧打上所属vlan的tag，发出帧剥离tag。一般用来连接pc或路由器。h3c交换机默认所有端口都是access类型，属于vlan1;华为是hybrid
	- Trunk 一般用于连接交换机， 可以允许多个vlan端口通过，发出帧不玻璃tag，由接收方去剥离（不然无法找到对应的目标）。不同vlan跨交换机互通（如果是缺省vlan,发出端口剥离 缺省vlan tag,接收方端口打上自己的默认vlan tag）
	- Hybrid 既可以连pc也可以连路由器。可以允许多个vlan的数据通过，可以手动配置从hybrid端口发出的帧，哪个vlan保留tag,哪个vlan剥离tag,hybrid收到未打tag的帧，会重新打上缺省vlan的tag。

- pvid
	- 表示某个端口的缺省vlan
	- Access端口所属vlan就是pvid,不用配置，默认就是vlan1
	- Trunk端口需要手动配置，默认是vlan1
	- Hybrid端口需要手动配置，默认是vlan1

## VLAN类型

- 基于端口的vlan 端口固定于某个vlan
- 基于Mac地址的vlan Mac地址绑定到vlan,同一Mac地址的设备，无论连接在哪个端口，vlan归属不变，端口类型需要配置为bybrid
- 基于协议的vlan，三层协议(ip)绑定到vlan，同一协议的报文，无论从哪个端口接收，vlan归属不变，端口类型需要配置为bybrid
- 基于IP子网的vlan ip网段绑定到vlan,同一ip子网的设备，无论连接在哪个端口，vlan归属不变。需要hybrid类型
- vlan归属优先级 ：Mac>ip子网>协议>端口

## VLAN常用命令

- [h3c]vlan 'vlan id' 创建vlan,进入vlan视图
- [h3c-vlan10] name 'text' vlan命名
- [h3c-vlan10] port 'port id ' 把端口以Access类型加入到vlan
- [h3c-GigabtEhernet1/0/1]port link-type 'access/trunk/hybrid' 配置端口类型
- [h3c-GigabtEhernet1/0/1]port trunk premit vlan 'vlan-id-list/all' 配置trunk端口允许通过的vlan
- [h3c]display vlan 'valn id' 查看某个vlan的详细信息
- [h3c]display vlan brief 查看vlan摘要信息
- [h3c]display port trunk 查看Trunk端口信息
- [h3c-GigabtEhernet1/0/1]undo port link-type 还原交换机的默认端口类型

交换机mac地址表老化时间，300秒

## 生成树协议（Spanning Tree Protocol）

### 二层环路带来的问题

- 广播风暴
- Mac地址表震荡

### 生成树定义

- STP，用来解决二层环路问题

### STP相关概念

- BPDU 报文
	- 定义 桥协议数据单元，用于传递STP协议相关报文
	- 分类 配置BPDU 用于传递 STP配置信息，TCN BPDU 用于通告拓补变更信息

- STP选举机制 
	- 在所有交换机中选举出一台根网桥（Root bridge）
		-  选举规则 bridge id小的优先
		- bridge-id: 
			- 定义 ：桥id即bid，用来标识交换机身份
			- 格式
				- 优先级+Mac地址
				- 优先级默认32768，必须是4296的倍数

	- 每台非根网桥（交换机）选举出一个根端口（Root Port）
		- 选举规则
			- 1. 到达根网桥开销小的优先
			- 2. 对端交换机BID小的优先
			- 3.端口id小的优先

		- 开销 
			- Cost ,代表路径耗费的代价和成本，贷款越大，耗费越小

	- 每个物理段上选举出一个指定端口（Designated port）
		- 选举机制
			- 1. 到达根网桥开销小的优先
			- 2.本机机BID小的优先
			- 3.端口id小的优先

	- 剩下没有角色的端口就是阻塞端口（Blocked Port）

- STP初始化流程，交换机端口状态
	- disable 禁用状态，被关闭的端口
	- blocking 阻塞状态 接收BPDU ，但不发送BPDU，不学习Mac地址，不转发数据
	- listening 接收并发送BPDU ，不学习Mac地址，不转发数据，持续15秒
	- learning 学习状态 接收并发送BPDU，学习Mac地址，不转发数据，持续15秒
	- forwarding 转发数据 接收并发送BPDU，学习Mac地址，转发数据

- STP计时器
	- Hello time 2秒，配置 BPDU的发送周期
	- Max age 20秒 ，判断链路故障的时间，10个Hello time周期
	- Forwarding delay 15秒，状态切换延迟

- STP拓扑变更机制
	1. Max age超时/有接口变更为转发装填，判断为拓扑发生变更，向根网桥发起TCN BPDU
	2. 收到TCN BPDU的交换机继续向根网桥转发TCN BPDU，到达根网桥为止
	3. 根网桥收到TCN BPDU后，向所有端口发起TC置位的配置BPDU
	4. 交换机收到TC置位的配置BPDU后，MAC地址表的老化时间缩短为15秒

- STP的问题
	- 收敛速度慢，故障切换时间太长
	- 网络中大量主机频繁上下线，会导致TCN BPDU以及TC置位BPDU大量发送

- RSTP 快速生成树协议
- MSTP 多生成树协议
- STP常用命令
	- [h3c] display stp 查看STP相关信息
	- [h3c] display stp brief 查看STP端口状态
	- [h3c] stp global enable 全局启用STP
	- [h3c-GigabitEthernet 1/0/1]undo stp enable 关闭端口上STP
	- [h3c]stp mode 'stp/rstp/mstp' 更改STP模式 ，默认模式是mstp
	- [h3c] strp priority 更换交换机优先级
	- [h3c-GigabitEthernet 1/0/1] stp cost 'cost' 更改接口生成树的cost
	- [h3c-GigabitEthernet 1/0/1] stp edged-port 配置端口为边缘端口

## 链路聚合

### 定义

- 把连接到同一台交换机上的多个物理端口捆绑为一个逻辑端口

### 功能
- 提高链路可靠性，聚合组内只要还有物理端口存活，链路就不会中断
- 增加链路传输带宽 
	- 避免了STP计算，聚合组内物理端口不会被闭塞
	- 交换机之间的流量会自动在聚合组内所有物理端口上负载分担

- 负载分担 聚合后的链路会基于流自动负载分担

### 分类

- 静态聚合 双方不会协商聚合参数
- 动态聚合 双方通过LACP协议协商聚合参数

### 常用命令

- [h3c] interface bridge-aggregation ‘number' 创建聚合端口，进入聚合端口视图
- [h3c-GigabiEhternet 1/0/1] port link-aggregation group 'number' 物理接口加入聚合组
- [h3c] display link-aggregation summary 查看聚合链路信息
