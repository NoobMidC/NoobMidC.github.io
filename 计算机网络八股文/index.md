# 计算机网络八股文


<!--more-->

# 计算机网络

## 为什么要对网络协议分层？

​	网络协议是计算机在通信过程中要遵循的一些约定好的规则。

​	网络分层的原因：

- 易于实现和维护，因为各层之间是独立的，层与层之间不会收到影响。

- 有利于标准化的制定



## 协议模型

OSI七层协议模型包括: 应用-表示-会话-传输-网络-数据链路-物理层

五层协议模型包括: 应用-传输-网络-数据链路-物理层

- 物理层

  ​		主要解决两台物理机之间的通信，通过二进制比特流的传输来实现，二进制数据表现为电流电压上的强弱，到达目的地再转化为二进制机器码。网卡、集线器工作在这一层。

  

- 数据链路层

  ​		在不可靠的物理介质上提供可靠的传输，接收来自物理层的位流形式的数据，并封装成帧，传送到上一层;同样，也将来自上层的数据帧，拆装为位流形式的数据转发到物理层。这一层在物理层提供的比特流的基础上，通过**差错控制**、**流量控制**方法，使有差错的物理线路变为无差错的数据链路。提供物理地址寻址功能。交换机工作在这一层。

  

- 网络层

  ​		将网络地址翻译成对应的物理地址，并决定如何将数据从发送方路由到接收方，通过路由选择算法为分组通过通信子网选择最佳路径。路由器工作在这一层。常见的协议有IP协议，ARP协议、ICMP协议。

  

- 传输层

  ​		传输层提供了进程间的逻辑通信，传输层向高层用户屏蔽了下面网络层的核心细节，使应用程序看起来像是在两个传输层实体之间有一条端到端的逻辑通信信道。主要协议为TCP、UDP协议

  

- 会话层

  1. 建立会话:身份验证，权限鉴定等; 
  2. 保持会话:对该会话进行维护，在会话维持期间两者可以随时使用这条会话传输数据;
  3.  断开会话:当应用程序或应用层规定的超时时间到期后，OSI会话层才会释放这条会话。

  

- 表示层

  ​		对数据格式进行编译，对收到或发出的数据根据应用层的特征进行处理，如处理为文字、图片、音频、视频、文档等，还可以对压缩文件进行解压缩、对加密文件进行解密等。

  

-  应用层

  ​		应用层的任务是通过应用进程之间的交互来完成特定的网络作用，常见的应用层协议有DNS，HTTP、FTP协议等。



## [应用层]HTTP协议

### HTTP 状态码

| 类别 |       描述       |
| :--- | :--------------: |
| 1xx  |   信息性状态码   |
| 2xx  |    成功状态码    |
| 3xx  |   重定向状态码   |
| 4xx  | 客户端错误状态码 |
| 5xx  | 服务端错误状态码 |

- 1xx

  100 Continue：表示正常，客户端可以继续发送请求

  101 Switching Protocols：切换协议，服务器根据客户端的请求切换协议。

- 2xx

  200 OK：请求成功

  201 Created：已创建，表示成功请求并创建了新的资源

  202 Accepted：已接受，已接受请求，但未处理完成。

  204 No Content：无内容，服务器成功处理，但未返回内容。

  205 Reset Content：重置内容，服务器处理成功，客户端应重置文档视图。

  206 Partial Content：表示客户端进行了范围请求，响应报文应包含Content-Range指定范围的实体内容

- 3xx

  301 Moved Permanently：永久性重定向

  302 Found：临时重定向

  303 See Other：和301功能类似，但要求客户端采用get方法获取资源

  304 Not Modified：所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。

  305 Use Proxy：所请求的资源必须通过代理访问

  307 Temporary Redirect： 临时重定向，与302类似，要求使用get请求重定向。

- 4xx

  400 Bad Request：客户端请求的语法错误，服务器无法理解。

  401 Unauthorized：表示发送的请求需要有认证信息。

  403 Forbidden：服务器理解用户的请求，但是拒绝执行该请求

  404 Not Found：服务器无法根据客户端的请求找到资源。

  405 Method Not Allowed：客户端请求中的方法被禁止

  406 Not Acceptable：服务器无法根据客户端请求的内容特性完成请求

  408 Request Time-out：服务器等待客户端发送的请求时间过长，超时

