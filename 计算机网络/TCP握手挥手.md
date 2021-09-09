## 认识TCP

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843ZPb6tFLvCVuXEn98khfs7y2KRvOV0ia5icVByzIK3aAKRURuVZKagsKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

**序列号**：在建立连接时由计算机生成的随机数作为其初始值，通过 SYN 包传给接收端主机，每发送一次数据，就「累加」一次该「数据字节数」的大小。**用来解决网络包乱序问题。**

**确认应答号**：指下一次「期望」收到的数据的序列号，发送端收到这个确认应答以后可以认为在这个序号以前的数据都已经被正常接收。**用来解决不丢包的问题。**

**控制位：**

- *ACK*：该位为 `1` 时，「确认应答」的字段变为有效，TCP 规定除了最初建立连接时的 `SYN` 包之外该位必须设置为 `1` 。
- *RST*：该位为 `1` 时，表示 TCP 连接中出现异常必须强制断开连接。
- *SYC*：该位为 `1` 时，表示希望建立连接，并在其「序列号」的字段进行序列号初始值的设定。
- *FIN*：该位为 `1` 时，表示今后不会再有数据发送，希望断开连接。当通信结束希望断开连接时，通信双方的主机之间就可以相互交换 `FIN` 位置为 1 的 TCP 段。



TCP 是**面向连接的、可靠的、基于字节流**的传输层通信协议。

- 面向对象：一对一；
- 可靠的：TCP 保证一个报文一定能够到达接收端；
- 字节流：消息是「没有边界」的，所以无论我们消息有多大都可以进行传输。并且消息是「有序的」，当「前一个」消息没有收到的时候，即使它先收到了后面的字节已经收到，那么也不能扔给应用层去处理，同时对「重复」的报文会自动丢弃。



<u>***什么是TCP连接？***</u>

简单来说就是，**用于保证可靠性和流量控制维护的某些状态信息，这些信息的组合，包括Socket、序列号和窗口大小称为连接。**所以建立一个 TCP 连接是需要客户端与服务器端达成上述三个信息的共识：

- **Socket**：由 IP 地址和端口号组成
- **序列号**：用来解决乱序问题等
- **窗口大小**：用来做<u>流量控制</u>



**<u>*如何唯一确定一个 TCP 连接呢？*</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843m0iaYviaJWmRyichZmR3x1gptwEsnJ6yAObqicnLkO0uNBwbYrxCoic27cA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 50%;" /> 

**源地址和目的地址**的字段（**32位**）是在 **IP 头部**中，作用是通过 IP 协议发送报文给对方主机。



**<u>*有一个 IP 的服务器监听了一个端口，它的 TCP 的最大连接数是多少？*</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843wBh1Ca3jpEqO0Xia0YzlicCgFdhLw8N4f0TCfglTwtxzecpECvmhBtEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" /> 

对 IPv4，客户端的 IP 数最多为 `2` 的 `32` 次方，客户端的端口数最多为 `2` 的 `16` 次方，也就是服务端单机最大 TCP 连接数，约为 `2` 的 `48` 次方。

当然，服务端最大并发 TCP 连接数远不能达到理论上限。

- 首先主要是**文件描述符限制**，Socket 都是文件，所以首先要通过 `ulimit` 配置文件描述符的数目；
- 另一个是**内存限制**，每个 TCP 连接都要占用一定内存，操作系统是有限的。



### TCP和UDP的区别

*1. 连接*

- TCP 是面向连接的传输层协议，<u>传输数据前先要建立连接</u>。
- UDP 是不需要连接，<u>即刻传输数据</u>。

*2. 服务对象*

- TCP 是一对一的两点服务，即一条连接只有两个端点。
- UDP 支持一对一、一对多、多对多的交互通信

*3. 可靠性*

- TCP 是可靠交付数据的，数据可以无差错、不丢失、不重复、按需到达。
- UDP 是尽最大努力交付，不保证可靠交付数据。

*4. 拥塞控制、流量控制*

- TCP <u>有拥塞控制和流量控制机制</u>，保证数据传输的安全性。
- UDP 则没有，即使<u>网络非常拥堵</u>了，也<u>不会影响 UDP 的发送速率</u>。

*5. 首部开销*

- TCP 首部长度较长，会有一定的开销，首部在没有使用「选项」字段时是 `20` 个字节，如果使用了「选项」字段则会变长的。
- UDP 首部只有 8 个字节，并且是固定不变的，开销较小。



### TCP建立连接

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210324164213287.png" alt="image-20210324164213287" style="zoom: 67%;" /> 

从上面的过程可以发现**第三次握手是可以携带数据的，前两次握手是不可以携带数据的**。



#### <u>为什么是三次握手？不是两次、四次？为什么三次握手才可以初始化Socket、序列号和窗口大小并建立 TCP 连接？</u>

“因为三次握手才能保证双方具有接收和发送的能力。” ---- 较片面

***原因一：避免历史连接***

