# 我的线上服务行为又不符合预期了

![我的服务行为又不符合预期了](/images/online-service-arthas/online-service-why.png)
你是不是遇到过这样的问题：自己的一个服务一直稳定运行的挺长时间了，突然它的行为就不符合预期了。然后你的同事来找你看看为啥行为变了呢？我最近也遇到了一个类似的问题，这里就简单记录一下自己排查这个问题的过程。
&lt;!--more--&gt;
## 服务描述
![线上服务](/images/online-service-arthas/online-service.png)
这里的用户通过Rest API 访问`service-1`的`function1`，`function1`通过Rest API访问`service-2`的`function2`，`function2`计算出结果之后返回给调用方。这里`service-2``function2`就是我上线挺长时间的一个功能。简化java class和方法的定义如下，另外说明一下这里`function1`实际里面有很多方法调用，这里全部简化掉只是为了说明相关的排查方法。
```Java
// function1
package com.eason.service1;
class Service1() {
  public List&lt;Result&gt; function1(String params);
}
  
// function2
package com.eason.service2;
class Service2() {
  public List&lt;Result&gt; function2(String params);
}
```
这里遇到的问题就是用户之前可以拿到结果，现在没有结果了。
## 问题排查
首先直接调用`service-2``function2`的Rest API发现其功能是正常的，那么排除`function2`是不可用的。
然后怀疑是`function1`调用`function2`的时候出现了问题。那么如何排查调用是真的有问题呢？如果有日志记录调用过程，那么排查会方便一些，但是不是所有的调用都会记录日志的，否则日志量可能太大了。
接下来就寄出了本次排查问题的利器：[Arthas](https://github.com/alibaba/arthas)。这里排查的基本思路就是中`function1`和`function2`上分别监控调用的入参、返回值和预期的值进行对比。下面就整理一些具体的步骤。
1. 当然首先要做的肯定是下载并解压`arthas-bin.zip`，可以去`Arthas`在`Github`[1]仓库进行下载。这里面包含多个jar和相关的脚本。具体使用可以参加其文档。

2. 我通常习惯使用`java -jar arthas-boot.jar`启动arthas server，这里需要：需要使用和服务相同的账户和`Java`版本。
3. 然后我们通过数字选择对应的服务，上面的命令会有类似`jps`的输出，每个`Java`服务都会列出来，选择我们的目标服务即可
4. 使用`watch`[2]命令来观察函数的入参、返回值
    - `watch com.eason.service1.Service1 function1 &#34;{params,target,returnObj}&#34; -x 4 -b -n 5`
        - `&#34;{parama,returnObj}&#34;`为观察表达式
        - `-x`指定输出结果属性的便利深度，就是将输出结果以可读的方式展示
        - `-b`在函数调用前观察，当然也有`-e`在函数调用之后观察
        - `-n`这里是执行几次
    - `watch com.eason.service1.Service2 function2 &#34;{params,target,returnObj}&#34; -x 4 -b -n 5`
5. 通过4的结果发现，`function1`调用`function2`的参数传的不符合预期，又对`function1`内部的其他相关的方法做了`watch`，最终定位到了问题。`

`Arthas`有非常多的功能，可以通过命令列表看到其支持的功能[3]，常用的有`watch`、`heapdump`、`profiler`、`trace`等。熟悉这些命令的基本使用非常有利于线上问题的排查。
当然线上服务的问题多种多样，`Arthas`只是排查问题的一个工具。如果服务是分布式的且有很多实例，那么文中这种排查方式处理起来就有些麻烦了，在多个实例上执行这套流程是比较耗时的，可能需要更都工具化的东西来支持。
## 参考
[1] https://github.com/alibaba/arthas/releases  
[2] https://arthas.aliyun.com/doc/watch.html  
[3] https://arthas.aliyun.com/doc/commands.html

---

> Author: [eason.qin](https://github.com/qinf)  
> URL: https://qinf.github.io/posts/729576f/  