- 5xx

  500 Internal Server Error：服务器内部错误，无法完成请求

  501 Not Implemented：服务器不支持请求的功能，无法完成请求





### HTTP版本

- HTTP1.0

  ​		HTTP1.0 使用的是非持久连接，主要缺点是**客户端必须为每一个待请求的对象建立并维护一个新的连接**，即每请求一个文档就要有两倍RTT 的开销。因为同一个页面可能存在多个对象，所以非持久连接可能使一个页面的下载变得十分缓慢，而且这种 短连接增加了网络传输的负担。

  > RTT(Round Trip Time)：一个连接的往返时间，即数据发送时刻到接收到确认的时刻的差值；

- HTTP1.1

  1. 支持长连接。
  2. 在HTTP1.0的基础上引入了更多的缓存控制策略。
  3. 引入了请求范围设置，优化了带宽。
  4. 在错误通知管理中新增了错误状态响应码。
  5. 增加了Host头处理，可以传递主机名（hostname）

> http keep-alive
>
> - Connection: Keep-Alive
> - Http1.0需要显示添加
> - http1.1以及之后的http连接都是默认有此选项的。
>
> 优势 
>
> - 减少了tcp连接的数量
> - 运行请求和应答的流水线(数据流)
> - 报告错误无需关闭tcp连接
>
> 劣势 
>
> - 多于不断被不同客户端请求的资源
> - 可能会占用不必要的资源

- HTTP1.X优化（SPDY）

  ​		SPDY 并不是新的一种协议，而是在 HTTP 之前做了一层会话层。为了达到减少页面加载时间的目标，SPDY 引入了一个新的二进制分帧数据层，以实现优先次序、最小化及消除不必要的网络延迟，目的是更有效地利用底层 TCP 连接。

  1. 多路复用，为多路复用设立了请求优先级。
  2. 对header部分进行了压缩。
  3. 引入了HTTPS加密传输。
  4. 客户端可以在缓存中取到之前请求的内容。

- HTTP2.0（SPDY的升级版）

  1. HTTP2.0支持明文传输，而HTTP 1.X强制使用SSL/TLS加密传输。
  2. 和HTTP 1.x使用的header**压缩方法**不同。
  3. HTTP2.0 基于二进制格式进行解析，而HTTP 1.x基于文本格式进行解析。增加二进程的传输方式，相对于文本传输更加安全.
  4. **多路复用**，HTTP1.1是多个请求串行化单线程处理，HTTP 2.0是并行执行，一个请求超时并不会影响其他请求。
  5. 请求划分优先级

- HTTP 3.0 (QUIC)

  QUIC (Quick UDP Internet Connections), 快速 UDP 互联网连接。QUIC是基于UDP协议的。两个主要特性：

  1. 线头阻塞(HOL)问题的解决更为彻底：

     ​		基于TCP的HTTP/2，尽管从逻辑上来说，不同的流之间相互独立，不会相互影响，但在实际传输方面，数据还是要一帧一帧的发送和接收，一旦某一个流的数据有丢包，则同样会阻塞在它之后传输的流数据传输。而基于UDP的QUIC协议则可以更为彻底地解决这样的问题，让不同的流之间真正的实现相互独立传输，互不干扰。

  2. 切换网络时的连接保持

     ​		当前移动端的应用环境，用户的网络可能会经常切换，比如从办公室或家里出门，WiFi断开，网络切换为3G或4G。基于TCP的协议，由于切换网络之后，IP会改变，因而之前的连接不可能继续保持。而基于UDP的QUIC协议，则可以内建与TCP中不同的连接标识方法，从而在网络完成切换之后，恢复之前与服务器的连接。

### HTTP方法

| 方法    |                          作用                           |
| ------- | :-----------------------------------------------------: |
| GET     |                        获取资源                         |
| POST    |                      传输实体主体                       |
| PUT     |                        上传文件                         |
| DELETE  |                        删除文件                         |
| HEAD    | 和GET方法类似，但只返回报文首部，不返回报文实体主体部分 |
| PATCH   |                   对资源进行部分修改                    |
| OPTIONS |                 查询指定的URL支持的方法                 |
| CONNECT |                 要求用隧道协议连接代理                  |
| TRACE   |             服务器会将通信路径返回给客户端              |

为了方便记忆，可以将PUT、DELETE、POST、GET理解为客户端对服务端的增删改查。

