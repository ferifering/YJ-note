selector 单从字面意思不好理解，需要结合服务器的设计演化来理解它的用途
多线程版设计

## subgraph 多线程版
```
t1(thread) --> s1(socket1)
t2(thread) --> s2(socket2)
t3(thread) --> s3(socket3)
```
⚠️ 多线程版缺点
● 内存占用高
● 线程上下文切换成本高
● 只适合连接数少的场景
线程池版设计

## subgraph 线程池版
```
t4(thread) --> s4(socket1)
t5(thread) --> s5(socket2)
t4(thread) -.-> s6(socket3)
t5(thread) -.-> s7(socket4)
```
⚠️ 线程池版缺点
● 阻塞模式下，线程仅能处理一个 socket 连接
● 仅适合短连接场景

## selector 版设计
selector 的作用就是配合一个线程来管理多个 channel，获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，
不会让线程吊死在一个 channel 上。适合连接数特别多，但流量低的场景（low traffic）

```
thread --> selector
selector --> c1(channel)
selector --> c2(channel)
selector --> c3(channel)
```

调用 selector 的 select() 会阻塞直到 channel 发生了读写就绪事件，这些事件发生，select 方法就会返回这些事件交给 thread 来
处理

多路复用
单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用
● 多路复用仅针对网络 IO、普通文件 IO 没法利用多路复用
● 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
	● 有可连接事件时才去连接
	● 有可读事件才去读取
	● 有可写事件才去写入
	● 限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件
	
好处
● 一个线程配合 selector 就可以监控多个 channel 的事件，事件发生线程才去处理。避免非阻塞模式下所做无用功
● 让这个线程能够被充分利用
● 节约了线程的数量
● 减少了线程上下文切换
创建
```
Selector selector = Selector.open();
```
绑定 Channel 事件
也称之为注册事件，绑定的事件 selector 才会关心
```
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, 绑定事件);
```
● channel 必须工作在非阻塞模式
● FileChannel 没有非阻塞模式，因此不能配合 selector 一起使用
● 绑定的事件类型可以有
	● connect - 客户端连接成功时触发
	● accept - 服务器端成功接受连接时触发
	● read - 数据可读入时触发，有因为接收能力弱，数据暂不能读入的情况
	● write - 数据可写出时触发，有因为发送能力弱，数据暂不能写出的情况
监听 Channel 事件
可以通过下面三种方法来监听是否有事件发生，方法的返回值代表有多少 channel 发生了事件
方法1，阻塞直到绑定事件发生
```
int count = selector.select();
```
方法2，阻塞直到绑定事件发生，或是超时（时间单位为 ms）
```
int count = selector.select(long timeout);
```
方法3，不会阻塞，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件
```
int count = selector.selectNow();
```
💡 select 何时不阻塞
● 事件发生时
	● 客户端发起连接请求，会触发 accept 事件
	● 客户端发送数据过来，客户端正常、异常关闭时，都会触发 read 事件，另外如果发送的数据大于 buffer 缓冲
	区，会触发多次读取事件
	● channel 可写，会触发 write 事件
	● 在 linux 下 nio bug 发生时
● 调用 selector.wakeup()
● 调用 selector.close()
● selector 所在线程 interrupt