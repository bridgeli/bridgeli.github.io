---
title: Spring MVC 通过 SSE 实现消息推送
author: Bridge Li
type: post
date: 2025-02-27T01:26:08+00:00

categories:
  - Java
tags:
  - SSE
  - 消息推送

---
又好久没有写文章了，自从有了大模型之后写文章的态度越来越提不起兴趣了，有问题，直接问大模型即可。前几天公司有个需求，想用 SSE 实现，之前从没写过，所以让大模型直接写，然后实现超级简单：  
1. 编写 SSE 服务，来进行创建链接和发送消息

```

package cn.bridgeli.demo;

import lombok.Getter;  
import lombok.extern.slf4j.Slf4j;  
import org.apache.commons.collections4.CollectionUtils;  
import org.springframework.stereotype.Service;  
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.io.IOException;  
import java.util.List;  
import java.util.Map;  
import java.util.concurrent.ConcurrentHashMap;

@Slf4j  
@Getter  
@Service  
public class SseService {

private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

public SseEmitter stream(String usrId) {  
SseEmitter emitter = emitters.computeIfAbsent(usrId, k -> new SseEmitter(Long.MAX_VALUE));

emitter.onCompletion(() -> {  
log.info("SSE emitter completed");  
emitters.remove(usrId);  
});

emitter.onError((throwable) -> {  
log.error("Error occurred in SSE emitter", throwable);  
emitter.complete();  
emitters.remove(usrId);  
});

emitter.onTimeout(() -> {  
log.warn("SSE emitter timed out");  
emitter.complete();  
emitters.remove(usrId);  
});  
// 可选：连接成功时向客户端发送一个初始事件  
try {  
emitter.send(SseEmitter.event().name("connect").data("连接成功"));  
} catch (IOException e) {  
log.error("Error occurred while sending initial event", e);  
emitter.completeWithError(e);  
}

return emitter;  
}

public void send(List<String> userIds, String name, Object object) {  
if (!emitters.isEmpty()) {  
// 遍历所有用户的 SseEmitter，推送数据  
if (CollectionUtils.isEmpty(userIds)) {  
emitters.forEach((userId, emitter) -> {  
try {  
emitter.send(SseEmitter.event().name(name).data(object));  
} catch (IOException e) {  
// 如果发送失败，则移除该用户的 emitter  
log.error("Error occurred while sending event to user {}", userId, e);  
emitter.completeWithError(e);  
emitters.remove(userId);  
}  
});  
} else {  
userIds.forEach(userId -> {  
SseEmitter emitter = emitters.get(userId);  
if (emitter != null) {  
try {  
emitter.send(SseEmitter.event().name(name).data(object));  
} catch (IOException e) {  
// 如果发送失败，则移除该用户的 emitter  
log.error("Error occurred while sending event to user {}", userId, e);  
emitter.completeWithError(e);  
emitters.remove(userId);  
}  
}  
});  
}  
}  
}  
}

```

2. 编写对应的 Controller 给前端提供接口：

```

package cn.bridgeli.demo;

import cn.bridgeli.BaseAuthController;  
import io.swagger.v3.oas.annotations.tags.Tag;  
import jakarta.annotation.Resource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.http.MediaType;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

@Slf4j  
@RestController  
@Tag(name = "SSE 推送服务")  
@RequestMapping("/auth/common/sse")  
public class SseController extends BaseAuthController {

@Resource  
private SseService sseService;

@GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)  
public SseEmitter stream() {  
return sseService.stream(getLoginUsr().getUsrId());  
}  
}

```

3. 消息推送具体实现：

```

package cn.bridgeli.demo;

import cn.bridgeli.common.SseService;  
import cn.bridgeli.monitor.MonitorService;  
import cn.bridgeli.vo.CpuInfoVo;  
import jakarta.annotation.Resource;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.scheduling.annotation.Scheduled;  
import org.springframework.stereotype.Component;  
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.util.Map;

@Component  
@Slf4j  
public class ScheduledTask {

@Resource  
private MonitorService monitorService;  
@Resource  
private SseService sseService;

/**  
* 每分钟执行一次  
*/  
@Scheduled(cron = "0 0/1 \* \* * ?")  
public void updateOrderStatus() {  
log.info("=============定时任务=============");  
Map<String, SseEmitter> emitters = sseService.getEmitters();  
if (null == emitters || emitters.isEmpty()) {  
log.info("sse emitters is empty");  
return;  
}  
CpuInfoVo cpuData = monitorService.getCpuData();  
sseService.send(null, "cpu", cpuData);  
}

}

```

其实就是前端连接之后创建一个连接，保存连接，然后别的地方产生消息，推送消息，我的实例是通过 oshi 获取 CPU 使用率，实现对 CPU 的实时监控。