- PUT：上传文件，向服务器添加数据，可以看作增
- DELETE：删除文件
- POST：传输数据，向服务器提交数据，对服务器数据进行更新。
- GET：获取资源，查询服务器资源

#### GET和POST的区别

- 作用GET用于获取资源，POST用于传输实体主体
- 参数位置GET的参数放在URL中，POST的参数存储在实体主体中，并且GET方法提交的请求的URL中的数据做多是2048字节，POST请求没有大小限制。
- 安全性GET方法因为参数放在URL中，安全性相对于POST较差一些
- 幂等性GET方法是具有幂等性的，而POST方法不具有幂等性。这里幂等性指客户端连续发出多次请求，收到的结果都是一样的.

### HTTP和HTTPS的区别

|              |       HTTP        |                  HTTPS                  |
| ------------ | :---------------: | :-------------------------------------: |
| 端口         |        80         |                   443                   |
| 安全性       |      无加密       |         有加密机制、安全性较高          |
| 资源消耗     |       较少        |       由于加密处理，资源消耗更多        |
| 是否需要证书 |      不需要       |                  需要                   |
| 协议         | 运行在TCP协议之上 | 运行在SSL协议之上，SSL运行在TCP协议之上 |



### HTTPS的加密过程

- HTTPS使用的是对称加密和非对称加密的混合加密算法。具体做法就是使用非对称加密来传输对称密钥来保证安全性，使用对称加密来保证通信的效率。
- 简化的工作流程：服务端生成一对非对称密钥，将公钥发给客户端。客户端生成对称密钥，用服务端发来的公钥进行加密，加密后发给服务端。服务端收到后用私钥进行解密，得到客户端发送的对称密钥。通信双方就可以通过对称密钥进行高效地通信了。

HTTPS的详细加密过程：

1. 客户端向服务端发起第一次握手请求，告诉服务端客户端所支持的SSL的指定版本、加密算法及密钥长度等信息。
2. 服务端将自己的公钥发给数字证书认证机构，数字证书认证机构利用自己的私钥对服务器的公钥进行数字签名，并给服务器颁发公钥证书。
3. 服务端将证书发给客服端。
4. 客服端利用数字认证机构的公钥，向数字证书认证机构验证公钥证书上的数字签名，确认服务器公开密钥的真实性。
5. 客服端使用服务端的公钥加密自己生成的对称密钥，发给服务端。
6. 服务端收到后利用私钥解密信息，获得客户端发来的对称密钥。
7. 通信双方可用对称密钥来加密解密信息。![截屏2022-08-28 下午9.27.18](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%889.27.18.png)





## [应用层]DNS协议

DNS : 域名系统；DNS系统采用的是分布式的层次数据数据库模式，还有缓存的机制。

​	工作流程

1. 主机向本地域名服务器的查询一般是采用递归查询，而本地域名服务器向根域名的查询一般是采用迭代查询
   1.  递归查询，是服务器向上级服务器发送请求报文，返回给你ip地址
   2. 迭代查询，是服务器通知你，需要向哪个上级服务器发请求报文，请求ip.
2. 实例:
   - 在浏览器中输入百度域名，操作系统会先检查自己本地的hosts文件是否有这个域名的映射关系，如果有，就先调用这个IP地址映射，完成域名解析。
   - 如果hosts文件中没有，则查询本地DNS解析器缓存，如果有，则完成地址解析。
   - 如果本地DNS解析器缓存中没有，则去查找本地DNS服务器，如果查到，完成解析。
   - 如果没有，则本地服务器会向根域名服务器发起查询请求。根域名服务器会告诉本地域名服务器去查询哪个顶级域名服务器。
   - 本地域名服务器向顶级域名服务器发起查询请求，顶级域名服务器会告诉本地域名服务器去查找哪个权限域名服务器。
   - 本地域名服务器向权限域名服务器发起查询请求，权限域名服务器告诉本地域名服务器百度所对应的IP地址。
   - 本地域名服务器告诉主机百度所对应的IP地址。



## [传输层]TCP/UDP协议详解

### TCP

#### 数据结构

​		TCP头部: 前20个字节是固定的，后面有4n个字节是根据需而增加的选项，所以TCP首部最小长度为20字节。![截屏2022-08-28 下午2.19.14](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%882.19.14.png)

- 序号：seq，占32位，用来标识从发送端到接收端发送的字节流。(一般来说在0 - 2^32-1之间)

