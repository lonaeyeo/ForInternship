## 概括

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgTfasBzdn2sIB39aFcqL22zhAa7v9d9vR1oZF4mibLUKouDEfKjYoZww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:65%;" /> 

TCP 是通过序列号、确认应答、重发控制、连接管理以及窗口控制等机制实现可靠性传输的。



## 重传机制

TCP实现可靠传输的方式之一，是通过**序列号与确认应答**。

接下来说说常见的重传机制：

- 超时重传
- 快速重传
- SACK
- D-SACK

### 超时重传

发送数据时，设定一个定时器，当超过指定的时间后，没有收到对方的 `ACK` 确认应答报文，就会重发该数据，也就是我们常说的**超时重传**。

两种情况：

- 数据包丢失（数据包发出去了，服务器没收到，根本不会发出ACK）
- 确认应答丢失（数据包发出去了，服务器收到了，ACK发出，但客户端没有收到）

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgObdlvOJianvD0oj586rMc8wVs4hlzUtgRibWfD0WBpAJhRtHxOPd9ibibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 60%;" /> 

*<u>**超时时间应该设置为多少呢？**</u>*

RTT(Round-Trip Time 往返时延)

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhg0gQic4pOyb48icGE2bMfSup2icU0ZAb1SWsEyCibHND1x7V3icliaWeuxwQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

超时重传时间是以 `RTO` （Retransmission Timeout 超时重传时间）表示。

**<u>*假设在重传的情况下，超时时间 `RTO` 「较长或较短」时，会发生什么事情呢？*</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgyx80wY6Zsy2xW0U5egNib7icspUPsE9PKYq5WLaoJGpkTIoZ1c5gIjCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

**超时重传时间 RTO 的值应该略大于报文往返  RTT 的值**。

**<u>*我们来看看 Linux 是如何计算 `RTO` 的呢？*</u>**

估计往返时间，通常需要采样以下两个：

- 需要 TCP 通过采样 RTT 的时间，然后进行**加权平均**，算出**一个平滑 RTT** 的值，而且这个**值还是要不断变化**的，因为网络状况不断地变化。
- 除了采样 RTT，**还要采样 RTT 的波动范围**，这样就避免如果 RTT 有一个大的波动的话，很难被发现的情况。

如果超时重发的数据，再次超时的时候，又需要重传的时候，TCP 的策略是**超时间隔加倍。**

也就是**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送。**

超时触发重传存在的问题是，<u>超时周期可能相对较长。那是不是可以有更快的方式呢？</u>

### 快速重传

**快速重传（Fast Retransmit）机制**，它**不以时间为驱动，而是以数据驱动重传**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhggRhxrfiaRIsia0z3T4l7TxVM7J9vjFktFRKHkdva953UY6vFIpsYsOicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

- **发送端收到了三个 Ack = 2 的确认，知道了 Seq2 还没有收到，就会在定时器过期之前，重传丢失的 Seq2。**

所以，快速重传的工作方式是**当收到三个相同的 ACK 报文时，会在定时器过期之前，重传丢失的报文段。**

快速重传机制只解决了一个问题，就是超时时间的问题，但是它依然面临着另外一个问题。就是**重传的时候，是重传之前的一个，还是重传所有的问题。**

比如对于上面的例子，是重传 Seq2 呢？还是重传 Seq2、Seq3、Seq4、Seq5 呢？因为发送端并不清楚这连续的三个 Ack 2 是谁传回来的。为了解决不知道该重传哪些 TCP 报文，于是就有 `SACK` 方法。



### SACK方法

**SACK（ Selective Acknowledgment ）：选择性确认。**

这种方式需要在 TCP 头部「选项」字段里加一个 `SACK` 的东西，它**可以将缓存的地图发送给发送方**，这样发送方就可以知道哪些数据收到了，哪些数据没收到，知道了这些信息，就可以**只重传丢失的数据**。

如下图，发送方收到了三次同样的 ACK 确认报文，于是就会触发快速重发机制，通过 `SACK` 信息发现只有 `200~299` 这段数据丢失，则重发时，就只选择了这个 TCP 段进行重复。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhg3RbEZItSOQY1TadJpUTuibIDziaibALaEmk7JO0Ill7iaeiaGI94Wyia0NXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

ACK:        200,         200,         200

SACK: 300-400, 300-500, 300-600

如果要支持 `SACK`，必须双方都要支持。在 Linux 下，可以通过 `net.ipv4.tcp_sack` 参数打开这个功能（Linux 2.4 后默认打开）。

### Duplicate SACK 

Duplicate SACK 又称 `D-SACK`，其主要**使用了 SACK 来告诉「发送方」有哪些数据被重复接收了。**

*栗子一号：ACK 丢包*

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgYe99dD53qcV16GAJGCeaw1fuqhvhBC1mVs4kETqHlP3flTByXia9ILQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

