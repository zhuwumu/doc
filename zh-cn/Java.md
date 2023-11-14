# 线程池

## 设置线程池参数

> 线程池参数设置https://blog.51cto.com/u_16099170/6771992

**具体是怎么设置呢？**

假设机器有N个CPU

1.那么对于计算密集型的任务，corePoolSize 应该设置线程数为N+1

2.对于IO密集型的任务，corePoolSize 应该设置线程数为2N

3.对于同时又计算工作和IO工作的任务，应该考虑使用两个线程池，一个处理计算任务，一个处理IO任务，分别对两个线程池按照计算密集型和IO密集型来设置线程数

> 获取cpu代码

```java
int i = Runtime.getRuntime().availableProcessors();
System.out.println(i);
```



## 创建线程池

> https://www.cnblogs.com/badaoliumangqizhi/p/17304186.html

ThreadPoolTaskExecutor （属于spring）和ThreadPoolExecutor（属于JDK）的区别：https://blog.csdn.net/qq_44754515/article/details/125805766

> Java线程池中三种方式创建 ThreadFactory 设置线程名称
>
> https://cloud.tencent.com/developer/article/1948703?areaSource=102001.16&traceId=5aZi_fV1b9JskW2w7c9er

> https://www.cnblogs.com/LiPengFeiii/p/15766351.html



使用common下创建ThreadFactory时，设置`daemon=true`变为守护线程，当主程序结束后，线程也会结束。如果要查看打印，需要给主线程加一下Thread.sleep(5000); // 主线程休眠5秒

```jav
ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build();
```

示例

```java
package com.cetc.gatekeeper.config;

import org.apache.commons.lang3.concurrent.BasicThreadFactory;
import org.springframework.stereotype.Component;

import java.util.concurrent.*;

/**
 * @auther lanmei
 * @date 2023/10/17
 */
public class ThreadPoolConfig {

    /** 线程池核心池的大小 */
    private static int corePoolSize = 2;

    /** 线程池中允许的最大线程数量 */
    private static int maximumPoolSize = 2;

    /** 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间 默认秒s*/
    private static long keepAliveTime = 60L;

    /** 队列 */
    private static int capacity = 1000;

    private static ThreadFactory threadFactory = new BasicThreadFactory.Builder()
            .namingPattern("data-send-pool-%d").daemon(true).build();

    private static ExecutorService executorService = new ThreadPoolExecutor(
            corePoolSize,
            maximumPoolSize,
            keepAliveTime,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(capacity),
            threadFactory,
            new ThreadPoolExecutor.CallerRunsPolicy()
    );


    /**
     * 获取线程池service
     * @return
     */
    public static ExecutorService getExecutorService() {
        return executorService;
    }
}

```





# UDP

> 关于netty UDP不能发送大于2048字节包的问题https://blog.csdn.net/KokJuis/article/details/72864018

* udp理论上支持最大发送64K的包，那为什么netty udp不能发送大于2048字节呢？实际上网络通信中，还受到很多其他因素的影响，netty udp并非不能发送大于2048字节的包。而是收到MTU的影响。MTU【最大传输单元（Maximum Transmission Unit，MTU）是指一种通信协议的某一层上面所能通过的最大数据包大小（以字节为单位）。最大传输单元这个参数通常与通信接口有关（网络接口卡、串口等）】。

* MTU国际默认规定是1500，不同的网络接入方式，不同地区的网络运营商，甚至不同的路由器，它们的MTU值都可能是不同的，例如：ADSL接入时MTU为1492字节。操作系统中可以通过命令查看：netsh interface ipv4 show subinterfaces
* 最后说一下netty udp每次发送包大小的建议，因为国内很多用户的上网方式都是ADSL。而ADSL的MTU值1492。但每个UDP包都包含28字节的“数据包报头”，所以实际你每次只能发送的数据是1464个字节。



# NIO

## ByteBuffer

![1699669677755](../../doc\static\images\Java\1699669677755.png)



## channel