- 确认号：ack，占32位，只有ACK标志位为1时，确认序号字段才有效，ack=seq+1。

- 标志位：

  1. SYN：发起一个新连接。

  2. FIN：释放一个连接。

  3. ACK：确认序号有效。

#### TCP可靠传输

> 主要有校验和、序列号、超时重传、流量控制及拥塞避免等几种方法。

- 校验和: 	在发送端和接收端分别计算数据的校验和，如果两者不一致，则说明数据在传输过程中出现了差错，TCP将丢弃和不确认此报文段。

- 序列号:     TCP会对每一个发送的字节进行编号，接收方接到数据后，会对发送方发送确认应答(ACK报文)，并且这个ACK报文中带有相应的确认编号，告诉发送方，下一次发送的数据从编号多少开始发。如果发送方发送相同的数据，接收端也可以通过序列号判断出，直接将数据丢弃. 

  ![截屏2022-08-28 下午2.28.48](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%882.28.48.png)

- 超时重传:    在上面说了序列号的作用，但如果发送方在发送数据后一段时间内（可以设置重传计时器规定这段时间）没有收到确认序号ACK，那么发送方就会重新发送数据。

  这里发送方没有收到ACK可以分两种情况：

  1. 如果是发送方发送的数据包丢失了，接收方收到发送方重新发送的数据包(序列号大于了丢失的数据的序列号)后会马上重新给发送方发送(原丢失数据的)ACK；
  2. 如果是接收方之前接收到了发送方发送的数据包，而返回给发送方的ACK丢失了，这种情况，发送方重传后，接收方会直接丢弃发送方重传的数据包，然后再次发送ACK响应报文。如果数据被重发之后还是没有收到接收方的确认应答，则进行再次发送。此时，等待确认应答的时间将会以2倍、4倍的指数函数延长，直到最后关闭连接。

- 流量控制: 如果发送端发送的数据太快，接收端来不及接收就会出现丢包问题。

  ​		为了解决这个问题，TCP协议利用了**滑动窗口**进行了流量控制。在TCP首部有一个16位字段大小的窗口，窗口的大小就是接收端接收数据缓冲区的剩余大小。接收端会在收到数据包后发送ACK报文时，将自己的窗口大小填入ACK中，发送方会根据**ACK报文中的窗口大小**进而控制发送速度。如果窗口大小为零，发送方会停止发送数据。

- 拥塞控制:   如果网络出现拥塞，则会产生丢包等问题，这时发送方会将丢失的数据包继续重传，网络拥塞会更加严重，所以在网络出现拥塞时应注意控制发送方的发送数据，降低整个网络的拥塞程度。

  拥塞控制主要有四部分组成：**慢开始、拥塞避免、快重传、快恢复**

  ![截屏2022-08-28 下午5.40.02](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%885.40.02.png)

  （这里的发送方会维护一个拥塞窗口的状态变量，它和流量控制的滑动窗口是不一样的，滑动窗口是根据**接收方数据缓冲区大小**确定的，而**拥塞窗口**是根据网络的拥塞情况动态确定的，一般来说发送方真实的发送窗口为滑动窗口和拥塞窗口中的最小值）

  1. 慢开始:   为了避免一开始发送大量的数据而产生网络阻塞，会先初始化cwnd为1，当收到ACK后到下一个传输轮次，cwnd为2，以此类推成指数形式增长。
  2. 拥塞避免:   因为cwnd的数量在慢开始是指数增长的，为了防止cwnd数量过大而导致网络阻塞，会设置一个慢开始的门限值ssthresh，当cwnd>=ssthresh时，进入到拥塞避免阶段，cwnd每个传输轮次加1。但网络出现超时，会将门限值ssthresh变为出现超时cwnd数值的一半，cwnd重新设置为1，如上图，在第12轮出现超时后，cwnd变为1，ssthresh变为12。
  3. 快重传：在网络中如果出现超时或者阻塞，则按慢开始和拥塞避免算法进行调整。但如果只是丢失某一个报文段，如下图，则使用快重传算法。

  ![截屏2022-08-28 下午5.48.56](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%885.48.56.png)

  > 但是根据快重传算法，要求在这种情况下，需要快速向发送端发送M2的确认报文，在发送方收到三个M2的确认报文后，无需等待重传计时器所设置的时间，可直接进行M3的重传，这就是快重传。

  4. 快恢复:   当发送收到三个重复的ACK，会进行快重传和快恢复。快恢复是指将ssthresh设置为发生快重传时的cwnd数量的一半，而cwnd不是设置为1而是设置为为门限值ssthresh，并开始拥塞避免阶段。



