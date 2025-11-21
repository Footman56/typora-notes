```
capacity:固定大小值
position: 
	写模式下表示当前已经写到的位置
	读模式下表示当前已经读到的位置
limit:
	写模式下表示最多可写到哪里，limit=capacity
	读模式下表示最多可读到哪里（有效位置） limit=之前写模式下 position的位置
	
flip（）
	limit = position;
  position = 0;
```

# 写数据到Buffer

```
1、channel的read（）方法
2、buffer.put()
```

filp() : 从写模式切换到读模式

## 从Buffer中读数据

```
1、channel的write()方法
2、buffer.get()
```

```
rewind()   position=0,limit 不变，重新读
clear()    position=0,limit = capacity
mark（）。标定一个特定的position
reset()  回到特定的position
```



```
 只读缓冲区  byteBuffer.asReadOnlyBuffer() 不能修改，但父缓冲区可以修改
 内存缓冲区          ByteBuffer.allocateDirect(1024)
 子缓冲区  byteBuffer.slice(); 根据position与limit 划分，共享内存
```

