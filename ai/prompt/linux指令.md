```
2025-02-20T10:29:08.833+0800: 48968.079: [GC pause (G1 Evacuation Pause) (young), 0.0402826 secs]
   [Parallel Time: 36.8 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 48968080.3, Avg: 48968080.3, Max: 48968080.4, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 13.0, Avg: 13.3, Max: 13.5, Diff: 0.6, Sum: 53.1]
      [Update RS (ms): Min: 7.8, Avg: 7.8, Max: 7.8, Diff: 0.0, Sum: 31.3]
         [Processed Buffers: Min: 50, Avg: 70.2, Max: 84, Diff: 34, Sum: 281]
      [Scan RS (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.0, Sum: 1.1]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.4]
      [Object Copy (ms): Min: 14.7, Avg: 14.9, Max: 15.3, Diff: 0.6, Sum: 59.6]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 2.5, Max: 4, Diff: 3, Sum: 10]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.2, Max: 0.3, Diff: 0.3, Sum: 0.7]
      [GC Worker Total (ms): Min: 36.4, Avg: 36.5, Max: 36.7, Diff: 0.3, Sum: 146.1]
      [GC Worker End (ms): Min: 48968116.7, Avg: 48968116.8, Max: 48968117.0, Diff: 0.3]
   [Code Root Fixup: 0.1 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.6 ms]
   [Other: 2.8 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.5 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.1 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.9 ms]
   [Eden: 1602.0M(1602.0M)->0.0B(1610.0M) Survivors: 36.0M->28.0M Heap: 2268.7M(4096.0M)->658.6M(4096.0M)]
 [Times: user=0.13 sys=0.00, real=0.04 secs]
 
linux机器上的日志格式如上，请写出获取 2025 2-18 15:32 ～ 2025 2-18 15:35 这段时间的liunx指令，
提示：需要获取的是给定时间范围内的，但是日志中可能没有指定时间。并且不是所有日志都有时间
```



```
2025-02-18T15:34:57.470+0800: 15742.165: [GC pause (G1 Evacuation Pause) (young), 0.0442481 secs]
   [Parallel Time: 40.5 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 15742166.2, Avg: 15742166.2, Max: 15742166.2, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 11.2, Avg: 11.6, Max: 12.0, Diff: 0.9, Sum: 46.4]
      [Update RS (ms): Min: 22.9, Avg: 23.0, Max: 23.0, Diff: 0.1, Sum: 91.8]
         [Processed Buffers: Min: 105, Avg: 116.2, Max: 123, Diff: 18, Sum: 465]
      [Scan RS (ms): Min: 0.3, Avg: 0.3, Max: 0.4, Diff: 0.1, Sum: 1.3]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [Object Copy (ms): Min: 4.9, Avg: 5.3, Max: 5.7, Diff: 0.8, Sum: 21.2]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 0.3]
      [GC Worker Total (ms): Min: 40.2, Avg: 40.3, Max: 40.4, Diff: 0.1, Sum: 161.1]
      [GC Worker End (ms): Min: 15742206.5, Avg: 15742206.5, Max: 15742206.5, Diff: 0.1]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.4 ms]
   [Other: 3.4 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.6 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.2 ms]
      [Humongous Reclaim: 0.1 ms]
      [Free CSet: 0.9 ms]
   [Eden: 1101.0M(1101.0M)->0.0B(1793.0M) Survivors: 3072.0K->7168.0K Heap: 2107.6M(4089.0M)->1007.6M(4091.0M)]
 [Times: user=0.16 sys=0.00, real=0.04 secs]
 
 linux机器上的日志格式如上，这是其中的一组，每一组的格式都是一样的，需求是获取 2025 2-18 15:32 ～ 2025 2-18 15:35的内容，内容需要对一组数据进行过滤只需要 2025-02-18T15:34:57.470+0800: 15742.165: [GC pause (G1 Evacuation Pause) (young), 0.0442481 secs]
和   [Eden: 1101.0M(1101.0M)->0.0B(1793.0M) Survivors: 3072.0K->7168.0K Heap: 2107.6M(4089.0M)->1007.6M(4091.0M)]
 这两种格式的日志
提示：需要获取的是给定时间范围内的，但是日志中可能没有指定时间。并且不是所有日志都有时间
要求写出linux指令 要求指令比较通用，并且简单易操作
```

```
grep -E '2025-02-18T15:3[2-5]' -A100 gc-2025-02-18_11-12-35.log | grep -E 'GC pause|Eden'
```

