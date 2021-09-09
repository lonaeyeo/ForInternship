<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuPf6AqOc2paEIcMBPL2qd8ctN6iaSE3DLNWflY4r0o9LHGGqpl3mISxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />  

## IP协议

网络层的主要作用是：**实现主机与主机之间的通信，也叫点对点（end to end）通信。**

**<u>*网络层和数据链路层的区别*</u>**

**MAC** 的作用则是**实现「直连」的两个设备之间通信**，而 **IP 则负责在「没有直连」**的两个网络之间进行通信传输。

源IP地址和目标IP地址在传输过程中是不会变化的，只有**源 MAC 地址和目标 MAC 一直在变化**。



### IPV4基础知识

IP 地址最大值也就是<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuP5EBBdIcDPBSSS6uMXjQoLlfXIL9uIVLW1kVuu5MFE5lvSQQz1Wq1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 40%;" /> 

也就说，最大允许 43 亿台计算机连接到网络。但实际上，IP 地址并不是根据主机台数来配置的，而是以网卡。像服务器、路由器等设备都是有 2 个以上的网卡，也就是它们会有 2 个以上的 IP 地址。

#### IP地址的分类

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yu3u9AgedCb0tFU7W1fZlqG0ImZVnIoAan7z2vX71WYeKZz1waagqoXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

A: 0xxxxxxx **xxxxxxxx xxxxxxxx xxxxxxxx**

B: 10xxxxxx xxxxxxxx **xxxxxxxx xxxxxxxx**

C: 110xxxxx xxxxxxxx xxxxxxxx **xxxxxxxx** 最大主机个数为256-2=254，因为主机号全1用于广播，全为0指定某个网络。

D: 用于多播，多播用于**将包发送给特定组内的所有主机。**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuYzUxC95uwtWFIFnlXVwcjqSdQSylF2wiaIuibboOEafdKYpYgU2aqNFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



#### 广播

广播地址用于在**同一个链路中相互连接的主机之间**发送数据包。

广播地址可以分为本地广播和直接广播两种。

- **在本网络内广播的叫做本地广播**。例如网络地址为 192.168.0.0/24 的情况下，广播地址是 192.168.0.255 。因为这个广播地址的 IP 包会被路由器屏蔽，所以不会到达 192.168.0.0/24 以外的其他链路上。
- **在不同网络之间的广播叫做直接广播**。例如网络地址为 192.168.0.0/24 的主机向 192.168.1.255/24 的目标地址发送 IP 包。收到这个包的路由器，将数据转发给192.168.1.0/24，从而使得所有 192.168.1.1~192.168.1.254 的主机都能收到这个包（由于直接广播有一定的安全问题，多数情况下会在路由器上设置为不转发）。

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yucqib29r2nmgSNdhueuWH5nSYlLSu70pcZM5NSKSaatp8HfRicyrNWicBw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:40%;" /> 



#### 单播、广播和多播（组播）

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuPHVENpfzl2iaODOyy0rjcseRJ3sJV1pVQDORnb8VAoAdCFrBWSzGkYw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:40%;" /> 



#### IP分类的缺点

***缺点一：***同一网络下没有地址层次

***缺点二：***

A、B、C类有个尴尬处境，就是**不能很好的与现实网络匹配**。

- C 类地址能包含的最大主机数量实在太少了，只有 254 个，估计一个网吧都不够用。
- 而 B 类地址能包含的最大主机数量又太多了，6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。

这两个缺点，都可以在 `CIDR` 无分类地址解决。



### 无分类地址CIDR

不再有分类地址的概念，32 比特的 IP 地址被划分为两部分，前面是**网络号**，后面是**主机号**。

表示形式 `a.b.c.d/x`，其中 `/x` 表示前 x 位属于**网络号**， x 的范围是 `0 ~ 32`，这就使得 IP 地址更加具有灵活性。

举个栗子：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yu4fzGvqbHIHM5cmOibtsrzNjoicPWlXUibaib37H0w0DeMo8iahxiafTEOsMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



**子网掩码：掩盖掉主机号，与运算。**

子网划分实际上是将主机地址分为两个部分：**子网网络地址**和**子网主机地址**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yutNlCIP4A8dbFcQbJfAU0GhGTuDFicyO5ibRZBOmyPfr6OTgOvk9ksicaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210328183803626.png" alt="image-20210328183803626" style="zoom:60%;" /> 



**公有IP地址与私有IP地址**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuDe8iajBuHpINIYkU6VIiboL5mJfHwXOsy157Vh1gr3iakgicUVynPFAwIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



### IP地址与路由控制

IP地址的网络地址是用于进行路由控制。

路由控制表中**记录着网络地址**与**下一步应该发送至路由器的地址**，在主机和路由器上都会有各自的路由器控制表。

