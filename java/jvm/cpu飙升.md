现象 2025-03-06 appraisal-02机器突然间没有dubbo提供者，上层调用报错，机器显示cpu使用超过60%

# 一、确定案发时间 

2025-03-06 09:49 

![image-20250306103106765](https://raw.githubusercontent.com/Footman56/images/master/img202503061031132.png)

发现网络流量很大， 就是说接口返回的数据比较多，导致网络传输压力很大。

# 二、检查gc回收情况

gc回收情况和内存使用都很正常

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202503061032423.png" alt="image-20250306103248322" style="zoom:50%;" />

# 三、检查慢sql 

看是否有大量的接口数据返回



# 四、检查慢操作

发现一个14s请求的，根据traceId 查看执行情况

![image-20250306103431019](https://raw.githubusercontent.com/Footman56/images/master/img202503061034085.png)

否定啦，因为执行的时间不对，不在哪个区间内。



c65eb2448b1a4fd8b12288329a2b69ab：归档

899b37d20637402a93b7716e950f8b73：归档

c9cd1f87f31242d7a21fc03f29d8082e： 指标考评提交

899b37d20637402a93b7716e950f8b73： 复杂流程节点

