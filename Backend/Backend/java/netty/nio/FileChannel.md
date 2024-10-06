FileChannel 只能工作在阻塞模式下

不能直接打开 FileChannel，必须通过 FileInputStream、FileOutputStream 或者 RandomAccessFile 来获取 FileChannel，它们
都有 getChannel 方法
	● 通过 FileInputStream 获取的 channel 只能读
	● 通过 FileOutputStream 获取的 channel 只能写
	● 通过 RandomAccessFile 是否能读写根据构造 RandomAccessFile 时的读写模式决定
读取
会从 channel 读取数据填充 ByteBuffer，返回值表示读到了多少字节，-1 表示到达了文件的末尾
```
int readBytes = channel.read(buffer);
```
写入的正确姿势如下， SocketChannel
```
ByteBuffer buffer = ...;
buffer.put(...); // 存入数据
buffer.flip(); // 切换读模式
while(buffer.hasRemaining()) {
channel.write(buffer);
}
```
在 while 中调用 channel.write 是因为 write 方法并不能保证一次将 buffer 中的内容全部写入 channel
关闭
channel 必须关闭，不过调用了 FileInputStream、FileOutputStream 或者 RandomAccessFile 的 close 方法会间接地调用channel 的 close 方法
强制写入
操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘。可以调用 force(true) 方法将文件内容和元数据（文件的权限等信
息）立刻写入磁盘

```
String FROM = "helloword/data.txt";
String TO = "helloword/to.txt";
long start = System.nanoTime();
	try (FileChannel from = new FileInputStream(FROM).getChannel();
		FileChannel to = new FileOutputStream(TO).getChannel();
	) {
	from.transferTo(0, from.size(), to);
} catch (IOException e) {
	e.printStackTrace();
}
long end = System.nanoTime();
System.out.println("transferTo 用时：" + (end - start) / 1000_000.0);
```
使用transferTo时会使用系统的零拷贝进行优化
	传输数据有上限，最大2g
```
	for (long left = size; left > 0; ) {
	System.out.println("position:" + (size - left) + " left:" + left);
	left -= from.transferTo((size - left), left, to);
}
```