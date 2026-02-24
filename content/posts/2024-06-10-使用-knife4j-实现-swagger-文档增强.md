---
title: 使用 knife4j 实现 Swagger 文档增强
author: Bridge Li
type: post
date: 2024-06-10T05:56:21+00:00

categories:
  - Java
tags:
  - knife4j
  - swagger

---
相信使用 Java 开发的人，对 Swagger 一定不会感到陌生，不过个人对 Swagger 一直没有太多好感，因为他的 UI 实在太难看了，用起来也颇为不顺手，所以国内有人开发了 knife4j 对 Swagger 进行增强，随着时间的推移，现在很多项目都在从 Java8 到 Java17，SpringBoot2 到 SpringBoot3 的迁移，发现 knife4j 现在也开始做了支持，而且用起来更方便。下面简单说一说如何使用。

1. 引入依赖

```

<dependency>  
<groupId>com.github.xiaoymin</groupId>  
<artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>  
<version>4.5.0</version>  
</dependency>

```

从中我们可以看到 artifactId 做了全新的修改，这个需要注意。另外 Spring Boot 3 只支持 OpenAPI3 规范。Knife4j提供的 starter 已经引用 springdoc-openapi 的 jar，大家需注意避免 jar 包冲突，引入之后，其余的配置，开发者即可完全参考 springdoc-openapi 的项目说明，Knife4j 只提供了增强部分，如果要启用 Knife4j 的增强功能，可以在配置文件中进行开启，其实个人测试就算完全不配置，此时也已经可以通过 http://ip:port/doc.html 查看文档：

```

knife4j:  
enable: true  
basic:  
enable: true  
username: BridgeLi  
password: BridgeLi  
springdoc:  
default-flat-param-object: true

```

最后，使用 OpenAPI3 的规范注解，注释各个 Spring 的 Rest 接口。

接口层：

```

import io.swagger.v3.oas.annotations.Operation;  
import io.swagger.v3.oas.annotations.tags.Tag;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.PostMapping;  
import org.springframework.web.bind.annotation.RequestBody;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;  
import java.util.List;

@Tag(name = "案件流程 ")  
@Slf4j  
@RestController  
@RequestMapping("/ajlc")  
public class AjlcController extends BaseController {

@Resource  
private AjlcService ajlcService;

@Operation(summary = "分页查询列表")  
@GetMapping("/queryAjlcsPage")  
public Result<TableDataInfo<Ajlc>> queryAjlcsPage(Ajlc ajlc) {  
startPage();  
List<Ajlc> ajlcs = ajlcService.queryAjlcs(ajlc);

return Result.success(getDataTable(ajlcs));  
}

}

```

实体类：

```

import io.swagger.v3.oas.annotations.media.Schema;  
import lombok.Data;

import java.io.Serializable;

/**  
* 案件流程  
* 表: ajlc 的 model 类  
*  
* @author BridgeLi  
* @date 2024-05-25 11:23:13  
*/  
@Schema(title = "案件流程")  
@Data  
public class Ajlc implements Serializable {  
/**  
* 类的 serial version id  
*/  
private static final long serialVersionUID = 1L;

/**  
* 字段: id，主键  
*/  
@Schema(title = "主键")  
private Integer id;

/**  
* 字段: ajlc_id，案件流程ID  
*/  
@Schema(title = "案件流程ID")  
private String ajlcId;  
}

```

最后本人也更新了 mybatis-generator-plugin 实现了对相关功能的支持，欢迎使用，具体见参考资料 2。

参考资料：  
1. https://doc.xiaominfo.com/docs/quick-start  
2. https://github.com/bridgeli/mybatis-generator-plugin