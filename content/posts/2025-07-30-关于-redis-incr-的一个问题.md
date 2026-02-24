---
title: 关于 Redis incr 的一个问题
author: Bridge Li
type: post
date: 2025-07-30T10:23:46+00:00

categories:
  - Java
tags:
  - redis
---
前一段时间有一个需求，需要计数，理所当地的使用了 redis 的 incr 方法。代码大概如下：

```

@Scheduled(cron = "0 0/10 \* \* * ?")  
public void test() {

long yellowInterval = 5L;  
boolean isReachable = false;  
// TODO  
long delta = isReachable ? -1L : 1L;  
ValueOperations<String, Long> valueOperations = redisTemplate.opsForValue();  
String key = RFID_NETWORK_STATUS_PREFIX + rfDevice.getId();  
Long increment = valueOperations.increment(key, delta);  
if (increment == null || increment <= 0L) {

}  
} else if (increment >= yellowInterval) {  
if (Constants.RFID_NETWORK_STATUS_GREEN.equals(rfDevice.getNetworkStatus())) {

}  
if (increment >= yellowInterval * 3) {  
valueOperations.set(key, yellowInterval * 3);  
}  
}

```

大概就是某个操作之后记一下数，加一或者减一，如果加到了某个值，就把它设置为某个值。我们这里先不考虑并发问题。结果在运行的时候遇到了一个问题，报错信息如下：

```

2025-07-22 14:51:13,069 [threadPoolTaskScheduler-12] ERROR o.s.s.s.TaskUtils$LoggingErrorHandler 95 &#8211; Unexpected error occurred in scheduled task  
org.springframework.data.redis.RedisSystemException: Error in execution  
at org.springframework.data.redis.connection.lettuce.LettuceExceptionConverter.convert(LettuceExceptionConverter.java:52)  
at org.springframework.data.redis.connection.lettuce.LettuceExceptionConverter.convert(LettuceExceptionConverter.java:50)  
at org.springframework.data.redis.connection.lettuce.LettuceExceptionConverter.convert(LettuceExceptionConverter.java:41)  
at org.springframework.data.redis.PassThroughExceptionTranslationStrategy.translate(PassThroughExceptionTranslationStrategy.java:40)  
at org.springframework.data.redis.FallbackExceptionTranslationStrategy.translate(FallbackExceptionTranslationStrategy.java:38)  
at org.springframework.data.redis.connection.lettuce.LettuceConnection.convertLettuceAccessException(LettuceConnection.java:256)  
at org.springframework.data.redis.connection.lettuce.LettuceConnection.await(LettuceConnection.java:969)  
at org.springframework.data.redis.connection.lettuce.LettuceConnection.lambda$doInvoke$4(LettuceConnection.java:826)  
at org.springframework.data.redis.connection.lettuce.LettuceInvoker$Synchronizer.invoke(LettuceInvoker.java:665)  
at org.springframework.data.redis.connection.lettuce.LettuceInvoker.just(LettuceInvoker.java:109)  
at org.springframework.data.redis.connection.lettuce.LettuceStringCommands.incrBy(LettuceStringCommands.java:176)  
at org.springframework.data.redis.connection.DefaultedRedisConnection.incrBy(DefaultedRedisConnection.java:382)  
at org.springframework.data.redis.core.DefaultValueOperations.lambda$increment$1(DefaultValueOperations.java:135)  
at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:406)  
at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:373)  
at org.springframework.data.redis.core.AbstractOperations.execute(AbstractOperations.java:97)  
at org.springframework.data.redis.core.DefaultValueOperations.increment(DefaultValueOperations.java:135)  
at cn.bridgeli.schedule.AmsScheduledTask.lambda$ping$1(AmsScheduledTask.java:154)  
at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)  
at cn.bridgeli.schedule.AmsScheduledTask.ping(AmsScheduledTask.java:135)  
at jdk.internal.reflect.GeneratedMethodAccessor249.invoke(Unknown Source)  
at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)  
at java.base/java.lang.reflect.Method.invoke(Method.java:568)  
at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84)  
at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)  
at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:96)  
at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:539)  
at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)  
at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)  
at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)  
at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)  
at java.base/java.lang.Thread.run(Thread.java:833)  
Caused by: io.lettuce.core.RedisCommandExecutionException: ERR value is not an integer or out of range  
at io.lettuce.core.internal.ExceptionFactory.createExecutionException(ExceptionFactory.java:147)  
at io.lettuce.core.internal.ExceptionFactory.createExecutionException(ExceptionFactory.java:116)  
at io.lettuce.core.protocol.AsyncCommand.completeResult(AsyncCommand.java:120)  
at io.lettuce.core.protocol.AsyncCommand.complete(AsyncCommand.java:111)  
at io.lettuce.core.protocol.CommandHandler.complete(CommandHandler.java:746)  
at io.lettuce.core.protocol.CommandHandler.decode(CommandHandler.java:681)  
at io.lettuce.core.protocol.CommandHandler.channelRead(CommandHandler.java:598)  
at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:442)  
at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:420)  
at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:412)  
at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1410)  
at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:440)  
at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:420)  
at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:919)  
at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:166)  
at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:788)  
at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:724)  
at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:650)  
at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:562)  
at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:997)  
at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)  
at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)  
&#8230; 1 common frames omitted

```

然后用工具查看了一下 Redis 中存储的数据是 15L，那似乎就很明显了，当 incr 返回的数据大于 15 是，往 Redis 中存储了一个 Long 类型的 15，也就是 15L，当下一轮循环的时候对 15L 进行加一或者减一，就报了上面的错，我们都知道 incr 的 value 必须是数字类型，15L 被当成了字符串，所以会报错。  
当我们知道是这个原因后其实就很简单了，最简单的一个方案就是：

```

int yellowInterval = 5L;  
ValueOperations<String, Integer> valueOperations = redisTemplate.opsForValue();

```

也就是将 yellowInterval 定义为 int 类型，而不是 Long 类型，这样在 incr 操作的时候就不会报错了。另外还有一个方案就是修改 Redis 的序列号方式，存储 Long 类型的数字时，不能把它序列化成字符串，而是要处理成数字类型也可以。