在发送 IP 包时，首先要确定 IP 包首部中的目标地址，再从路由控制表中找到与该地址具有**相同网络地址**的记录，根据该记录将 IP 包转发给相应的下一个路由器。如果路由控制表中存在多条相同网络地址的记录，就选择相同位数最多的网络地址，也就是最长匹配。

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210328184310507.png" alt="image-20210328184310507" style="zoom: 50%;" /> 

1. 主机 A 要发送一个 IP 包，其源地址是 `10.1.1.30` 和目标地址是 `10.1.2.10`，由于没有在主机 A 的路由表找到与目标地址 `10.1.2.10` 的网络地址，于是把包被转发到默认路由（路由器 `1` ）
2. 路由器 `1` 收到 IP 包后，也在路由器 `1` 的路由表匹配与目标地址相同的网络地址记录，发现匹配到了，于是就把 IP 数据包转发到了 `10.1.0.2` 这台路由器 `2`
3. 路由器 `2` 收到后，同样对比自身的路由表，发现匹配到了，于是把 IP 包从路由器 `2` 的 `10.1.2.1` 这个接口出去，最终经过交换机把 IP 数据包转发到了目标主机



#### 环回地址

127.0.0.1，localhost



### IP分片与重组

常见数据链路是以太网，MTU是1500字节。IP数据包 > MTU，就会被分片。

在分片传输中，一旦某个分片丢失，则会造成整个 IP 数据报作废，所以 TCP 引入了 `MSS` 也就是在 TCP 层进行分片不由 IP 层分片，那么对于 UDP 我们尽量不要发送一个大于 `MTU` 的数据报文。



## IPV6初识

**128位**

 IPv4 和 IPv6 不能相互兼容，不但需要电脑、手机之类的设备支持，还需要网络运营商对现有的设备进行升级，所以IPv6普及率比较慢。

### 亮点

- IPv6 可自动配置，即使没有 DHCP 服务器也可以实现自动分配IP地址，真是**便捷到即插即用**啊。
- IPv6 包头包首部长度采用固定的值 `40` 字节，去掉了包头校验和，简化了首部结构，减轻了路由器负荷，大大**提高了传输的性能**。
- IPv6 有应对伪造 IP 地址的网络安全功能以及防止线路窃听的功能，大大**提升了安全性**。



### 地址结构

IPv6 地址长度是 128 位，是以每 16 位作为一组，**每组用冒号 「:」 隔开**。

如果出现连续的 0 时还可以将这些 0 省略，**并用两个冒号 「::」隔开**。但是，一个 IP 地址中只允许出现一次两个连续的冒号。

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210328192659765.png" alt="image-20210328192659765" style="zoom:67%;" /> 

IPv6 的地址主要有一下类型地址：

- 单播地址，用于一对一的通信
- 组播地址，用于一对多的通信
- 任播地址，用于通信最近的节点，最近的节点是由路由协议决定
- 没有广播地址

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuAK4X5GADdLZNMcL8sqTuLyKHpd3LHhhM42qpLuNpF7okxk4Z6pFRlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



#### IPv6单播地址类型

于一对一通信的 IPv6 地址，主要划分了三类单播地址，每类地址的有效范围都不同。

- 在同一链路单播通信，不经过路由器，可以使用**链路本地单播地址**，IPv4 没有此类型
- 在内网里单播通信，可以使用**唯一本地地址**，相当于 **IPv4 的私有 IP**
- 在互联网通信，可以使用**全局单播地址**，相当于 **IPv4 的公有 IP**



#### IPv4首部与IPv6首部

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuu9TDgTaJve4iaTh8icMqQpl3RAXz7VMCNh6NH3ESYOxic5icuygKLHhk4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

IPv6 相比 IPv4 的首部改进：

- **取消了首部校验和字段。** 因为在数据链路层和传输层都会校验，因此 IPv6 直接取消了 IP 的校验。
- **取消了分片/重新组装相关字段。** 分片与重组是耗时的过程，IPv6 不允许在中间路由器进行分片与重组，这种操作只能在源与目标主机，这将大大提高了路由器转发的速度。
- **取消选项字段。** 选项字段不再是标准 IP 首部的一部分了，但它并没有消失，而是可能出现在 IPv6 首部中的「下一个首部」指出的位置上。删除该选项字段是的 IPv6 的首部成为固定长度的 `40` 字节。



## IP协议相关技术

- DNS 域名解析
- ARP 与 RARP 协议
- DHCP 动态获取 IP 地址
- NAT 网络地址转换
- ICMP 互联网控制报文协议
- IGMP 因特网组管理协议

### DNS

在域名中，**越靠右**的位置表示其层级**越高**。

举个栗子：

比如 `www.server.com`，根域是在最顶层，它的下一层就是 com 顶级域，再下面是 server.com。

