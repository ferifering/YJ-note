Channel & Buffer
	channel 有一点类似于 stream，它就是读写数据的双向通道，可以从 channel 将数据读入 buffer，也可以将 buffer 的数据写入 channel，而之前的 stream 要么是输入，要么是输出，channel 比 stream 更为底层
常见的 Channel 有
	● FileChannel
	● DatagramChannel
	● SocketChannel
	● ServerSocketChannel

buffer 则用来缓冲读写数据，常见的 buffer 有
	● ByteBuffer
	● MappedByteBuffer
	● DirectByteBuffer
	● HeapByteBuffer
	● ShortBuffer
	● IntBuffer
	● LongBuffer
	● FloatBuffer
	● DoubleBuffer
	● CharBuffer