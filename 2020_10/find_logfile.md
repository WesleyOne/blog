---
sort: 89000
title: log日志输出文件定位
tags: 日志 log
---

# log日志输出文件定位



## 前言

​	这篇没有啥深奥的原理，单纯解决下新人查找日志输出文件的问题。深度研究者跳过。

​	为了更快搜索日志，往往在项目中将不同场景的日志输出到不同文件，这里面就涉及到程序中的`logger`对象和日志输出文件的关系。



## 来呀，快活呀

三步走：
1. 找当前logger的name
2. 匹配配置的logger（重点）
3. 查logger关联appender


### 找当前logger的name

#### 日志工厂类形式
```java
package com.logger;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoggerTest {
		// 对应logger的name是 com.logger.LoggerTest
    private static Logger logger = LoggerFactory.getLogger(LoggerTest.class);
    // 对应logger的name是 running
    private static Logger logger = LoggerFactory.getLogger("running");

    public void log() {
        logger.info("log use LoggerFactory.getLogger(Clazz)");
        logger.info("log use LoggerFactory.getLogger(String)");
    }
}
```

#### @Slf4j注解形式

```java
package com.logger;

import lombok.extern.slf4j.Slf4j;

// 对应logger的name是 com.logger.LoggerTest2
@Slf4j
public class LoggerTest2 {
  	// @Slf4j等同于如下代码
		// private  final Logger logger = LoggerFactory.getLogger(当前类.class)
  
    public void log() {
        log.info("log use @Slf4j");
    }
}
```



### 匹配配置的logger（重点）

​	以上文`com.logger.LoggerTest`为例，根据`.`分割产生的多个logger名称：

1. com.logger.LoggerTest
2. com.logger
3. com

   以日志框架的配置文件为例，按以上优先级顺序查找和`<logger>`标签里`name`属性值相同的第一个logger，如果都没有，则使用`<root>`标签配置。

> root可以理解为特殊的logger

### 查logger关联appender

​	在上一步匹配的`<logger>`或`<root>`标签里的可能存在0或多个`<appender-ref>`标签，`<appender-ref>`标签内部的`ref`属性值关联的`<appender>`标签，配置了实际输出文件的类型及格式。

​	另外如果`<logger>`标签的存在属性`additivity="true"`，则会继承父级`logger`的`appender`配置。如下所示：
​	
```xml
<logger name="com.logger.LoggerTest" additivity="true" level="debug" >
	<appender-ref ref="xxx">
</logger>
<logger name="com" additivity="true" level="debug">
	<appender-ref ref="xxx">
</logger>
<root>
	<appender-ref ref="xxx" level="debug">
</logger>
```
名为`com`的`logger`是名为`com.logger.LoggerTest`的`logger`的父级`logger`；
`root`是名为`com`的`logger`的父级`logger`；
所以当使用`com.logger.LoggerTest`的`logger`输出日志时，实质上会在三个`logger`里的所有`appender`输出。

> 如果发现日志还是没有输出到对应文件，请一定确认下logger的level级别，以及appender是否配置filter

## 参考

http://www.logback.cn/

http://logback.qos.ch/manual/index.html

https://arthas.aliyun.com/doc/logger.html (阿里巴巴开源Java诊断工具，可以在项目运行时修改logger.level哦)