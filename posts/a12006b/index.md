# Mocking 是一种反模式


这周看到来一篇文章[Mocking is an Anti-Pattern](https://www.amazingcto.com/mocking-is-an-antipattern-how-to-test-without-mocking/)，这里的观点引起了我的共鸣，工作中确实体会到过Mock带来的坏处或者说是不好的实践。
&lt;!--more--&gt;
![Mock是一种反模式](img/mock-is-an-anti-pattern/mock.png)
## 什么是Mock，什么是反模式
&gt; 在面向对象程序设计中，[**模拟对象**](https://zh.wikipedia.org/wiki/%E6%A8%A1%E6%8B%9F%E5%AF%B9%E8%B1%A1)（英语：mock object，也译作模仿对象）是以可控的方式模拟真实对象行为的假的对象。程序员通常创造模拟对象来测试其他对象的行为，很类似汽车设计者使用碰撞测试假人来模拟车辆碰撞中人的动态行为。

&gt; 在软件工程中，[反面模式](https://zh.wikipedia.org/wiki/%E5%8F%8D%E9%9D%A2%E6%A8%A1%E5%BC%8F)（anti-pattern或antipattern）指的是在实践中经常出现但又低效或是有待优化的设计模式[1][2]，是用来解决问题的带有共同性的不良方法。它们已经经过研究并分类，以防止日后重蹈覆辙，并能在研发尚未投产的系统时辨认出来。
&gt; 
在代码测试中，有些有副作用的逻辑无法直接调用的时候`Mock`就上场了。`Mock`模拟了代码里面的参数传递、函数调用以及我们预期结果的验证。一切好像极其的合理，代码的执行也跟我们假想的一模一样。
## Mock带来哪些坏处
但是就如[Mocking is an Anti-Pattern](https://www.amazingcto.com/mocking-is-an-antipattern-how-to-test-without-mocking/)这篇文章所说的“Mocking frameworks are a path to insanity!”，在实践中我们总是代码的`happy path`进行建模，虽然这是必须的但是无助于我们发现代码中的错误。    
过多的`Mock`导致了很多浪费体力、时间的无效工作。这让我想起了多年前我还在字节工作的时候的一次实践：为了提高代码测试覆盖率编写了大量的`Mock`，因为很多涉及到`IO`代码无法直接调用。  
`Mock` `IO`操作的可能会让忽略很多case，包括：边缘情况、故障等等。
## 除了Mock我们可以怎么做
那么除了`Mock`外还有哪些手段可以改进这种情况呢？文中提到了好几种方式。
### 做更多的单元测试
使用[Decompose Conditional](https://www.refactoring.com/catalog/decomposeConditional.html)和[Introduce parameter Object](https://refactoring.com/catalog/introduceParameterObject.html)等手段进行重构来方便做单元测试。
这两个都是《重构》这本书中提到的手段。
```Java
// Decompose Conditional
// before refactor
if (!aDate.isBefore(plan.summerStart) &amp;&amp; !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate &#43; plan.regularServiceCharge;

// after refactor
if (summer())
  charge = summerCharge();
else
  charge = regularCharge();


// Introduce Parameter Object
// before refactor
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}

// after refactor
function amountInvoiced(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```
### 更易于测试的IO
1. 使用内存数据库替代生产环境数据库进行测试。
2. 使用虚拟文件系统(in memory)模拟文件操作。这里需要考虑接口的设计不要使用文件名作为参数，而应该使用`Reader(golang)`或者`IOStream(Java)`这种抽象接口或者实现。
### 做IO测试
那么就用IO做测试吧，现在的IO设备已经很快了。这里提到了使用`Postgres`的复制测试数据库的功能可以方便进行测试。
### 逻辑和服务/IO分离
将业务逻辑和服务/IO分离开，这里业务逻辑就可以进行单独的测试了。对于IO骨架可以使用[`Command pattern`](https://en.wikipedia.org/wiki/Command_pattern)进行测试。
### E2E集成测试
基于UI的集成测试是有必要的，但是UI驱动测试比较脆弱，而且很不容易维护。所以可以考虑编写更多的后端E2E的集成测试。

原文里面提供了比较详细的例子来辅助说明各种方式，可以作为参考。  
`Mock`在日常的开发中还是比较经常使用的，比如`Flink`里面很多测试都单独编写了单独的`Mock Object`来方便进行测试，在日常的开发中我们可以进行参考。等之后积累的更多的案例可以再完善一下本文。

---

> Author: [eason.qin](https://github.com/qinf)  
> URL: https://qinf.github.io/posts/a12006b/  