简单来说，三次握手的**首要原因是为了防止旧的重复连接初始化造成混乱。**

<img src="https://mmbiz.qlogo.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS8436nKau10lAsztRqbyhjC1C1GRcsEz04icZmomMjwcxgeGn97BnKUoxibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2" alt="图片" style="zoom: 40%;" /> 



客户端连续发送多次 SYN 建立连接的报文，在网络拥堵等情况下：

如果是两次握手连接，就不能判断当前连接是否是历史连接，三次握手则可以在客户端（发送方）准备发送第三次报文时，客户端因有足够的上下文来判断当前连接是否是历史连接：

- 如果是历史连接（序列号过期或超时），则**第三次握手发送的报文是 `RST` 报文**，以此中止历史连接；
- 如果不是历史连接，则**第三次发送的报文是 `ACK` 报文**，通信双方就会成功建立连接；

所以， TCP 使用三次握手建立连接的最主要原因是**防止历史连接初始化了连接。**



***原因二：同步双方初始序列号***

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843HWajXhQQfx6CH4EUxLqib0AAOXolZfIvuoEDkDoXaQ3RIceibo8ia9MQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

当客户端发送携带「初始序列号」的 `SYN` 报文的时候，需要服务端回一个 `ACK` 应答报文，表示客户端的 SYN 报文已被服务端成功接收，那当服务端发送「初始序列号」给客户端的时候，依然也要得到客户端的应答回应，**这样一来一回，才能确保双方的初始序列号能被可靠的同步。**

四次握手其实也能够可靠的同步双方的初始化序号，但由于**第二步和第三步可以优化成一步**，所以就成了「三次握手」。

而两次握手只保证了一方的初始序列号能被对方成功接收，没办法保证双方的初始序列号都能被确认接收。

***原因三：避免资源浪费***

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843CaTeGEvR5jg3iaHbUTEroayMBUoK3yfy9zGwlIia8pJu8x4RDkDGFLicg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:40%;" /> 

如果只有「两次握手」，当客户端的 `SYN` 请求连接在网络中阻塞，客户端没有接收到 `ACK` 报文，就会重新发送 `SYN` ，由于没有第三次握手，<u>服务器不清楚客户端是否收到了自己发送的建立连接的 `ACK` 确认信号</u>，所以每收到一个 `SYN` 就只能先主动建立一个连接，这会**造成堵塞**。

如果客户端的 `SYN` 阻塞了，重复发送多次 `SYN` 报文，那么服务器在收到请求后就会**建立多个冗余的无效链接，造成不必要的资源浪费。**

即两次握手会造成消息滞留情况下，服务器重复接受无用的连接请求 `SYN` 报文，而造成重复分配资源。

**<u>小结</u>**

TCP 建立连接时，通过三次握手**能防止历史连接的建立，能减少双方不必要的资源开销，能帮助双方同步初始化序列号**。序列号能够保证数据包不重复、不丢弃和按序传输。

不使用「两次握手」和「四次握手」的原因：

- 「两次握手」：无法防止历史连接的建立，会造成双方资源的浪费，也无法可靠的同步双方序列号；
- 「四次握手」：三次握手就已经理论上最少可靠连接建立，所以不需要使用更多的通信次数。



<u>***为什么客户端和服务端的初始序列号 ISN 是不相同的？***</u>

因为网络中的报文**会延迟、会复制重发、也有可能丢失**，这样会造成的不同连接之间产生互相影响，所以为了避免互相影响，客户端和服务端的初始序列号是随机且不同的。



**<u>*初始序列号 ISN 是如何随机产生的？*</u>**

起始 `ISN` 是基于时钟的，每 4 毫秒 + 1，转一圈要 4.55 个小时。RFC1948 中提出了一个较好的初始化序列号 ISN 随机生成算法。



**<u>*既然 IP 层会分片，为什么 TCP 层还需要 MSS 呢？*</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS84378ic06fUySbbZgdeQ0uo6q0ulThJWexy6Bic6sJGCLVzkvXmvgjk7B2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

- `MTU (Maximum Transfer Unit)`：一个网络包的最大长度，以太网中一般为 `1500` 字节；
- `MSS (Maximum Segment Size)`：除去 IP 和 TCP 头部之后，一个网络包所能容纳的 TCP 数据的最大长度；

当 IP 层有一个超过 `MTU` 大小的数据（TCP 头部 + TCP 数据）要发送，那么 IP 层就要进行分片，把数据分片成若干片，保证每一个分片都小于 MTU。把一份 IP 数据报进行分片以后，由目标主机的 IP 层来进行重新组装后，在交给上一层 TCP 传输层。

**如果一个 IP 分片丢失，整个 IP 报文的所有分片都得重传**。因为 IP 层本身**没有超时重传机制**，它由传输层的 TCP 来负责超时和重传。

当接收方发现 TCP 报文（头部 + 数据）的某一片丢失后，则不会响应 ACK 给对方，那么发送方的 TCP 在超时后，就会**重发「整个 TCP 报文（头部 + 数据）」**。因此，可以得知由 IP 层进行分片传输，是非常没有效率的。

