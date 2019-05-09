---
title: "使用 raw socket 进行网络读写"
date: 2018-10-03
categories:
  - Socket
---

第一次接触 [raw socket](https://en.wikipedia.org/wiki/Network_socket#Raw_socket) 是在大三的时候，那时候旁听网络攻防课，老师提到可以把网卡切换到混杂模式，然后使用 raw socket 进行监听，做坏事。但是当时只是对 Raw socket 有了一些基本的了解，都没有编程实现。这次由于项目的需要实现了一个 UDP proxy，使用到了 raw socket，于是总结一下使用经验。在本篇博客中，我们的目的是实现一个使用raw socket读取UDP数据包，并转发出去的程序。值得注意的是，我们不是简单的转发UDP数据包的内容，而是完整转发整个IP数据包。换而言之，我们收到IP数据包长什么样子，发出去的就还是这样。

我们知道，在Linux中，每条网络连接都对应于一个socket文件，想要执行网络操作的话，无论是作为客户端还是服务端，第一步都是创建一个socket文件。创建socket的命令是`int socket(int domain, int type, int protocol)`,返回值便是被创建的socket文件的描述符。其中第二个参数type指定了网络协议类型，可以是SOCK_STREAM，SOCK_DGRAM，或者SOCK_RAW，他们分别对应于TCP，UDP，以及raw socket。于是我们可以通过如下命令创建一个raw socket。

```c
int sockfd = socket(PF_PACKET, SOCK_RAW, htons(ETHER_TYPE)));
```

之后我们便需要使用这个socket监听网络端口。在监听之前，我们还需要设置一个socket option，用于讲socket绑定到特定的网卡(eth0)上，命令如下。

```c
char* ifName = "eth0";
setsockopt(sockfd, SOL_SOCKET, SO_BINDTODEVICE, ifName, 4)
```

于是，我们便可以使用这个socket监听网络了。

```c
#define BUF_SIZ        (1024 * 128)
uint8_t buf[BUF_SIZ];
int numbytes = recvfrom(sockfd, buf, BUF_SIZ, 0, NULL, NULL);
```

我们将所有的网络数据包缓存在buf中，因为我们在创建socket的时候指定了协议为`ETHER_TYPE`，所以这里buf里面的将是包含物理层报头的报文。之后我们需要将原始报文的每一部分进行解析。比如说，`iph->saddr`就是数据包的源IP地址。

```c
struct ether_header *eh = (struct ether_header *) buf;
struct iphdr *iph = (struct iphdr *) (buf + sizeof(struct ether_header));
struct udphdr *udph = (struct udphdr *) (buf + sizeof(struct iphdr) + sizeof(struct ether_header));
```

因为raw socket会监听所有的数据包，所以我们需要对监听到的数据包进行筛选，找到发送到端口8080的UDP数据包。

```c
if (iph->protocol != IPPROTO_UDP || udph->dest != htons(8080)) {
  return;
}
```

筛选出满足要求的数据包后，我们便可以将他从其他网卡转发出去。转发出去的过程和接收类似，我们需要创建一个新的raw socket，与另外一个网卡(eth1)绑定，再执行写的操作。

```c
char *opt = "eth1";
int len = strnlen(opt, IFNAMSIZ);
int sd = socket(AF_INET, SOCK_RAW, IPPROTO_RAW);
int on = 1;
setsockopt(sd, SOL_SOCKET, SO_BINDTODEVICE, opt, len)
setsockopt(sd,IPPROTO_IP,IP_HDRINCL,&on,sizeof(on))
struct sockaddr_in target;
target.sin_family = AF_INET;
target.sin_port = udph->dest;
target.sin_addr.s_addr = iph->daddr;
sendto(sd, iph, ntohs(iph->tot_len), MSG_DONTROUTE, (struct sockaddr *)&target, sizeof(target))
```

值得注意的是，这里我们又设置了两个socket option，`SO_BINDTODEVICE`前文介绍过，用于绑定socket到网卡上，`IP_HDRINCL`的作用是告诉底层协议栈，我们的数据以及包含了IP头，在转发的时候不要再添加IP报文头了。

由此，我们便成功的将来自eth0的UDP数据包从eth1转发出去，实现了一个简单的UDP proxy的第一步。在这里，我们进行数据包转发的时候没有修改IP报文的源地址和目的地址，如果只是简单这么做的话后续网络是无法正常进行路由的，因为目的地址就是自己的IP地址。另一方面，我们转发出去了一个源地址并不属于自己的数据包，这在网络中也是可能出现问题的，我们将在后续博文中解决这个问题。
