# JDK NIO 与 OIO

## IO 的多种形式

- 文件(计算机块设备,硬盘)
  
- TCP
  
  java 的 Socket 通信,

- UDP
  
- UDS(Unix Domain Socket)

## Linux 一些基础概念

- 用户态
- 内核态

## Linux IO五种模型

- 阻塞IO
- 非阻塞IO

java oio(InputStream,OutputStream) 可以实现阻塞式也可以实现非阻塞式模型.但是均是同步模型.InputStream#available 可以避免在读入操作上阻塞.但是在 OutPutStream 操作上则无法避免写出操作的同步阻塞操作.

- IO多路复用

java nio 通过 Selector(select,poll,epoll 实现的多路复用调用机制)

- 信号驱动IO
- 异步IO。

AIO():
OIO():
NIO():

## IO 常见问题

- 1GB 文件,我只要读取中间的 960M-970MB怎么操作?

- 作为 TCP 的服务端怎么保证高效?

## OIO 简介

## NIO 简介

## OIO API

## NIO 基础概念

### Channel

- FileChannel

- SelectableChannel
  
- SocketChannel (TCP)
  
- DatagramChannel(UDP)

### Buffer

- DirectBuffer
  
- HeapBuffer

### Selector

- EPollSelectorImpl

- PollSelectorImpl

## Reference

- [Open JDK源码包含 hotspot 虚拟机 和 JDK Native 实现](https://github.com/openjdk/jdk)