所以，为了达到最佳的传输效能 TCP 协议在**建立连接的时候通常要协商双方的 MSS 值**，当 TCP 层发现数据超过 MSS 时，则就先会进行分片，当然由它形成的 IP 包的长度也就不会大于 MTU ，自然也就不用 IP 分片了。

经过 TCP 层分片后，**如果一个 TCP 分片丢失后，进行重发时也是以 MSS 为单位，**而不用重传所有的分片，大大增加了重传的效率。



### SYN攻击

攻击者短时间伪造不同 IP 地址的 `SYN` 报文，服务端每接收到一个 `SYN` 报文，就进入`SYN_RCVD` 状态，<u>但服务端发送出去的 `ACK + SYN` 报文，无法得到未知 IP 主机的 `ACK` 应答</u>，久而久之就会**占满服务端的 SYN 接收队列（未连接队列）**，使得服务器不能为正常用户服务。

避免SYN攻击方式一

其中一种解决方式是通过修改 Linux 内核参数，控制队列大小和当队列满时应做什么处理。

避免SYN攻击方式二

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843J4ygolhCqYoh52C8EFia9QUGsEY52oGTNicruORWBg0MD7TTeIiajh0rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

- 当 「 SYN 队列」满之后，后续服务器收到 SYN 包，不进入「 SYN 队列」；
- 计算出一个 `cookie` 值，再以 SYN + ACK 中的「序列号」返回客户端，
- 服务端接收到客户端的应答报文时，服务器会检查这个 ACK 包的合法性。如果合法，直接放入到「 Accept 队列」。
- 最后应用通过调用 `accpet()` socket 接口，从「 Accept 队列」取出的连接。



### TCP断开连接

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843KaMMu2mHfFLZNgiaREDZ5JicRYrlaiciayQjh9HDsacxIbMT0emGUpAX5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" /> 

<u>***为什么挥手四次？***</u>

- 关闭连接时，客户端向服务端发送 `FIN` 时，仅仅表示客户端不再发送数据了但是还能接收数据。
- 服务器收到客户端的 `FIN` 报文时，先回一个 `ACK` 应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送 `FIN` 报文给客户端来表示同意现在关闭连接。

从上面过程可知，服务端通常需要等待完成数据的发送和处理，所以服务端的 `ACK` 和 `FIN` 一般都会分开发送，从而比三次握手导致多了一次。

**<u>*为什么 TIME_WAIT 等待的时间是 2MSL？*</u>**

MSL 与 TTL 的区别：MSL 的单位是时间，而 TTL 是经过路由跳数。所以 **MSL 应该要大于等于 TTL 消耗为 0 的时间**，以确保报文已被自然消亡。网络中可能存在来自发送方的数据包，当这些发送方的数据包被接收方处理后又会向对方发送响应，所以**一来一回需要等待 2 倍的时间**。



**<u>*为什么需要 TIME_WAIT 状态？*</u>**

需要 TIME-WAIT 状态，主要是两个原因：

- 防止具有相同「四元组」的「旧」数据包被收到；
- 保证「被动关闭连接」的一方能被正确的关闭，即保证最后的 ACK 能让被动关闭方接收，从而帮助其正常关闭



**<u>*如果已经建立了连接，但是客户端突然出现故障了怎么办？*</u>**

TCP 有一个机制是**保活机制**。这个机制的原理是这样的：

定义一个时间段，在这个时间段内，如果没有任何连接相关的活动，TCP 保活机制会开始作用，每隔一个时间间隔，发送一个探测报文，该探测报文包含的数据非常少，如果连续几个探测报文都没有得到响应，则认为当前的 TCP 连接已经死亡，系统内核将错误信息通知给上层应用程序。



**<u>*针对 TCP 应该如何 Socket 编程？*</u>**

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS8436ACEChhUgLWdrnLISt2gyF5LrC5D3yrnQHtgGQHj4iaTwUBqz4VEW5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 67%;" /> 

<img src="C:\Users\lonaeyeo\AppData\Roaming\Typora\typora-user-images\image-20210325170946008.png" alt="image-20210325170946008" style="zoom:67%;" /> 

这里需要注意的是，服务端调用 `accept` 时，连接成功了会返回一个已完成连接的 socket，后续用来传输数据。所以，监听的 socket 和真正用来传送数据的 socket，是「两个」 socket，一个叫作**监听 socket**，一个叫作**已完成连接 socket**。成功连接建立之后，双方开始通过 read 和 write 函数来读写数据，就像往一个文件流里面写东西一样。



**<u>*listen 时候参数 backlog 的意义？*</u>**

Linux内核中会维护两个队列：

- 未完成连接队列（SYN 队列）：接收到一个 SYN 建立连接请求，处于 SYN_RCVD 状态；
- 已完成连接队列（Accpet 队列）：已完成 TCP 三次握手过程，处于 ESTABLISHED 状态；