- 「接收方」发给「发送方」的两个 ACK 确认应答都丢失了，所以发送方超时后，重传第一个数据包（3000 ~ 3499）
- **于是「接收方」发现数据是重复收到的，于是回了一个 SACK = 3000~3500**，告诉「发送方」 3000~3500 的数据早已被接收了，因为 ACK 都到了 4000 了，已经意味着 4000 之前的所有数据都已收到，所以这个 SACK 就代表着 `D-SACK`。
- 这样「发送方」就知道了，数据没有丢，是「接收方」的 ACK 确认报文丢了。

*栗子二号：网络延时*

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgGSKaMMbmPiaUQmCvR4cz2kQ5OV8SqaPIwdkZ7T19uEs5WxCc6h66x7w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

**所以「接收方」回了一个 SACK=1000~1500，因为 ACK 已经到了 3000，所以这个 SACK 是 D-SACK，表示收到了重复的包。**

可见，`D-SACK` 有这么几个好处：

1. 可以让「发送方」知道，是**发出去的包丢了**，还是**接收方回应的 ACK 包丢了**;
2. 可以知道是不是「发送方」的数据包被网络延迟了;
3. 可以知道网络中是不是把「发送方」的数据包给复制了;

在 Linux 下可以通过 `net.ipv4.tcp_dsack` 参数开启/关闭这个功能（Linux 2.4 后默认打开）。



## 滑动窗口

窗口大小就是指**无需等待确认应答，而可以继续发送数据的<u>最大值</u>**。

假设窗口大小为 `3` 个 TCP 段，那么发送方就可以「连续发送」 `3` 个 TCP 段，并且中途若有 ACK 丢失，**可以通过「下一个确认应答进行确认」**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgz9LqxXsGESpzbick5b29No4rkqEubmVxexM2pM5DASr53mk65Qx5icZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 67%;" /> 

图中的 **<u>ACK 600 确认应答报文丢失</u>**，也没关系，因为可以通话下一个确认应答进行确认，只要发送方收到了 ACK 700 确认应答，就意味着 700 之前的所有数据「接收方」都收到了。这个模式就叫**累计确认**或者**累计应答**。

**<u>*窗口大小由哪一方决定？*</u>**

TCP 头里有一个字段叫 `Window`，也就是窗口大小。

**这个字段是接收端告诉发送端自己还有多少缓冲区可以接收数据。于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来。**

所以，通常窗口的大小是由接收方的决定的。

**<u>发送方的窗口</u>**

举个栗子：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgesUOXicVCTx6j4WPOa8heRQc3aPPwWDAIMnUJocXjmABL8JSjnkyenw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:60%;" /> 



**<u>*程序是如何表示发送方的四个部分的呢？*</u>**

TCP 滑动窗口方案使用三个指针来跟踪在四个传输类别中的每一个类别中的字节。其中两个指针是绝对指针（指特定的序列号），一个是相对指针（需要做偏移）。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhge0ujfyrkBBMicd4U4qiaubDemmA9BhjDalRicd1cxrAkBQqlBQSL1oGzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:60%;" /> 

- `SND.WND`：表示发送窗口的大小（大小是由接收方指定的）；
- `SND.UNA`：指向的是已发送但未收到ACK确认的第一个字节的序列号；
- `SND.NXT`：指向未发送但可发送范围的第一个字节的序列号。

**<u>接收方的窗口</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhglnibEWRdZcD4QWuC61y9iaRoKlWUWyicnP0iaS01fAjsdGgKJnYJcfm0wA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:60%;" /> 

- `RCV.WND`：表示接收窗口的大小，它会通告给发送方。
- `RCV.NXT`：是一个指针，它指向期望从发送方发送来的下一个数据字节的序列号，也就是 #3 的第一个字节。

<u>***接收窗口和发送窗口的大小是相等的吗？***</u>

并不是，大致相等。



## 流量控制

如果一直无脑的发数据给对方，但对方处理不过来，那么就会导致触发重发机制，从而导致网络流量的无端的浪费。

为了解决这种现象发生，**TCP 提供一种机制可以让「发送方」根据「接收方」的实际接收能力控制发送的数据量，这就是所谓的流量控制。**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgrEtDiblwiaNET0Dx9BMfqyVXY6Kc00ib4BD6l1yyPEZrb2ptxV31VSmFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 60%;" /> 



#### 操作系统缓冲区与滑动窗口的关系

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgEnFtyFXUGUib92pibiacHuqQDAhbnC4AcVuMiaLs50TjjW0nbyrSnLs9Rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

可见最后窗口都收缩为 0 了，也就是发生了窗口关闭。当发送方可用窗口变为 0 时，发送方实际上会定时发送窗口探测报文，以便知道接收方的窗口是否发生了改变。

第二个栗子：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgoTmpebyoD5Qa0cbibkicaGibzdicpo4jibwkLR0L77fyrXyelL5nkZ40ibBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

实际上，发送窗口和接收窗口中所存放的字节数，都是放在操作系统内存缓冲区中的，而操作系统的缓冲区，会**被操作系统调整**。当应用进程没办法及时读取缓冲区的内容时，也会对我们的缓冲区造成影响。



**<u>*那操心系统的缓冲区，是如何影响发送窗口和接收窗口的呢？*</u>**

**为了防止这种情况发生，TCP 规定是不允许同时减少缓存又收缩窗口的，而是采用先收缩窗口，过段时间在减少缓存，这样就可以避免了丢包情况。**



