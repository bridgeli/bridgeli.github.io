---
title: Spring和websocket整合应用示例（下）
author: Bridge Li
type: post
date: 2016-04-04T14:14:24+00:00

duoshuo_thread_id:
  - 6.2697042124902E+18
categories:
  - Java
tags:
  - 长连接
---
在[上篇][1]中，我们已经实现了websocket，但还有一个核心的业务实现类没有实现，这里我们就实现这个业务核心类，因为老夫参与的这个系统使用websocket发送消息，所以其实现就是如何发送消息了。

7. NewsListenerImpl的实现

```

package cn.bridgeli.websocket;

import com.google.gson.Gson;  
import com.google.gson.GsonBuilder;  
import cn.bridgeli.DateUtil;  
import cn.bridgeli.enumeration.PlatNewsCategoryType;  
import cn.bridgeli.model.PlatNewsVo;  
import cn.bridgeli.model.SearchCondition;  
import cn.bridgeli.quartz.impl.TimingJob;  
import cn.bridgeli.service.PlatNewsService;  
import org.apache.commons.lang.StringUtils;  
import org.json.simple.JSONArray;  
import org.json.simple.JSONObject;  
import org.quartz.*;  
import org.quartz.impl.StdSchedulerFactory;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Component;  
import org.springframework.web.socket.TextMessage;

import java.io.IOException;  
import java.util.Date;  
import java.util.List;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;

/**  
* @Description : 站内消息监听器实现  
* @Date : 16-3-7  
*/  
@Component  
public class NewsListenerImpl implements NewsListener{  
private static final Logger logger = LoggerFactory.getLogger(NewsListenerImpl.class);  
Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd HH:mm:ss").create();

//线程池  
private ExecutorService executorService = Executors.newCachedThreadPool();

//任务调度  
private SchedulerFactory sf = new StdSchedulerFactory();

@Autowired  
private PlatNewsService platNewsService;

@Override  
public void afterPersist(PlatNewsVo platNewsVo) {  
logger.info("监听到有新消息添加。。。");  
logger.info("新消息为:"+gson.toJson(platNewsVo));  
//启动线程  
if(null != platNewsVo && !StringUtils.isBlank(platNewsVo.getCurrentoperatoremail())){  
//如果是定时消息  
if(platNewsVo.getNewsType() == PlatNewsCategoryType.TIMING_TIME.getCategoryId()){  
startTimingTask(platNewsVo); //定时推送  
}else{  
//立即推送  
executorService.execute(new AfterConnectionEstablishedTask(platNewsVo.getCurrentoperatoremail()));  
}  
}  
}

@Override  
public void afterConnectionEstablished(String email) {  
logger.info("建立websocket连接后推送新消息。。。");  
if(!StringUtils.isBlank(email)){  
executorService.execute(new AfterConnectionEstablishedTask(email));  
}  
}

/**  
* @Description ： 如果新添加了定时消息，启动定时消息任务  
* @param platNewsVo  
*/  
private void startTimingTask(PlatNewsVo platNewsVo){  
logger.info("开始定时推送消息任务。。。");

Date timingTime = platNewsVo.getTimingTime();  
if(null == timingTime){  
logger.info("定时消息时间为null。");  
return;  
}  
logger.info("定时推送任务时间为："+DateUtil.date2String(timingTime));

JobDetail jobDetail= JobBuilder.newJob(TimingJob.class)  
.withIdentity(platNewsVo.getCurrentoperatoremail()+"定时消息"+platNewsVo.getId(), "站内消息")  
.build();

//传递参数  
jobDetail.getJobDataMap().put("platNewsService",platNewsService);  
jobDetail.getJobDataMap().put("userEmail",platNewsVo.getCurrentoperatoremail());

Trigger trigger= TriggerBuilder  
.newTrigger()  
.withIdentity("定时消息触发"+platNewsVo.getId(), "站内消息")  
.startAt(timingTime)  
.withSchedule(SimpleScheduleBuilder.simpleSchedule()  
.withIntervalInSeconds(0) //时间间隔  
.withRepeatCount(0) //重复次数  
)  
.build();

//启动定时任务  
try {  
Scheduler sched = sf.getScheduler();  
sched.scheduleJob(jobDetail,trigger);  
if(!sched.isShutdown()){  
sched.start();  
}

} catch (SchedulerException e) {  
logger.info(e.toString());  
}  
logger.info("完成开启定时推送消息任务。。。");

}

/**  
* @Description : 建立websocket链接后的推送线程  
*/  
class AfterConnectionEstablishedTask implements Runnable{

String email ;  
public AfterConnectionEstablishedTask(String email){  
this.email = email;  
}  
@Override  
public void run() {  
logger.info("开始推送消息给用户:"+email+"。。。");

if(!StringUtils.isBlank(email)){  
SearchCondition searchCondition = new SearchCondition();  
searchCondition.setOperatorEmail(email);

JSONArray jsonArray = new JSONArray();

for(PlatNewsCategoryType type : PlatNewsCategoryType.values()){  
searchCondition.setTypeId(type.getCategoryId());  
int count = platNewsService.countPlatNewsByExample(searchCondition);  
JSONObject object = new JSONObject();  
object.put("name",type.name());  
object.put("description",type.getDescription());  
object.put("count",count);

jsonArray.add(object);  
}  
if(null != jsonArray && jsonArray.size()>0){  
UserSocketVo userSocketVo = WSSessionLocalCache.get(email);  
TextMessage reMessage = new TextMessage(gson.toJson(jsonArray));  
try {  
if(null != userSocketVo){  
//推送消息  
userSocketVo.getWebSocketSession().sendMessage(reMessage);  
//更新推送时间  
userSocketVo.setLastSendTime(DateUtil.getNowDate());  
logger.info("完成推送新消息给用户:"+userSocketVo.getUserEmail()+"。。。");  
}

} catch (IOException e) {  
logger.error(e.toString());  
logger.info("站内消息推送失败。。。"+e.toString());  
}  
}  
}  
logger.info("结束推送消息给"+email+"。。。");

}  
}  
}

```

这个类就是websocket的核心业务的实现，其具体肯定和业务相关，由于业务的不同，实现肯定不同，因为老夫参与的系统是发送消息，所以里面最核心的一句就是：

```

userSocketVo.getWebSocketSession().sendMessage(reMessage);

```

通过WebSocketSession的sendMessage方法把我们的消息发送出去。另外，这主要是后端的实现，至于前端的实现，因为老夫是后端程序猿比较关注后端，所以前端就不多做介绍了，大家可以自己去网上查资料。  
最后需要说明的是，老夫之前搜一些学习资料的时候，发现老夫该同事的写法和有一篇文章几乎一样，我想该同事应该是参考了这篇文章，所以列在下面，算作参考资料

参考资料：http://blog.csdn.net/heng_ji/article/details/39007227

 [1]: https://www.bridgeli.cn/archives/262 "Spring和websocket整合应用示例（上）"