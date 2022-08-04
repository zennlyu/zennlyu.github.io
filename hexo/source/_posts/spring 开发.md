---
title: some spring dev bugs i ran into
categories: [java backend]
tags: [springboot]
---

## `Non-resolvable import POM`

```shell
mac@macdeMacBook-Pro WX-Fleas-Market-Demo % mvn clean package -Dmaven.test.skip=true
[INFO] Scanning for projects...
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] Non-resolvable import POM: org.springframework.cloud:spring-cloud-dependencies:pom:Greenwich.M1 was not found in https://maven.aliyun.com/repository/public during a previous attempt. This failure was cached in the local repository and resolution is not reattempted until the update interval of aliyunmaven has elapsed or updates are forced @ line 44, column 25
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project io.github.nnkwrik:fangxianyu:0.0.1-SNAPSHOT (/Users/mac/github/WX-Fleas-Market-Demo/pom.xml) has 1 error
[ERROR]     Non-resolvable import POM: org.springframework.cloud:spring-cloud-dependencies:pom:Greenwich.M1 was not found in https://maven.aliyun.com/repository/public during a previous attempt. This failure was cached in the local repository and resolution is not reattempted until the update interval of aliyunmaven has elapsed or updates are forced @ line 44, column 25 -> [Help 2]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
[ERROR] [Help 2] http://cwiki.apache.org/confluence/display/MAVEN/UnresolvableModelException
```

转到发现

```
<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
</dependencies>
```

参考了这篇[博客](https://blog.csdn.net/weixin_44259720/article/details/104531575)后，知道系统文件自动将 `${spring-cloud.version}` 读为 `Greenwich.M1` 版本。将其改成 `Edgware.SR3` 即可跑通。

开发代号看似没有什么规律，但实际上首字母是有顺序的，比如：Dalston版本，我们可以简称 D 版本，对应的 Edgware 版本我们可以简称 E 版本。

D、E版本：二者均基于SpringBoot的1.5.x版本，但支持其他组件的版本不同，如以 Dalston.SR4 和 Edgware.RELEASE 来对比：

pring-cloud-config 分别对应 1.3.3和 1.4.0；
spring-cloud-netflix 分别对应 1.3.5和 1.4.0；
spring-cloud-consul 分别对应 1.2.1和 1.3.0；
spring-cloud-gateway 前者不支持，后者 1.0.0。
F版本：F版本是个绝对的大版本，几乎所有组件，全部同步变更版本号为2.x；

SNAPSHOT： 小版本，快照版本，随时可能修改；

M： MileStone，小版本，M1表示第1个里程碑版本，一般同时标注PRE，表示预览版版。

SR： Service Release，小版本，SR1表示第1个正式版本，一般同时标注GA：(GenerallyAvailable),表示稳定版本。

## `jar missing problem`

```
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] 'dependencies.dependency.version' for org.springframework.cloud:spring-cloud-starter-gateway:jar is missing. @ line 29, column 21
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project io.github.nnkwrik:gateway:0.0.1-SNAPSHOT (/Users/mac/github/WX-Fleas-Market-Demo/gateway/pom.xml) has 1 error
[ERROR]     'dependencies.dependency.version' for org.springframework.cloud:spring-cloud-starter-gateway:jar is missing. @ line 29, column 21
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
```

子在pom中为他们指定版本号就好了。版本号可以在https://mvnrepository.com/ 中通过artifactId 查询

还有写项目的时候突然报了这个错误：`Error:java:java.lang.ExceptionInInitializerError: com.sun.tools.javac.code.TypeTags`

```
WARNING: All illegal access operations will be denied in a future release
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for fangxianyu 0.0.1-SNAPSHOT:
[INFO] 
[INFO] fangxianyu ......................................... SUCCESS [  5.236 s]
[INFO] common ............................................. FAILURE [ 55.328 s]
[INFO] inner-api .......................................... SKIPPED
[INFO] eureka ............................................. SKIPPED
[INFO] gateway ............................................ SKIPPED
[INFO] auth-service ....................................... SKIPPED
[INFO] user-service ....................................... SKIPPED
[INFO] goods-service ...................................... SKIPPED
[INFO] im-service ......................................... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:00 min
[INFO] Finished at: 2022-07-13T23:39:33+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.0:compile (default-compile) on project common: Fatal error compiling: java.lang.ExceptionInInitializerError: com.sun.tools.javac.code.TypeTags -> [Help 1]
然后当时就很懵逼，后来通过排错，发现是使用的lombok版本过低，我使用的springboot的版本是:: Spring Boot :: (v2.2.1.RELEASE)
```

## package org.springframework.cloud.openfeign does not exist

1. 没有添加版本号

2. 没有添加以下依赖

   ```
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
     <version>2.1.3.RELEASE</version>
   </dependency>
   ```

   