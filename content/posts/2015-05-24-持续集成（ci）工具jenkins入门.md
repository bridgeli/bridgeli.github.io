---
title: 持续集成（C I）工具Jenkins入门
author: Bridge Li
type: post
date: 2015-05-24T14:24:39+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - 持续集成
tags:
  - CI
  - 持续集成
  - 自动化编译

---
这几天研究了一下持续集成（CI）工具Jenkins，感觉很强大，入门也很简单，今天就写一个入门的小例子，对于一般性的小项目足够用了，如果大家用到了更复杂的相信学会这篇文章之后，也一定能自己搞定了

1. 安装

Jenkins是Java开发的，我们只需要到他的官网：http://jenkins-ci.org下载一个war包，扔到servlet容器（例如：tomcat）中，启动就可以了，和普通的war启动没什么差别，如果你还不会，那么你需要补J2EE的基础了。

2. 安装插件

启动完成之后，是这样的，  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524172222-300x115.png" alt="20150524172222" width="300" height="115" class="alignnone size-medium wp-image-176" />][1]  
Jenkins自己已经帮我们安装了好多很好用的插件，但有些还是要我们自己装，例如：Git Client Plugin、Git Plugin，要安装这些，我们只需要点击左边的“系统管理” &#8211;> 管理插件 &#8211;> 可选插件，找到这两个插件装上就好了，装好插件下一步就是配置了

3. 配置

配置，同样“系统管理” &#8211;> 系统设置，下面开始配置：

①. Maven Configuration配置  
就是maven的settings文件的位置  
②. JDK配置  
新增JDK，然后大家一看就应该懂了，尽量不要自动安装  
③. Git  
也一样，就是Git的路径，需要注意的是：Git的路径一直要到bin下面的exe文件(Linux还没用过，所以不清楚，但我相信大家都懂得了)  
④. Maven配置  
这个和JDK的配置是一样的  
⑤. Maven项目配置  
Maven仓库的地址，我选的是第三个Local to the workspace，其实无所谓  
⑥. Jenkins URL  
就改成127.0.0.1吧，或者其他合适的，邮件地址就写自己的啦  
⑦. Git plugin  
这个相信不用多说了，Git我们都配过

这些目前是必须的配置，其他的大家就根据需要或者自己的理解，要么不配也行

需要说明的是，我没有配ant，用到了可以自己配一下，一样的，配置完成后选择“保存”就好了

4. 新建一个任务

这个大家都能找到在哪，然后进入到  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524175501-300x117.png" alt="20150524175501" width="300" height="117" class="alignnone size-medium wp-image-177" />][2]  
填入Item名称，选择构建一个maven项目，点击“OK”到下一页，需要说明的：  
①. 源码管理，这个也是最重要的，我选的是Git，这也是在一开始我安装那两个插件的原因，如果不安装那两个插件，是不会有Git这个选项的，然后键入要构建的项目的URL，点击“Add”，添加用户名和密码，切记一定要这么选择  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2015/05/20150518163202_jenkins-300x145.png" alt="20150518163202_jenkins" width="300" height="145" class="alignnone size-medium wp-image-174" />][3]  
其中Username是默认的，可以改，但Kind和Private Key强烈建议和我选的一样，否则可能会报代码拉不下来的错误，反正我是被这个错浪费了一天，然后这么选之后就对了，所以建议大家这么干，当然也可以自己选择其他的，大家可以试试。

②. 构建触发器  
Poll SCM、Build periodically这个是我们配置定时任务，也就是定时自动构建项目的，他们的区别：  
Poll SCM：定时检查源码变更（根据SCM软件的版本号），如果有更新就checkout最新code下来，然后执行构建动作。我的配置如下：

```  
\*/5 \* \* \* * （每5分钟检查一次源码变化）  
```

Build periodically：周期进行项目构建（它不care源码是否发生变化），我的配置如下：

```  
0 2 \* \* * （每天2:00 必须build一次源码）  
```

③. 其他的配置  
这些配置只有源码管理是必须的，其他的大家就可以根据需要自己进行配置了，如：Pre Steps和Post Steps，就是构建之前和构建之后做一些什么工作，例如构建之前我们需要删除一些文件，构建之后我们需要把构建后的文件copy到tomcat中，他们也支持shell编程的，其实很简单易懂，读者看一眼自己亲手操作一下就知道了，陆游说过，纸上得来终觉浅，绝知此事要躬行

5. 其他的配置  
①. 我们在点击“系统管理”之后，发现它提示我们：Your container doesn&#8217;t use UTF-8 to decode URLs. If you use non-ASCII characters as a job name etc, this will cause problems. See Containers and Tomcat i18n for more details.也就是说我们需要改变我们servlet的配置为UTF-8，因为老夫用的是tomcat的配置，所以就说一下tomcat怎么配的，其实很简单，只需要在tomcat中只要修改server.xml中如下就可以了：

```

<Connector port="8080" protocol="HTTP/1.1"  
connectionTimeout="20000"  
redirectPort="8443" URIEncoding="UTF-8" />

```

②. 安全配置  
我们现在的配置，是任何人都可以访问而且可以做任何事的，当然我们并不希望这样，那么我们可以点击“系统管理”，然后在右边有一个“安全设置”，点击它，进入到下面这个页面  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524183345-300x130.png" alt="20150524183345" width="300" height="130" class="alignnone size-medium wp-image-178" />][4]  
我们只需要和图中选的一样就可以了，当然大家也可以根据自己的需要，选择其他的

最后需要说明的是，如果我们的系统报“编码GBK的不可映射字符”，也就是如下这个错，  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2015/05/20150518175202-300x171.png" alt="20150518175202" width="300" height="171" class="alignnone size-medium wp-image-175" />][5]  
我们需要在我们的pom.xml文件中，增加

```

<properties>  
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
</properties>

```

就好了。

最后的最后需要说明的是，Jenkins其实很强大的，本文讲的只是一个最基本最基本的入门级教程，其他的还有很多没有写到，学完本文，老夫相信大家有了一定的自学能力，请大家自己去探索了，可以参考一个系列的文章：http://blog.csdn.net/wangmuming/article/category/2167947，细心的读者可能会发现这个博客本不是这些文章的原始作者，但老夫为什么还是要贴这个呢？因为作者全都注明了出处，所以希望我的读者如果有自己的博客的时候，如果转载他人的博客，也能做到这一点。

 [1]: https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524172222.png
 [2]: https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524175501.png
 [3]: https://www.bridgeli.cn/wp-content/uploads/2015/05/20150518163202_jenkins.png
 [4]: https://www.bridgeli.cn/wp-content/uploads/2015/05/20150524183345.png
 [5]: https://www.bridgeli.cn/wp-content/uploads/2015/05/20150518175202.png