网络层地址
网络位长度和数字完全一致的地址属于同一网段

### 分类

- A类 地址范围 1.x.x.x-126.x.x.x 。前8位为网络位，后24位为主机位
- B类 地址范围 128.x.x.x-191.x.x.x。前16位为网络位，后16位为主机位
- C类 地址范围 192.x.x.x-223.x.x.x。前24位为网络位，后8位为主机位
- D类 地址范围 224.x.x.x-239.x.x.x。组播地址，不可用于配置为主机地址
- E类 地址范围 240.x.x.x-255.x.x.x。科研地址，不对外开放
- 不同分类用来划分不同网络规模

### 特殊地址

- 127.x.x.x  本地环回地址，用于标识本机。本机tcp/ip协议栈正常
- 主机位全0的地址  网络地址，用于标识某个网段（192.168.1.1-192.168.1.254）
- 主机位全1的地址（二进制全是1，11111111=>255，如192.168.1.255，网段里面最大的）,本网段广播地址
- 255.255.255.255 全网广播地址
- 0.0.0.0 任意ip地址

公网/私网地址
- 公网地址 可以在互联网上寻址的地址，全球唯一，需要运营商分配
- 私网地址  本地范围随意使用（局域网）
	- A类：10.x.x.x
	- B类：172.16.x.x-172.31.x.x
	- C类：192.168.x.x
	- 自动私有地址
	- 运营商专用私有地址