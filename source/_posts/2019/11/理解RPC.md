---
title: 理解RPC及原理实现
permalink: meet-RPC
tags:
  - Node.js
categories:
  - 其他
date: 2019-11-07 14:43:21
updated: 2019-11-07 14:43:21
---

# 概述
缘起学习Node.js过程中实现BFF层时涉及到RPC概念，于是花了两三天时间好好整理了下相关知识点。从概念到原理再到Node.js的实现示例。重点在于理解RPC的技术思想，因为具体实践中都是用成熟的RPC框架。
<!-- more -->

# 详述

## 概念
RPC(Remote Procedure Call)：远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的思想。RPC 是一种技术思想而非一种规范或协议，熟悉Ajax和Redux的同学，学完RPC可能会找到一种“熟悉的陌生人”的感觉。

## 原理

### 完整框架
在一个典型 RPC 的使用场景中，包含了服务发现、负载、容错、网络传输、序列化等组件，其中“RPC协议”部分是实现一个RPC架构的核心，指明了程序如何进行网络传输和序列化。
![完整的RPC框架](01.jpg)

### 核心功能
一个 RPC 的核心功能主要有 5 个部分组成，分别是：客户端、客户端 Stub、网络传输模块、服务端 Stub、服务端。
![RPC核心功能](02.png)

一次 RPC 调用流程如下：

-  服务消费者(Client 客户端)通过本地调用的方式调用服务。
-  客户端存根(Client Stub)接收到调用请求后负责将方法、入参等信息序列化(组装)成能够进行网络传输的消息体。
-  客户端存根(Client Stub)找到远程的服务地址，并且将消息通过网络发送给服务端。
-  服务端存根(Server Stub)收到消息后进行解码(反序列化操作)。
-  服务端存根(Server Stub)根据解码结果调用本地的服务进行相关处理
-  服务端(Server)本地服务业务处理。
-  处理结果返回给服务端存根(Server Stub)。
-  服务端存根(Server Stub)序列化结果。
-  服务端存根(Server Stub)将结果通过网络发送至消费方。
-  客户端存根(Client Stub)接收到消息，并进行解码(反序列化)。
-  服务消费方得到最终结果。

RPC 的核心功能主要由 5 个模块组成，如果想要自己实现一个 RPC，最简单的方式要实现三个技术点，分别是：服务寻址、数据流的序列化和反序列化和网络传输。

#### 服务寻址
服务寻址可以使用 Call ID 映射。在本地调用中，函数体是直接通过函数指针来指定的，但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。
所以在 RPC 中，所有的函数都必须有自己的一个 ID。这个 ID 在所有进程中都是唯一确定的。
客户端在做远程过程调用时，必须附上这个 ID。然后我们还需要在客户端和服务端分别维护一个函数和Call ID的对应表。
当客户端需要进行远程调用时，它就查一下这个表，找出相应的 Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

实现方式：服务注册中心。

要调用服务，首先你需要一个服务注册中心去查询对方服务都有哪些实例。Dubbo 的服务注册中心是可以配置的，官方推荐使用 Zookeeper。

#### 序列化和反序列化
客户端怎么把参数值传给远程的函数呢?在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。
但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。
这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。
只有二进制数据才能在网络中传输，序列化和反序列化的定义是：
-  将对象转换成二进制流的过程叫做序列化
-  将二进制流转换成对象的过程叫做反序列化

这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。
#### 网络传输
客户端和服务端是通过网络连接的，所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把 Call ID 和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。
只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。
尽管大部分 RPC 框架都使用 TCP 协议，但其实 UDP 也可以，而 gRPC 干脆就用了 HTTP2。
TCP 的连接是最常见的，简要分析基于 TCP 的连接：通常 TCP 连接可以是按需连接(需要调用的时候就先建立连接，调用结束后就立马断掉)，也可以是长连接(客户端和服务器建立起连接之后保持长期持有，不管此时有无数据包的发送，可以配合心跳检测机制定期检测建立的连接是否存活有效)，多个远程过程调用共享同一个连接。
所以，要实现一个 RPC 框架，只需要把以下三点实现了就基本完成了：