#### TCP三次握手

![三次握手图](https://raw.githubusercontent.com/noobmid/pics/main/watermark%252Ctype_d3F5LXplbmhlaQ%252Cshadow_50%252Ctext_Q1NETiBA5ZGL5ZaD5ZCW%252Csize_20%252Ccolor_FFFFFF%252Ct_70%252Cg_se%252Cx_16.png)

发送端状态：CLOSED、SYN-SENT、ESTABLISHED

接收端状态：LISTEN、SYN-RCVD、ESTABLISHED

> 对于客户端而言,第二次握手就已经确定了建立链接,因此第三次握手实际上也算是数据传输的一次, 所以第一个数据包序列号为x+1

- TCP连接队列

  1. 半连接队列: 服务端收到客户端发起的 SYN 请求后，内核会把该连接存储到半连接队列（SYN队列），并向客户端响应 SYN+ACK.
  2. 全连接队列: 服务端收到第三次握手的ACK后，内核会把连接从半连接队列移除，然后创建新的完全的连接，并将其添加到accept队列，等待进程调用accept 函数时把连接取出来。

  

- 为什么握手需要三次?

  1. 假设建立TCP连接仅需要两次握手，那么如果第二次握手时，服务端返回给客户端的确认报文丢失了，客户端这边认为服务端没有和他建立连接，而服务端却以为已经和客户端建立了连接，并且可能服务端已经开始向客户端发送数据，但客户端并不会接收这些数据，浪费了资源。如果是三次握手，不会出现双方连接还未完全建立成功就开始发送数据的情况。
  2. 如果服务端接收到了一个早已失效的来自客户端的连接请求报文，会向客户端发送确认报文同意建立TCP连接。但因为客户端并不需要向服务端发送数据，所以此次TCP连接没有意义并且浪费了资源。
  3. 如果是一次握手,则退化成了UDP

- SYN洪泛攻击，以及解决策略是什么?

  ​        因为服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的，所以服务器容易受到SYN洪泛攻击。

  1. 洪泛攻击:    SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，由于源地址不存在，因此Server需要不断重发直至超时，这些伪造的SYN包将长时间占用半连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统资源被占用完而瘫痪。SYN 攻击是一种典型的 DoS/DDoS 攻击

  2. 查看检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。

     ```sh
     netstat -n -p TCP | grep SYN_RECV
     ```

     

  3. 常见的解决办法:

     1. 缩短超时（SYN Timeout）时间

     2. 增加最大半连接数: 可以保存更多的半连接数,防止丢弃新连接

     3. 增加过滤网关防护

     4. SYN cookies技术: 可以在不使用SYN半连接队列的情况下成功建立连接;

        ​	原理是，在TCP服务器接收到TCP SYN包并返回TCP SYN + ACK包时，不分配一个专门的数据区，而是根据这个SYN包计算出一个cookie值。这个cookie作为将要返回的SYN ACK包的初始序列号。当客户端返回一个ACK包时，根据包头信息计算cookie，与返回的确认序列号(初始序列号 + 1)进行对比，如果相同，则是一个正常连接，然后，分配资源，建立连接。

![SYN Cookies](https://raw.githubusercontent.com/noobmid/pics/main/watermark%252Ctype_ZHJvaWRzYW5zZmFsbGJhY2s%252Cshadow_50%252Ctext_Q1NETiBASmFja2V5czAwNw%253D%253D%252Csize_20%252Ccolor_FFFFFF%252Ct_70%252Cg_se%252Cx_16.png)

#### TCP四次挥手

![四次挥手](https://raw.githubusercontent.com/noobmid/pics/main/watermark%252Ctype_d3F5LXplbmhlaQ%252Cshadow_50%252Ctext_Q1NETiBA5ZGL5ZaD5ZCW%252Csize_20%252Ccolor_FFFFFF%252Ct_70%252Cg_se%252Cx_16-20220828202309378.png)

客户端状态：ESTABLISHED、FIN-WAIT-1、FIN-WAIT-2、TIME-WAIT、CLOSED

服务器状态：ESTABLISHED、CLOSE-WAIT、LAST-ACK、CLOSED

- 为什么要time_wait状态, 以及等待2MSL的时间?

  ​     MSL(Maximum Segment LifeTime)是报文最大生成时间，它是任何报文在网络上存在的最长时间，超过这个时间的报文将被丢弃。可以从两方面考虑：

  1. 客户端发送第四次挥手中的报文后，再经过2MSL，可使本次TCP连接中的所有报文全部消失，不会出现在下一个TCP连接中。

  2. 考虑丢包问题，如果第四挥手发送的报文在传输过程中丢失了，那么服务端没收到确认ack报文就会重发第三次挥手的报文。如果客户端发送完第四次挥手的确认报文后直接关闭，而这次报文又恰好丢失，则会造成服务端无法正常关闭。

- TIME_WAIT的作用

  1. 保证客户端发送的最后一个ack报文到达服务器

  2. 保证本次连接的报文在网络里消失

  3. time-wait定为2msl是为了保证服务端请求重传的报文到达

  4. tcp_tw_reuse、tcp_tw_recycle、tcp_timestamp可用于优化time_wait状态

- TIME_WAIT和CLOSE_WAIT的区别在哪?

  1. CLOSE_WAIT是被动关闭形成的，当客户端发送FIN报文，服务端返回ACK报文后进入CLOSE_WAIT。
  2. TIME__WAIT_是主动关闭形成的，当第四次挥手完成后，客户端进入TIME_WAIT状态

- TCP中的timewait状态过多会怎样?

  1. 占用过多的系统资源,占用服务器端口,导致无法使用, 建立链接失败
  2. 解决timewait状态过多的方法 :
     1. 允许timewait状态的端口被重用
     2. 减小timewait的时长
     3. 客户端头部设置keep-alive

- 出现了大量的CLOSE_WAIT状态怎么解决?

  ​		大量 CLOSE_WAIT 表示程序出现了问题，对方的 socket 已经关闭连接，而我方忙于读或写没有及时关闭连接，需 要检查代码，特别是释放资源的代码，或者是处理请求的线程配置。

- 为什么挥手需要四次?

  ​        由于TCP的**半关闭(half-close)**造成的。半关闭是指：TCP提供了连接的一方在结束它的发送后还能接受来自另一端数据的能力。通俗来说，就是不能发送数据，但是还可以接受数据。

  ​         当服务端发送完数据后还需要向客户端发送释放连接请求，客户端返回确认报文，TCP连接彻底关闭。所以断开TCP连接需要客户端和服务端分别通知对方并分别收到确认报文，一共需要四次。

#### 问题: 如果已经建立了TCP连接，但是客户端突然出现故障了怎么办？

​		如果TCP连接已经建立，在通信过程中，客户端突然故障，那么服务端不会一直等下去，过一段时间就关闭连接了。

​		TCP有一个保活机制，主要用在服务器端，用于检测已建立TCP链接的客户端的状态，防止因客户端崩溃或者客户端网络不可达，而服务器端一直保持该TCP链接，占用服务器端的大量资源(因为Linux系统中可以创建的总TCP链接数是有限制的)。

​		保活机制原理:  在TCP链接超过保活时间没有任何数据交互时，发送保活探测报文；在每次的保活探测报文都没有回复的时候, 则关闭连接.

### UDP

#### 数据结构

UDP的首部只有8个字节，源端口号、目的端口号、长度和校验和各两个字节。![截屏2022-08-28 下午8.47.16](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%888.47.16.png)



#### TCP和UDP对比

|      | 是否面向连接 | 可靠性 | 传输形式   | 传输效率 | 资源消耗 | 应用场景      | 首部字节 |
| ---- | :----------- | ------ | ---------- | -------- | -------- | ------------- | -------- |
| TCP  | 是           | 可靠   | 字节流     | 慢       | 多       | 文件/邮件传输 | 20-60    |
| UDP  | 否           | 不可靠 | 数据报文段 | 快       | 少       | 语音/视频     | 8        |





## [网络层]ARP协议

​		主要作用是实现从IP地址转换为MAC地址。

​		网络层实现的是主机之间的通信，而链路层实现的是链路之间的通信，所以从下图可以看出，在数据传输过程中，IP数据报的源地址(IP1)和目的地址(IP2)是一直不变的，而MAC地址(硬件地址)却一直随着链路的改变而改变。![截屏2022-08-28 下午9.52.26](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-28%20%E4%B8%8B%E5%8D%889.52.26.png)

ARP的工作流程:

1. 在局域网内，主机A要向主机B发送IP数据报时，首先会在主机A的ARP缓存表中查找是否有IP地址及其对应的MAC地址，如果有，则将MAC地址写入到MAC帧的首部，并通过局域网将该MAC帧发送到MAC地址所在的主机B。
2. 如果主机A的ARP缓存表中没有主机B的IP地址及所对应的MAC地址，主机A会在局域网内广播发送一个ARP请求分组。局域网内的所有主机都会收到这个ARP请求分组。
3. 主机B在看到主机A发送的ARP请求分组中有自己的IP地址，会向主机A以单播的方式发送一个带有自己MAC地址的响应分组。
4. 主机A收到主机B的ARP响应分组后，会在ARP缓存表中写入主机B的IP地址及其IP地址对应的MAC地址。
5. 如果主机A和主机B不在同一个局域网内，即使知道主机B的MAC地址也是不能直接通信的，必须通过路由器转发到主机B的局域网才可以通过主机B的MAC地址找到主机B。并且主机A和主机B已经可以通信的情况下，主机A的ARP缓存表中存的并不是主机B的IP地址及主机B的MAC地址，而是主机B的IP地址及该通信链路上的下一跳路由器的MAC地址。这就是上图中的源IP地址和目的IP地址一直不变，而MAC地址却随着链路的不同而改变。
6. 如果主机A和主机B不在同一个局域网，参考上图中的主机H1和主机H2，这时主机H1需要先广播找到路由器R1的MAC地址，再由R1广播找到路由器R2的MAC地址，最后R2广播找到主机H2的MAC地址，建立起通信链路。





## [网络层]IP协议

### TCP分段和IP分片

 分段分片的目的，都是为了能够传输上层交付的、数据量超过本层传输能力上限的数据，不得已才做的数据切分.

​	最大传输单元(Maximum Transmission Unit)，即MTU，为数据链路层的最大载荷上限。

​	最大报文段长度(Maximum Segment Size)，即MSS，为TCP传输层的最大载荷上限(即应用层数据最大长度)

MTU = MSS + TCP首部长度 + IP首部长度

- 分片发生在IP层，分段发生在tcp层

- IP层分片的原因是mtu的限制，tcp层分段的原因是mss的限制

- udp不进行分段会在ip层进行分片

### 有了IP地址，为什么还要用MAC地址？

​		标识网络中的一台计算机，比较常用的就是IP地址和MAC地址，但计算机的IP地址可由用户自行更改，管理起来相对困难，而MAC地址不可更改，所以一般会把IP地址和MAC地址组合起来使用。

​		随着网络中的设备越来越多，整个路由过程越来越复杂，便出现了子网的概念。对于目的地址在其他子网的数据包，路由只需要将数据包送到那个子网即可，这个过程就是上面说的ARP协议。

​		如果只是用MAC地址,路由器则需要记住每个MAC地址在哪个子网，这需要路由器有极大的存储空间，是无法实现的。如果只是用IP地址,没办法标志多个子网中唯一的设备.

​		IP地址可以比作为地址，MAC地址为收件人，在一次通信过程中，两者是缺一不可的





## [网络层]ICMP协议

​		网络层协议，主要是实现 IP 协议中未实现的部分功能，是一种网络层协议。该协议并不传输数据，只传输控制信息来辅助网络层通信.

应用:

1. ping

   ping的作用是测试两个主机的连通性。工作过程:

   1. 向目的主机发送多个ICMP回送请求报文

   2. 根据目的主机返回的回送报文的时间和成功响应的次数估算出数据包往返时间及丢包率。

      

2. TraceRoute

   ​		其主要用来跟踪一个分组从源点耗费最少 TTL 到达目的地的路径。TraceRoute 通过逐渐增大 TTL 值并重复发送数据报来实现其功能.(TTL, Time To live生存时间)

   1. 首先，TraceRoute 会发送一个 TTL 为 1 的 IP 数据报到目的地，当路径上的第一个路由器收到这个数据报时，它将 TTL 的值减 1，此时 TTL = 0，所以路由器会将这个数据报丢掉，并返回一个差错报告报文，
   2. 之后源主机会接着发送一个 TTL 为 2 的数据报，并重复此过程，直到数据报能够刚好到达目的主机。此时 TTL = 0，因此目的主机要向源主机发送 ICMP 终点不可达差错报告报文，之后源主机便知道了到达目的主机所经过的路由器 IP 地址以及到达每个路由器的往返时间。

## JWT

### Session、Cookie和Token的主要区别

- Cookie

​	Cookie是保存在客户端一个小数据块，其中包含了用户信息。当客户端向服务端发起请求，服务端会像客户端浏览器发送一个Cookie，客户端会把Cookie存起来，当下次客户端再次请求服务端时，会携带上这个Cookie，服务端会通过这个Cookie来确定身份。

- Session

​	Session是通过Cookie实现的，和Cookie不同的是，Session是存在服务端的。当客户端浏览器第一次访问服务器时，服务器会为浏览器创建一个sessionid，将sessionid放到Cookie中，存在客户端浏览器。比如浏览器访问的是购物网站，将一本《图解HTTP》放到了购物车，当浏览器再次访问服务器时，服务器会取出Cookie中的sessionid，并根据sessionid获取会话中的存储的信息，确认浏览器的身份是上次将《图解HTTP》放入到购物车那个用户。

- Token

​	客户端在浏览器第一次访问服务端时，服务端生成的一串字符串作为Token发给客户端浏览器，下次浏览器在访问服务端时携带token即可无需验证用户名和密码，省下来大量的资源开销。

### 客户端禁止 cookie 能实现 session 还能用吗

​	可以，Session的作用是在服务端来保持状态，通过sessionid来进行确认身份，但sessionid一般是通过Cookie来进行传递的。如果Cooike被禁用了，可以通过在**URL中传递sessionid**。





##  NAT

​	NAT（Network Address Translation），即网络地址转换，它是一种把内部私有网络地址翻译成公有网络 IP 地址的技术。

​	该技术不仅能解决 IP 地址不足的问题，而且还能隐藏和保护网络内部主机，从而避免来自外部网络的攻击。

NAT 的实现方式主要有三种：

- 静态转换：内部私有 IP 地址和公有 IP 地址是一对一的关系，并且不会发生改变。通过静态转换，可以实现外部网络对内部网络特定设备的访问，这种方式原理简单，但当某一共有 IP 地址被占用时，跟这个 IP 绑定的内部主机将无法访问 Internet。

- 动态转换：采用动态转换的方式时，私有 IP 地址每次转化成的公有 IP 地址是不唯一的。当私有 IP 地址被授权访问 Internet 时会被随机转换成一个合法的公有 IP 地址。当 ISP 通过的合法 IP 地址数量略少于网络内部计算机数量时，可以采用这种方式。

- 端口多路复用：该方式将外出数据包的源端口进行端口转换，通过端口多路复用的方式，实现内部网络所有主机共享一个合法的外部 IP 地址进行 Internet 访问，从而最大限度地节约 IP 地址资源。同时，该方案可以隐藏内部网络中的主机，从而有效避免来自 Internet 的攻击。

## URI 和URL的区别

- URI(Uniform Resource Identifier)：中文全称为统一资源标志符，主要作用是唯一标识一个资源。

- URL(Uniform Resource Location)：中文全称为统一资源定位符，主要作用是提供资源的路径。



## 路由器和交换机的区别？

|        | 所属网络模型的层级 |                             功能                             |
| ------ | :----------------: | :----------------------------------------------------------: |
| 路由器 |       网络层       | 识别IP地址并根据IP地址转发数据包，维护数据表并基于数据表进行最佳路径选择 |
| 交换机 |     数据链库层     |              识别MAC地址并根据MAC地址转发数据帧              |





## 在浏览器中输⼊url地址到显示主页的过程

1. 对输入到浏览器的url进行DNS解析，将域名转换为IP地址。

2. 和目的服务器建立TCP连接

3. 向目的服务器发送HTTP请求

4. 服务器处理请求并返回HTTP报文

5. 浏览器解析并渲染页面



## 一个主机可以建立多少连接

- 客户端 

​		每一个ip可建立的TCP连接理论受限于ip_local_port_range参数，也受限于65535。但可以通过配置多ip的方式来加大自己的建立连接的能力。

- 服务器

​		每一个监听的端口虽然理论值很大，但这个数字没有实际意义。最大并发数取决你的内存大小，每一条静止状态的TCP连接大约需要吃3.3K的内存