* FileChannel 的读和写都复制了2次
* DatagramChannel
* SocketChannel
* ServerSocketChannel



## selector

![1699671228517](../../doc\static\images\Java\1699671228517.png)

 为了实现Selector管理多个SocketChannel，必须将具体的SocketChannel对象注册到Selector，并声明需要监听的事件（这样Selector才知道需要记录什么数据），一共有4种事件： 

* connetc：客户端连接服务器事件， 对应值为SelectionKey.OPCONNECT(8)  
* accept：服务端接收客户端连接事件，对应值为SelectionKey.OPACCEPT(16)  
* read：读事件 ，对应值为SelectionKey.OPREAD(1) 
* write：写事件，对应值为SelectionKey.OPWRITE(4) 







# Netty

> 参数调优

https://blog.csdn.net/weixin_44680802/article/details/128462846

针对ScoketChannel，7个，通过.childOption设置，常用的两个如下：

* SO_KEEPALIVE，tcp层keepalvie，默认关闭，一般选择关闭tcp keepalive 而使用应keepalive
* TCP_NODELAY：设置是否启用nagle算法，该算法是tcp在发送数据时将小的、碎片化的数据拼接成一个大的报文一起发送，以此来提高效率，默认是false（启用），如果启用可能会导致有些数据有延时，如果业务不能忍受，小报文也需要立即发送则可以禁用该算法

针对ServerScoketChannel，3个，通过.Option设置，常用的一个如下：

* .option(ChannelOption.SO_BACKLOG, 1024) // 等待最大连接数量
* .childOption(ChannelOption.SO_KEEPALIVE, true) // 服务端开启tcp keepalive
* .childOption(ChannelOption.TCP_NODELAY, true) // 关闭nagle算法，tcp发送小数据时，直接发送，不再拼装，减少延迟



>  出现io.netty.util.IllegalReferenceCountException: refCnt: 0, decrement: 1的原因及解决办法 http://www.manongjc.com/detail/21-cvkttljmlbmamsz.html

```java 
msg.release();
```

从以上的分析文章中发现，SimpleChannelInboundHandler会自动释放内存（虽然这是一种软释放）即是refCnt引用数减一。

而本人在使用SimpleChannelInboundHandler作为Server端的时候，自己手动释放了一次msg的内存，导致refCnt引用数为0，这个时候框架试图去释放一

次，就报如上错误。释放代码如：

当然前提是你使用了SimpleChannelInboundHandler作为Handler处理事务，使用AbstractChannelInboundHandler是不会主动释放内容的，这个时候需要你自己手动释放一次。



> SimpleChannelInboundHandler和ChannelInboundHandlerAdapter区别https://www.pianshen.com/article/46501669845/

* SimpleChannelInboundHandler继承ChannelInboundHandlerAdapter
* SimpleChannelInboundHandler自动释放内存，ChannelInboundHandlerAdapter不自动释放
* 在Netty中客户端的Handler一般继承SimpleChannelInboundHandler抽象类，服务端Handler一般继承ChannelInboundHandlerAdapter抽象类
* 在客户端，当 channelRead0() 方法完成时，你已经有了传入消息，并且已经处理完它了。当该方法返回时，SimpleChannelInboundHandler负责释放指向保存该消息的ByteBuf的内存引用。
*  服务端：在EchoServerHandler中，你仍然需要将传入消息回送给发送者，而 write() 操作是异步的，直到 channelRead() 方法返回后可能仍然没有完成。为此，EchoServerHandler扩展了 ChannelInboundHandlerAdapter ，其在这个时间点上不会释放消息。 



> 示例https://mp.weixin.qq.com/s/RPTETiULRAkOS-ZTd6xM2A





# 复制对象

> https://blog.csdn.net/JokerLJG/article/details/122876890?spm=1001.2014.3001.5506

* 直接赋值
* 浅拷贝 Spring BeanUtils(不要使用apache的)、 MapStruct ，不推荐使用json序列化的方式
* 深拷贝 get|set、clone



# Jackson

> https://zhuanlan.zhihu.com/p/646744855