-  Call ID 映射：可以直接使用函数字符串，也可以使用整数 ID。映射表一般就是一个哈希表。
-  序列化反序列化：可以自己写，也可以使用 Protobuf 或者 FlatBuffers 之类的。
-  网络传输库：可以自己写 Socket，或者用 Asio，ZeroMQ，Netty 之类。

至于TCP、UDP、HTTP之间的区别，简单说来就是相对于HTTP，TCP和UDP更底层，传输的是二进制流，因此体积更小更快，当然缺点就是实现更复杂。反过来说就是，HTTP实现简单，可以利用成熟的web服务器收发请求，有完善的工具处理JSON、XML数据格式，但是相对来说体积大速度慢。至于TCP与UDP的区别，简言之就是TCP可靠、稳定，UDP更快但是可能会丢包。
因此，大部分RPC框架会选择TCP协议作为传输层实现。

## 实现示例（Node.js、TCP）

服务端实现示例：
```js
// server.js---------------------------------
const net = require('net');

// 假数据
const LESSON_DATA = {
  136797: '01 | 课程介绍',
  136798: '02 | 内容综述',
  136799: '03 | Node.js是什么？',
  136800: '04 | Node.js可以用来做什么？',
  136801: '05 | 课程实战项目介绍',
  136803: '06 | 什么是技术预研？',
  136804: '07 | Node.js开发环境安装',
  136806: '08 | 第一个Node.js程序：石头剪刀布游戏',
  136807: '09 | 模块：CommonJS规范',
  136808: '10 | 模块：使用模块规范改造石头剪刀布游戏',
  136809: '11 | 模块：npm',
  141994: '12 | 模块：Node.js内置模块',
  143517: '13 | 异步：非阻塞I/O',
  143557: '14 | 异步：异步编程之callback',
  143564: '15 | 异步：事件循环',
  143644: '16 | 异步：异步编程之Promise',
  146470: '17 | 异步：异步编程之async/await',
  146569: '18 | HTTP：什么是HTTP服务器？',
  146582: '19 | HTTP：简单实现一个HTTP服务器'
};

const server = net.createServer(socket => {
  let oldBuffer = null;
  socket.on('data', buffer => {
    // 把上一次data事件使用残余的buffer接上来
    if (oldBuffer) {
      buffer = Buffer.concat([oldBuffer, buffer]);
    }
    let packageLength = 0;
    // 只要还存在可以解成完整包的包长
    while ((packageLength = checkComplete(buffer))) {
      const package = buffer.slice(0, packageLength);
      buffer = buffer.slice(packageLength);

      // 把这个包解成数据和seq
      const result = decode(package);

      // 计算得到要返回的结果，并write返回
      socket.write(encode(LESSON_DATA[result.data], result.seq));
    }
    // 把残余的buffer记下来
    oldBuffer = buffer;
  });
});

server.listen(4000);

/**
 * 二进制包编码函数
 * 在一段rpc调用里，服务端需要经常编码rpc调用时，业务数据的返回包
 */
function encode(data, seq) {
  // 正常情况下，这里应该是使用 protobuf 来encode一段代表业务数据的数据包
  // 为了不要混淆重点，这个例子比较简单，就直接把课程标题转buffer返回
  const body = Buffer.from(data);

  // 一般来说，一个rpc调用的数据包会分为定长的包头和不定长的包体两部分
  // 包头的作用就是用来记载包的序号和包的长度，以实现全双工通信
  const header = Buffer.alloc(6);
  header.writeInt16BE(seq);
  header.writeInt32BE(body.length, 2);

  const buffer = Buffer.concat([header, body]);

  return buffer;
}

/**
 * 二进制包解码函数
 * 在一段rpc调用里，服务端需要经常解码rpc调用时，业务数据的请求包
 */
function decode(buffer) {
  const header = buffer.slice(0, 6);
  const seq = header.readInt16BE();

  // 正常情况下，这里应该是使用 protobuf 来decode一段代表业务数据的数据包
  // 为了不要混淆重点，这个例子比较简单，就直接读一个Int32即可
  const body = buffer.slice(6).readInt32BE();

  // 这里把seq和数据返回出去
  return {
    seq,
    data: body
  };
}

/**
 * 检查一段buffer是不是一个完整的数据包。
 * 具体逻辑是：判断header的bodyLength字段，看看这段buffer是不是长于header和body的总长
 * 如果是，则返回这个包长，意味着这个请求包是完整的。
 * 如果不是，则返回0，意味着包还没接收完
 * @param {} buffer
 */
function checkComplete(buffer) {
  if (buffer.length < 6) {
    return 0;
  }
  const bodyLength = buffer.readInt32BE(2);
  return 6 + bodyLength;
}
```
客户端实现示例：
```js
// client.js-------------------

const net = require('net');

const LESSON_IDS = [
  "136797",
  "136798",
  "136799",
  "136800",
  "136801",
  "136803",
  "136804",
  "136806",
  "136807",
  "136808",
  "136809",
  "141994",
  "143517",
  "143557",
  "143564",
  "143644",
  "146470",
  "146569",
  "146582"
]

const socket = new net.Socket({});

socket.connect({
    host: '127.0.0.1',
    port: 4000
});

let id = Math.floor(Math.random() * LESSON_IDS.length);

let oldBuffer = null;
socket.on('data', (buffer) => {
    // 把上一次data事件使用残余的buffer接上来
    if (oldBuffer) {
        buffer = Buffer.concat([oldBuffer, buffer]);
    }
    let completeLength = 0;

    // 只要还存在可以解成完整包的包长
    while (completeLength = checkComplete(buffer)) {
        const package = buffer.slice(0, completeLength);
        buffer = buffer.slice(completeLength);

        // 把这个包解成数据和seq
        const result = decode(package);
        console.log(`包${result.seq}，返回值是${result.data}`);
    }

    // 把残余的buffer记下来
    oldBuffer = buffer;
})


let seq = 0;
/**
 * 二进制包编码函数
 * 在一段rpc调用里，客户端需要经常编码rpc调用时，业务数据的请求包
 */
function encode(data) {
    // 正常情况下，这里应该是使用 protobuf 来encode一段代表业务数据的数据包
    // 为了不要混淆重点，这个例子比较简单，就直接把课程id转buffer发送
    const body = Buffer.alloc(4);
    body.writeInt32BE(LESSON_IDS[data.id]);

    // 一般来说，一个rpc调用的数据包会分为定长的包头和不定长的包体两部分
    // 包头的作用就是用来记载包的序号和包的长度，以实现全双工通信
    const header = Buffer.alloc(6);
    header.writeInt16BE(seq)
    header.writeInt32BE(body.length, 2);

    // 包头和包体拼起来发送
    const buffer = Buffer.concat([header, body])

    console.log(`包${seq}传输的课程id为${LESSON_IDS[data.id]}`);
    seq++;
    return buffer;
}

/**
 * 二进制包解码函数
 * 在一段rpc调用里，客户端需要经常解码rpc调用时，业务数据的返回包
 */
function decode(buffer) {
    const header = buffer.slice(0, 6);
    const seq = header.readInt16BE();

    const body = buffer.slice(6)

    return {
        seq,
        data: body.toString()
    }
}

/**
 * 检查一段buffer是不是一个完整的数据包。
 * 具体逻辑是：判断header的bodyLength字段，看看这段buffer是不是长于header和body的总长
 * 如果是，则返回这个包长，意味着这个请求包是完整的。
 * 如果不是，则返回0，意味着包还没接收完
 * @param {} buffer 
 */
function checkComplete(buffer) {
    if (buffer.length < 6) {
        return 0;
    }
    const bodyLength = buffer.readInt32BE(2);
    return 6 + bodyLength
}

for (let k = 0; k < 100; k++) {
    id = Math.floor(Math.random() * LESSON_IDS.length);
    socket.write(encode({ id }));
}
```

# 参考
http://developer.51cto.com/art/201906/597963.htm

https://www.yuque.com/egg/nodejs/dklip5

https://time.geekbang.org/course/detail/232-152724