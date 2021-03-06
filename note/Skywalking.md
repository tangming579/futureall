参考：https://www.toutiao.com/i6884571378981274123/

APM(Application Performance Management) 

### 什么是链路跟踪

链路追踪是分布式系统下的一个概念，它的目的就是要解决将一次分布式请求还原成调用链路，并集中展示，比如，各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。

衡量一个接口，我们一般会看三个指标：

1. 接口的 RT（Route-Target）你怎么知道?
2. 接口是否有异常响应?
3. 接口请求慢在哪里?

对于单体架构，我们可以使用 AOP（切面编程）来统计这三个指标

OpenTracing 通过提供与平台和厂商无关的 API，使得开发人员能够方便地添加追踪系统，就像单体架构下的AOP（切面编程）一样。

OpenTracing 的数据模型，主要有以下三个：

- **Trace**：一个完整请求链路（树形结构）
- **Span**：一次调用过程（需要有开始时间和结束时间，同一层级parent id相同，span id不同，span id从小到大表示请求的顺序）
- **SpanContext**：Trace 的全局上下文信息，如里面有traceId

**总结：通过事先在日志中埋点，找出相同traceId的日志，再加上parent id和span id就可以将一条完整的请求调用链串联起来。**

采样：

由于每一个请求都会生成一个链路，为了减少性能消耗，避免存储资源的浪费，dapper并不会上报所有的span数据，而是使用采样的方式。举个例子，每秒有1000个请求访问系统，如果设置采样率为1/1000，那么只会上报一个请求到存储端。

### Java 探针

动态字节码技术 

Java 代码都是要被编译成字节码后才能放到 JVM 里执行的，而字节码一旦被加载到虚拟机中，就可以被解释执行。

字节码文件（.class）就是普通的二进制文件，它是通过 Java 编译器生成的。而只要是文件就可以被改变，如果我们用特定的规则解析了原有的字节码文件，对它进行修改或者干脆重新定义，就可以改变代码行为了。

Java 生态里有很多可以动态生成字节码的技术，像 BCEL、Javassist、ASM、CGLib 等

ASM 是它们中最强大的一个，使用它可以动态修改类、方法，甚至可以重新定义类，连 CGLib 底层都是用 ASM 实现的。

### .Net 探针

参考：https://www.cnblogs.com/wucy/p/13532534.html

[SkyAPM-dotnet](https://github.com/SkyAPM/SkyAPM-dotnet) 是SkyWalking在.Net Core端的探针实现，其主要的收集日志的手段就是基于DiagnosticSource来进行诊断跟踪的。

当我们通过ConfigureWebHostDefaults配置Web主机的时候，程序就已经默认给我们注入了诊断名称为Microsoft.AspNetCore的DiagnosticListener和DiagnosticSource