域名的层级关系类似一个树状结构：

- 根 DNS 服务器
- 顶级域 DNS 服务器（com）
- 权威 DNS 服务器（server.com）

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuzockP2TeHnIgQhIfZSO6Ud5icBKUVj4KpgGibYCicicB31ia9AmtvS6WictA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

**根域**的 DNS 服务器信息**保存在**互联网中**所有的 DNS 服务器中**。这样一来，任何 DNS 服务器就都可以找到并访问根域 DNS 服务器了。

#### 域名解析的工作流程

浏览器先检查自己缓存中有否 ---> 若无向操作系统缓存要 ---> 若无检查本机域名解析文件hosts ---> 若无向DNS服务器进行查询：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuj1bqqp8P1QxT5vZcBYvCqTF6p9Zeg0xtttiaGFaK4knaA9AeoXegVOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 



### ARP与RARP

通过 **ARP（Address Resolution Protocol）协议**，求得下一跳的 MAC 地址。

ARP 是借助 **ARP 请求与 ARP 响应**两种类型的包确定 MAC 地址的。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yud8EMZpZq2H0Wt2EyLd7pohhg1vfXwqyxEjRG0Jaic8DcibTVJibI3sUnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

- 主机会通过**广播发送 ARP 请求**，这个包中包含了想要知道的 MAC 地址的主机 IP 地址。
- 当同个链路中的所有设备收到 ARP 请求时，会去拆开 ARP 请求包里的内容，如果 ARP 请求包中的目标 IP 地址与自己的 IP 地址一致，那么这个设备就将自己的 MAC 地址塞入 **ARP 响应包**返回给主机。



ARP 协议是已知 IP 地址 求 MAC 地址，那 RARP 协议正好相反。**已知 MAC 地址求 IP 地址**。例如将打印机服务器等小型嵌入式设备接入到网络时就经常会用得到。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yugZ7BBNs4MVsMDDWyx6ic4pGhpTg6RqSRbodf9hNneY4EaywmiaABX1Eg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



### DHCP

DHCP (Dynamic Host Configuration Protocol) 客户端进程监听的是 68 端口号，DHCP 服务端进程监听的是 67 端口号。

UDP广播

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuYPIh1qKGjHLr2pYtjymVffCBkfPNBTkqVrUm5jeKzQ66ibTyjwzPOKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

如果租约的 DHCP IP 地址快期后，客户端会向服务器发送 DHCP 请求报文：

- 服务器如果同意继续租用，则用 DHCP ACK 报文进行应答，客户端就会延长租期。
- 服务器如果不同意继续租用，则用 DHCP NACK 报文，客户端就要停止使用租约的 IP 地址。

**<u>*咦，用的是广播，那如果 DHCP 服务器和客户端不是在同一个局域网内，路由器又不会转发广播包，那不是每个网络都要配一个 DHCP 服务器？*</u>**

所以，为了解决这一问题，就出现了**DHCP 中继代理**。有了 DHCP 中继代理以后，**对不同网段的 IP 地址分配也可以由一个 DHCP 服务器统一进行管理。**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yuezueicuibFDLTA4HnZ1gRIcibyd6bb2ZBIeicWPOANEKNYyBX6oclnj7qg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 



### NAT、ICMP、IGMP

#### NAT网络地址转换

NAT 就是在同个公司、家庭、教室内的主机对外部通信时，把私有 IP 地址转换成公有 IP 地址。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yugY7SM2icID4T8YRDmzOSlvxvC9OzC7CociaCibuodYyibJFLrG4dFia5szA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:70%;" /> 

**<u>*那不是N 个 私有 IP 地址，你就要 N 个公有 IP 地址？这怎么就缓解了 IPv4 地址耗尽的问题？这不瞎扯吗？*</u>**

是，here comes NAPT网络地址与端口转换。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZdpI5tWwB8wMfZQ8ichJW1yu7icmfGYzhic3Ee3MgfX32iadOHK46TBlxN54ibcsnUh9yqweJ1nPKU4XFQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:60%;" /> 

两个客户端的本地端口都是 1025，**两个私有 IP 地址都转换 IP 地址为公有地址 120.229.175.121，但是以不同的端口号作为区分。**



**<u>*如何解决 NAT 潜在的问题呢？*</u>**

*第一种就是改用 IPv6*

*第二种 NAT 穿透技术*



#### ICMP互联网控制报文协议

ICMP 全称是 **Internet Control Message Protocol**，也就是**互联网控制报文协议**。

`ICMP` 主要的功能包括：**确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。**



#### IGMP

**IGMP 是因特网组管理协议，工作在主机（组播成员）和最后一跳路由之间**

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210328214426099.png" alt="image-20210328214426099" style="zoom:50%;" /> 

