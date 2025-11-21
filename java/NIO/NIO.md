**非阻塞读写：通过选择器来控制读写，当需要进行读写的时候才分配内存**

# 一、channel

```
通道、可双向读写。全双工
有FileChannel,DatagramChannel,SocketChannel,ServerSocketChannel
Channel 用于字节缓冲区和位于通道另一侧的实体之间有效的传递数据

分散： 将channelz中的数据发散到每一个buffer，默认是一个写满之后，写另一个
聚集： 将多个buffer中数据写到一个channel中，只写position与limit之间的内容。按照存放顺序写
```



# 二、buffer

```
缓冲区

读取数据流程：
1、写数据到buffer
2、调用flip()方法
3、从buffer中读取数据
4、调用clear（）或者compact()方法
	clear:会清空整个缓冲区
	compact()：只会清除读过的数据任何未读的数据会被移动缓冲区起始位置
```

# 三、Selector

```
Selector 运行单线程处理多个Channel.控制选择哪个Channel
```

![](/Users/peilizhi/screenshots/截屏2021-09-11 15.00.25.png)

**从通道读取数据到缓冲区，从缓冲区写数据到通道**



