# Java多版本管理,Maven单测

## Java 多版本管理
`sdkman`可安装多版本`JAVA`或者其他软件。[sdkman.io](https://sdkman.io/)  
`jEVN`是一个命令行工具让你免于设置`JAVA_HOME`配置。可以参考这篇文章[Configuring jenv the right way](https://developer.bring.com/blog/configuring-jenv-the-right-way/)
&lt;!-- more --&gt;
```bash
# 添加配置到.zshrc，如果使用bash的话需要添加到.bash_profile里面
echo &#39;export PATH=&#34;$HOME/.jenv/bin:$PATH&#34;&#39; &gt;&gt; ~/.zshrc
echo &#39;eval &#34;$(jenv init -)&#34;&#39; &gt;&gt; ~/.zshrc
# 添加jdk，这里以Mac OSX为例
jenv add /Library/Java/JavaVirtualMachines/jdk-10.0.1.jdk/Contents/Home
# 启用全局JAVA version
jenv global ${JAVA_VERSION}
# 设置SHELL指定的JAVA version
jenv shell ${JAVA_VERSION}
# 如果使用maven则还需要如下的配置
jenv enable-plugin maven
jenv enable-plugin export # 如果不开启的话 $JAVA_HOME 可能为空
```
## Java 单测时出现 `java.net.MalformedURLException: unknown protocol: socks5` 异常
在跑`Flink` `Kubernetes`模块的单测时遇到此异常，完整异常栈
```Java

io.fabric8.kubernetes.client.KubernetesClientException: Invalid proxy server configuration

	at io.fabric8.kubernetes.client.utils.HttpClientUtils.createHttpClient(HttpClientUtils.java:158)
	at io.fabric8.kubernetes.client.utils.HttpClientUtils.createHttpClientForMockServer(HttpClientUtils.java:66)
	at io.fabric8.kubernetes.client.server.mock.KubernetesMockServer.createClient(KubernetesMockServer.java:86)
	at org.apache.flink.kubernetes.MixedKubernetesServer.before(MixedKubernetesServer.java:64)
	at org.junit.rules.ExternalResource$1.evaluate(ExternalResource.java:46)
	at org.junit.rules.ExternalResource$1.evaluate(ExternalResource.java:48)
	at org.apache.flink.util.TestNameProvider$1.evaluate(TestNameProvider.java:45)
	at org.junit.rules.TestWatcher$1.evaluate(TestWatcher.java:55)
	at org.junit.rules.RunRules.evaluate(RunRules.java:20)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater$1.execute(IdeaTestRunner.java:38)
	at com.intellij.rt.execution.junit.TestsRepeater.repeat(TestsRepeater.java:11)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:35)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:235)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:54)
Caused by: java.net.MalformedURLException: unknown protocol: socks5
	at java.net.URL.&lt;init&gt;(URL.java:617)
	at java.net.URL.&lt;init&gt;(URL.java:507)
	at java.net.URL.&lt;init&gt;(URL.java:456)
	at io.fabric8.kubernetes.client.utils.HttpClientUtils.getProxyUrl(HttpClientUtils.java:221)
	at io.fabric8.kubernetes.client.utils.HttpClientUtils.createHttpClient(HttpClientUtils.java:144)
	... 24 more
```
此异常是由于设置的代理导致的，可以参考[Security Exception: MalformedURLException: unknown protocol: socket during opening JNLP file](https://stackoverflow.com/questions/17737564/security-exception-malformedurlexception-unknown-protocol-socket-during-openi)修改网络设置。但是只修改这里对于我的环境是没有生效的，因为在公司需要配置代理，设置了`HTTPS_PROXY=socket5://127.0.0.1:port`所以提示`unknown protocol: socks5`，需要在终端中执行`unset HTTPS_PROXY`，然后执行单测就可以了。

## Maven 单测使用
由于很少使用`maven`命令做测试，这次处理上面一个问题时候用到了，这里简单整理一下`maven test`相关的使用命令。
```bash
# 执行单个类的测试
maven test -Dtest=com.my.pkg.FarTest
# 执行单个类的指定方法测试
maven test -Dtest=com.my.pkg.FarTest#testFunc
# 跳过测试
maven clean package -DskipTests
# 跳过测试及测试编译
mvn package -Dmaven.test.skip=true
```


---

> Author:   
> URL: https://qinf.github.io/posts/java%E5%A4%9A%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86-maven%E5%8D%95%E6%B5%8B/  

