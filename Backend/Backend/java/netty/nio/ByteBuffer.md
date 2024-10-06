ByteBuffer 有以下重要属性
```
● capacity  // 容量
● position  // 写入位置（索引）
● limit  //写入限制
```
![[Pasted image 20241004205508.png]]
写模式下，position 是写入位置，limit 等于容量，下图表示写入了 4 个字节后的状态
![[Pasted image 20241004205437.png]]
clear 成为写模式
![[Pasted image 20241004205535.png]]
compact 方法，是把未读完的部分向前压缩，然后切换至写模式
![[Pasted image 20241004205659.png]]

Bytebuffer buf = ByteBuffer.allocate(16); //从java堆内存申请空间，读写效率低，且会受到GC影响（垃圾整理，内存搬迁）。
Bytebuffer buf = ByteBuffer.allocateDirect(16); //从系统内存申请空间，读写效率高（少拷贝一次）

向 buffer 写入数据
	有两种办法
	● 调用 channel 的 read 方法
	● 调用 buffer 自己的 put 方法
	int readBytes = channel.read(buf);
	和
	buf.put((byte)127);
从 buffer 读取数据
	同样有两种办法
	● 调用 channel 的 write 方法
	● 调用 buffer 自己的 get 方法
	int writeBytes = channel.write(buf);
	和
	byte b = buf.get();
	get 方法会让 position 读指针向后走，如果想重复读取数据
	● 可以调用 rewind 方法将 position 重新置为 0
	● 或者调用 get(int i) 方法获取索引 i 的内容，它不会移动读指针
	mark 和 reset
	mark 是在读取时，做一个标记，即使 position 改变，只要调用 reset 就能回到 mark 的位置

字符串转bytebuffer
1. String.getBytes() 将字符串传唤为byte[],然后buffer.put()
2. 通过标准字符集转换
	1. ByteBuffer buffer1 = StandardCharsets.UTF_8.encode("你好");
	2. ByteBuffer buffer2 = Charset.forName("utf-8").encode("你好");
3. 通过 wrap方法
	1. ByteBuffer buffer1 = ByteBuffer.wrap("string".getBytes());
转字符串
```
CharBuffer buffer3 = StandardCharsets.UTF_8.decode(buffer1);
System.out.println(buffer3.getClass());
System.out.println(buffer3.toString());
```
Buffer 是**非线程安全的**
## 2.4 Scattering Reads
分散读取，有一个文本文件 3parts.txt
onetwothree
使用如下方式读取，可以将数据填充至多个 buffer
```
try (RandomAccessFile file = new RandomAccessFile("helloword/3parts.txt", "rw")) {
	FileChannel channel = file.getChannel();
	ByteBuffer a = ByteBuffer.allocate(3);
	ByteBuffer b = ByteBuffer.allocate(3);
	ByteBuffer c = ByteBuffer.allocate(5);
	channel.read(new ByteBuffer[]{a, b, c});
	a.flip();
	b.flip();
	c.flip();
	debug(a);
	debug(b);
	debug(c);
} catch (IOException e) {
	e.printStackTrace();
}
```

## 2.5 Gathering Writes
使用如下方式写入，可以将多个 buffer 的数据填充至 channel
```
try (RandomAccessFile file = new RandomAccessFile("helloword/3parts.txt", "rw")) {
	FileChannel channel = file.getChannel();
	 ByteBuffer d = ByteBuffer.allocate(4);
	ByteBuffer e = ByteBuffer.allocate(4);
	channel.position(11);
	d.put(new byte[]{'f', 'o', 'u', 'r'});
	e.put(new byte[]{'f', 'i', 'v', 'e'});
	d.flip();
	e.flip();
	debug(d);
	debug(e);
	channel.write(new ByteBuffer[]{d, e});
} catch (IOException e) {
	e.printStackTrace();
}
```

粘包半包现象

> [!### 黏包（Stick Packet）

**黏包**发生在接收方无法正确地将多个数据包分开时，导致多个数据包被合并为一个数据包传输。这种情况通常发生在以下场景中：

- **发送方快速连续发送数据**：由于 TCP 是流式协议，接收方可能在读取数据时，实际上接收到了多个连续的发送数据而未分开。
- **没有明确的消息边界**：如果协议设计没有提供每个数据包的大小或结束标志，接收方将无法知道何时截止接收一个完整的数据包。

#### 解决方案

- **添加消息头**：在每个数据包前添加固定长度的消息头，包含消息体的长度信息，接收方先读取这个长度，再根据长度读取完整数据。
- **使用结束符**：在每个数据包后添加特殊字符（例如，换行符或其他特定分隔符），接收方可以根据这个字符判断数据包的结束。

### 半包（Half Packet）

**半包**发生在接收方只能接收到一个数据包的一部分，导致需要等待更多数据来完成整个数据包的接收。半包现象通常发生于以下场景：

- **发送方发送大数据包**：当数据包的大小超过 TCP 缓冲区的处理能力，数据包会被分割成多个小段进行传输。在接收方读取数据时，可能恰好读到了一部分数据，而不是完整的数据包。
- **网络延迟或丢包**：在高延迟或不稳定的网络环境下，数据可能由于网络原因导致接收不完整。

#### 解决方案

- **缓冲区处理**：接收方应使用一个缓冲区来存储接收到的原始数据，直到可以拼接出完整的数据包。可以根据协议设计来检查数据的完整性。
- **流控机制**：在发送方和接收方之间建立合适的流控机制，确保接收方能处理完一部分再发送新的数据。
>
网络上有多条数据发送给服务端，数据之间使用 \n 进行分隔 但由于某种原因这些数据在接收时，被进行了重新组合，例如原始
数据有3条为
	● Hello,world\n
	● I'm zhangsan\n
	● How are you?\n
变成了下面的两个 byteBuffer (黏包，半包)
	● Hello,world\nI'm zhangsan\nHo
	● w are you?\n
```
·public static void main(String[] args) {
	ByteBuffer source = ByteBuffer.allocate(32);
	
	source.put("Hello,world\nI'm zhangsan\nHo".getBytes());
	split(source);
	 source.put("w are you?\nhaha!\n".getBytes());
	split(source);
}
private static void split(ByteBuffer source) {
	source.flip();
	int oldLimit = source.limit();
	for (int i = 0; i < oldLimit; i++) {
		if (source.get(i) == '\n') {
		System.out.println(i);
		ByteBuffer target = ByteBuffer.allocate(i + 1 - source.position());
		source.limit(i + 1);
		target.put(source); // 从source 读，向 target 写
		debugAll(target);
		source.limit(oldLimit);
		}
	}
	source.compact();
}
```