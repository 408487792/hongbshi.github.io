---
title: 水平触发与边沿触发
date: 2018-11-05 11:21:50
categories:
- 网络编程
- 基础
tags:
- 水平触发
- 边沿触发
- 网络编程
---
# 一. 基础
## 1.1 水平触发
1. 基本概念
> - 读缓冲区不为空时, 读事件触发。
> - 写缓冲区不为满时, 写事件触发。

2. 处理流程
> - accept新的连接, 监听读事件。
> - 读事件到达, 处理读事件。
> - 需要写入数据, 向fd中写数据, 一次无法写完, 开启写事件监听。
> - 写事件到达, 继续写入数据, 写完后关闭写事件。

3. 优缺点
> - 不会遗漏事件, 易编程。
> - 长连接需要写入的数据量大时, 会频繁开启关闭写事件。  

## 1.2 边沿触发
1. 基本概念
> - 读缓冲区状态变化时, 读事件触发, 网卡接受到新数据。
> - 写缓冲区状态变化时, 写事件触发, 网卡发出了新数据。

2. 处理流程
> - accept新的连接, 同时监听读写事件。
> - 读事件到达, 需要一直读取数据, 直到返回EAGAIN。
> - 写事件到达, 无数据处理则不处理, 有数据待写入则一直写入，直到写完或者返回EAGAIN。

3. 优缺点
> - 不需要频繁开启关闭事件, 效率较高。
> - 读写事件处理不当, 可能导致事件丢失, 编程教复杂。

## 1.3 选择
1. 概述
> - 对于读事件而言，总体而言, 采用水平触发方式较好。应用程序在读取数据时，可能会一次无法读取全部数据，边沿触发在下一次可能不会触发。如果能够保证一次读取缓存的全部数据，可以采用边沿触发，效率更高, 但同时编程复杂度也高。
> - 对于写事件，当客户端服务端采用短连接或者采用长连接但发送的数据量比较少时(例如: Redis), 采用水平触发即可。当客户端与服务端是长连接并且数据写入的量比较大时(例如: nginx), 采用边沿触发, 因为边沿触发效率更高。
> - 目前，linux不支持读写事件分别设置不同的触发方式，具体采用哪种方式触发，需要根据具体需求。


2. 监听套接字事件设置
> - 监听套接字不需要监听写事件，只需要监听读事件。
> - 监听套接字一般采用水平触发方式。(nginx开启multi_accept时，会把监听套接字所有可读的事件全部读取，此时可以使用边沿触发。但为了保证连接不丢失，nginx仍然采用水平触发)

3. 通信套接字设置
> - redis对于与客户端通信使用的套接字默认使用水平触发。
> - nginx对于与客户端通信使用的套接字默认采用边沿触发。


# 二. 参考
- https://blog.csdn.net/dongfuye/article/details/50880251