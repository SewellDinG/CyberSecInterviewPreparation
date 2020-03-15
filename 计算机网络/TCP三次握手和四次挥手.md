## TCP 三次握手和四次挥手？

### 三次握手

客户端, 服务端 := A, B

- 第一次握手：建立连接，A发送连接请求报文段，将SYN位置为1，Sequence Number为x；然后，A进入SYN_SEND状态，等待B的确认；
- 第二次握手：B收到A的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1)；同时，还要发送SYN请求信息，将SYN位置为1，Sequence Number为y；B将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给A，此时B进入SYN_RECV状态；
- 第三次握手：A收到B的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向B发送ACK报文段，这个报文段发送完毕以后，A和B都进入ESTABLISHED状态，完成TCP三次握手。

数据传输：

- TCP连接建立后，浏览器就可以利用HTTP／HTTPS协议向服务器发送请求进行数据传输。

![TCP](https://github.com/SewellDinG/CyberSecInterviewPreparation/blob/master/计算机网络/images/TCP.jpg)

### 四次分手

A, B := 可以是客户端，也可以是服务端

- 第一次分手：A设置Sequence Number和Acknowledgment Number，向B发送一个FIN报文段；此时，A进入FIN_WAIT_1状态；这表示A没有数据要发送给B了；
- 第二次分手：B收到了A发送的FIN报文段，向A回一个ACK报文段，Acknowledgment Number为Sequence Number加1；A进入FIN_WAIT_2状态；B告诉A，我“同意”你的关闭请求；
- 第三次分手：B向A发送FIN报文段，请求关闭连接，同时B进入LAST_ACK状态；
- 第四次分手：A收到B发送的FIN报文段，向B发送ACK报文段，然后A进入TIME_WAIT状态；B收到A的ACK报文段以后，就关闭连接；此时，A等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，A也可以关闭连接了。

### 为什么不是两次或者四次？

- TCP作为一种可靠传输控制协议，其**核心思想：既要保证数据可靠传输，又要提高传输的效率，而用三次恰恰可以满足以上两方面的需求！**
- 四次握手的过程：很显然2和3这两个步骤可以合并，只需要三次握手，可以提高连接的速度与效率。
  - A发送同步信号SYN + A's Initial sequence number；
  - B确认收到A的同步信号，并记录A's ISN到本地，命名B's ACK sequence number；
  - B发送同步信号SYN + B's Initial sequence number；
  - A确认收到B的同步信号，并记录B's ISN到本地，命名A's ACK sequence number。
- 二次握手的过程：A与B就A的初始序列号达成一致，但B无法知道A是否已经接受到自己的同步信号，如果这个同步信号丢失，A和B就B的初始序列号将无法达成一致。
  - A发送同步信号SYN + A's Initial sequence number；
  - B发送同步信号SYN + B's Initial sequence number + B's ACK sequence number。

### TCP可靠传输的精髓

- TCP连接的一方A，由操作系统动态随机选取一个32位长的序列号(Initial Sequence Number)，假设A的初始序列号为1000，以该序列号为原点，对自己将要发送的每个字节的数据进行编号，1001，1002，1003..... 并把自己的初始序列号ISN告诉B，让B有个思想准备，什么样编号的数据是合法的，什么编号是非法的，比如编号900就是非法的，同时B还可以对A每一个编号的字节数据进行确认。如果A收到B确认编号为2001,则意味着字节编号为1001-2000，共1000个字节已经安全到达。
- 同理B也是类似的操作，假设B的初始序列号ISN为2000，以该序列号为原点，对自己将要发送的每个字节的数据进行编号，2001，2002，2003..... 并把自己的初始序列号ISN告诉A，以便A可以确认B发送的每一个字节。如果B收到A确认编号为4001,则意味着字节编号为2001-4000，共2000个字节已经安全到达。
- 一句话概括，TCP连接握手，握的是啥？**通信双方数据原点的序列号！**