### 窗口关闭

**如果窗口大小为 0 时，就会阻止发送方给接收方传递数据，直到窗口变为非 0 为止，这就是窗口关闭。**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgI1zQk2E30AjqibOmvbWiaNswZPhx3UcyYTyqgJTCoEJeuMEoic4t3KuOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

<u>***TCP 是如何解决窗口关闭时，潜在的死锁现象呢？***</u>

TCP 为每个连接设有一个持续定时器，**只要 TCP 连接一方收到对方的零窗口通知，就启动持续计时器。**

### 糊涂窗口综合征

如果接收方太忙了，来不及取走接收窗口里的数据，那么就会导致发送方的发送窗口越来越小。





## 阻塞控制

前面的流量控制是避免「发送方」的数据填满「接收方」的缓存，但是并不知道网络的中发生了什么。

**在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，这个情况就会进入恶性循环被不断地放大….**于是，就有了**拥塞控制**，控制的目的就是**避免「发送方」的数据填满整个网络。**



<u>***什么是拥塞窗口？和发送窗口有什么关系呢？***</u>

**拥塞窗口 cwnd**是发送方维护的一个 的状态变量，它会根据<u>**网络的拥塞程度动态变化的**</u>。发送窗口的值是**<u>swnd = min(cwnd, rwnd)</u>**，也就是拥塞窗口和接收窗口中的最小值。

拥塞窗口 `cwnd` 变化的规则：

- 只要网络中没有出现拥塞，`cwnd` 就会增大；
- 但网络中出现了拥塞，`cwnd` 就减少；



**<u>*那么怎么知道当前网络是否出现了拥塞呢？*</u>**

其实只要「发送方」没有在规定时间内接收到 ACK 应答报文，也就是**发生了超时重传，就会认为网络出现了用拥塞。**



拥塞控制主要是四个算法：

- **慢启动**
- **拥塞避免**
- **拥塞发生**
- **快速恢复**



### 慢启动

慢启动的算法记住一个规则就行：**当发送方每收到一个 ACK，就拥塞窗口 cwnd 的大小就会加 1。**慢启动算法，发包的个数是**指数性的增长**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgmialsBqYwdlLHfaykYgfgeZ3UM9bgvqoaAz07CRG8RI1Nken6JcQaqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:40%;" /> 

<u>*那慢启动涨到什么时候是个头呢？*</u>

有一个叫慢启动门限  `ssthresh` （slow start threshold）状态变量。

- 当 `cwnd < ssthresh` 时，使用慢启动算法。
- 当 `cwnd >= ssthresh` 时，就会使用「拥塞避免算法」。



### 拥塞避免算法

当拥塞窗口 `cwnd` 「超过」慢启动门限 `ssthresh` 就会进入拥塞避免算法。一般来说 `ssthresh` 的大小是 `65535` 字节。

那么进入拥塞避免算法后，它的规则是：**每当收到一个 ACK 时，cwnd 增加 1/cwnd。**

现假定 `ssthresh` 为 `8`：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhg6VqpU6rrp4BElebYKEibibWZ9AOFygwYD98vk7AKg4ykrU6qT58icpjXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:45%;" /> 

拥塞避免算法就是将原本慢启动算法的指数增长变成了线性增长，网络就会慢慢进入了拥塞的状况了，于是就会出现丢包现象，这时就需要对丢失的数据包进行重传。

**当触发了重传机制**，也就进入了「拥塞发生算法」。



### 拥塞发生

首先当网络出现拥塞，也就是会发生数据包重传，重传机制主要有两种：

- <u>超时重传</u>
- <u>快速重传</u>



发生<u>超时重传</u>的拥塞发生算法：

- `ssthresh` 设为 `cwnd/2`，
- `cwnd` 重置为 `1`



<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgDicBugvroe9EtiaFU38hk4JuVfDciauVPfecBNp8TPI1zkoqbibePA4dlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:45%;" /> 

重新开始慢启动，慢启动是会突然减少数据流的。这真是一旦「超时重传」，马上回到解放前。但是这种方式太激进了，反应也很强烈，会造成网络卡顿。



发生<u>快速重传</u>的拥塞发生算法：

当接收方发现丢了一个中间包的时候，发送三次前一个包的 ACK，于是发送端就会快速地重传，不必等待超时再重传。

- `cwnd = cwnd/2` ，也就是设置为原来的一半;
- `ssthresh = cwnd`;
- 进入**快速恢复算法**。



### 快速恢复

快速重传和快速恢复算法一般同时使用。

然后，进入快速恢复算法如下：

- 拥塞窗口 `cwnd = ssthresh + 3` （ 3 的意思是确认有 3 个数据包被收到了）；
- 重传丢失的数据包；
- 如果再收到重复的 ACK，那么 cwnd 增加 1；
- 如果收到新数据的 ACK 后，设置 cwnd 为 ssthresh，接着就进入了拥塞避免算法。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgR3w50EdpWF95ZM6QPpELCF3P1niazia8nBrSQUvX7e7F7LXMiaXR3iayUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

