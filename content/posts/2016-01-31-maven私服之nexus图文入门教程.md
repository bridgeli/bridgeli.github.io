---
title: Maven私服之Nexus入门图文教程
author: Bridge Li
type: post
date: 2016-01-31T14:03:25+00:00

duoshuo_thread_id:
  - 6.2459519313075E+18
categories:
  - Maven
tags:
  - maven
  - nexus

---
老夫相信看到这篇文章的人一定已经知道maven和nexus分别是什么东西了，所以就不多做介绍了，下面直接从下载安装开始讲。

1. 下载安装

大家可以直接从这个链接<a href="http://www.sonatype.org/nexus/go/" target="_blank" title="nexus下载地址">http://www.sonatype.org/nexus/go/</a>下载系统，下载完成之后解压到系统的任何文件夹下就可以了，老夫下载是：nexus-2.8.1-01，然后可就是安装了。  
解压一路进到nexus-2.8.1-01/bin/jsw。然后选择适合自己的系统的文件夹进去，老夫的电脑是win32，所以进去之后是这个样子

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/1-300x87.png" alt="1" width="300" height="87" class="alignnone size-medium wp-image-242" />][1]

然后用管理员身份先运行install-nexus.bat安装服务，然后运行console-nexus.bat，等起来之后直接在浏览器中输入：http://localhost:8081/nexus，我们看到下图就是成功了，忘了是不是需要安装服务：wrapper了，大家试一试就知道了。

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/2-300x65.png" alt="2" width="300" height="65" class="alignnone size-medium wp-image-243" />][2]

2. 配置nexus

要管理Nexus，你首先需要以管理员身份登陆，点击界面右上角的login，输入默认的登录名和密码：admin/admin123，登陆成功后，你会看到左边的导航栏增加了很多内容，变成了下面这个样子

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/3-300x92.png" alt="3" width="300" height="92" class="alignnone size-medium wp-image-244" />][3]

这里，可以管理仓库，配置Nexus系统，管理任务，管理用户，角色，权限，查看系统的RSS源，管理及查看系统日志，等等。你会看到Nexus的功能十分丰富和强大，本文，老夫只介绍一些最基本的管理和操作。

因为是老夫习惯于让大伙看了这篇文章就知道怎么做，所以这里多内容咱就不说了，直接上配置，点击左边Administration菜单下面的Repositories，找到右边仓库列表中的三个仓库Apache Snapshots，Codehaus Snapshots和Maven Central，然后再没有仓库的configuration下把Download Remote Indexes修改为true。如下图：

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/4-300x99.png" alt="4" width="300" height="99" class="alignnone size-medium wp-image-245" />][4]

然后在Apache Snapshots，Codehaus Snapshots和Maven Central这三个仓库上分别右键，选择Repari Index，这样Nexus就会去下载远程的索引文件。

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/5-300x113.png" alt="5" width="300" height="113" class="alignnone size-medium wp-image-246" />][5]

这样设置以后, Nexus会自动从远程中央仓库下载索引文件, 为了检验索引文件自动下载是否生效,可以切换到Browse Index看看是否生成了这些索引文件。

3. 添加一个代理仓库

加入我们想要代理Sonatype的公共仓库，其地址为：http://repository.sonatype.org/content/groups/public/。步骤如下，在Repositories面板的上方，点击Add，然后选择Proxy Repository，在下方的配置部分，我们填写如下的信息：Repository ID &#8211; sonatype；Repository Name &#8211; Sonatype Repository；Remote Storage Location &#8211; http://repository.sonatype.org/content/groups/public/。其余的保持默认值，需要注意的是Repository Policy，我们不要代理snapshot构件，至于原因老夫相信读者应该已经很清楚了，然后点击Save。因为比较简单就不上图了。

4. 管理宿主仓库

Nexus预定义了3个本地仓库，分别为Releases，Snapshots，和3rd Party。这三个仓库都有各自明确的目的。Releases用于部署我们自己的release构件，Snapshots用于部署我们自己的snapshot构件，而3rd Party用于部署第三方构件，有些构件如Oracle的JDBC驱动，我们不能从公共仓库下载到，我们就需要将其部署到自己的仓库中。  
当然你也可以创建自己的本地仓库，步骤和创建代理仓库类似，点击Repository面板上方的Add按钮，然后选择Hosted Repository，然后在下方的配置面板中输入id和name，注意这里我们不再需要填写远程仓库地址，Repository Type则为不可修改的hosted，而关于Repository Policy，你可以根据自己的需要选择Release或者Snapshot。

5. 管理Maven仓库组

Nexus 中仓库组的概念是Maven没有的，在Maven看来，不管你是hosted也好，proxy也好，或者group也好，对我都是一样的，我只管根据 groupId，artifactId，version等信息向你要构件。为了方便Maven的配置，Nexus能够将多个仓库，hosted或者 proxy合并成一个group，这样，Maven只需要依赖于一个group，便能使用所有该group包含的仓库的内容。  
最新neuxs默认自带了一个名为“Public Repositories”组，点击该组可以对他保护的仓库进行调整，把刚才建立的仓库Sonatype Repository加入其中，这样就不需要再在maven中明确指定仓库的地址了。同时创建一个Group ID为public-snapshots、Group Name为Public Snapshots Repositories的组，把Apache Snapshots、Codehaus Snapshots、Snapshots和Sonatype Repository也加入其中。

6. 配置Maven使用Nexus

默认情况下，Maven依赖于中央仓库，这是为了能让Maven开箱即用，但仅仅这么做明显是错误的，这不仅会造成大量的时间及带宽的浪费，而且也不能使用公司自己的jar文件。既然本文的前面已经介绍了如何安装和配置Nexus，现在我们就要配置Maven来使用本地的Nexus来解决这个问题。  
其实我们可以将Repository配置到POM中，但一般来说这不是很好的做法，原因很简单，你需要为所有的Maven项目重复该配置。因此，这里我们将Repository的配置放到$user_home/.m2/settings.xml中，放一个老夫在公司里面的完整的settings.xml的完整文件:

```

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0  
http://maven.apache.org/xsd/settings-1.0.0.xsd">  
<!&#8211;  
<localRepository>E:maven_repository</localRepository>  
&#8211;>  
<localRepository>D:/J2EE/repo</localRepository>  
<!&#8211;非官方插件命令行运行配置&#8211;>  
<pluginGroups>  
<pluginGroup>org.mortbay.jetty</pluginGroup>  
</pluginGroups>  
<servers>  
<server>  
<id>releases</id>  
<username>XXX</username>  
<password>XXXXXX</password>  
</server>  
<server>  
<id>snapshots</id>  
<username>XXX</username>  
<password>XXXXXX</password>  
</server>  
<server>  
<id>nexus</id>  
<username>XXX</username>  
<password>XXXXXX</password>  
</server>  
</servers>

<mirrors>  
<!&#8211;配置仓库镜像&#8211;>  
<mirror>  
<id>nexus</id>  
<mirrorOf>*</mirrorOf>  
<name>Human Readable Name for this Mirror.</name>  
<url>http://nexus.lagou.com/content/groups/public/</url>  
</mirror>

</mirrors>

<profiles>  
<!&#8211;配置仓库和插件仓库&#8211;>  
<profile>  
<id>nexus</id>  
<repositories>  
<repository>  
<id>central</id>  
<name>central</name>  
<url>http://central</url>  
<releases>  
<enabled>true</enabled>  
</releases>  
<snapshots>  
<enabled>true</enabled>  
</snapshots>  
</repository>  
</repositories>

<pluginRepositories>  
<pluginRepository>  
<id>central</id>  
<name>central</name>  
<url>http://central</url>  
<releases>  
<enabled>true</enabled>  
</releases>  
<snapshots>  
<enabled>true</enabled>  
</snapshots>  
</pluginRepository>  
</pluginRepositories>  
</profile>  
</profiles> 

<!&#8211;激活profile&#8211;>  
<activeProfiles>  
<activeProfile>nexus</activeProfile>  
</activeProfiles>

</settings>

```

其实如果只需要下载的话，settings.xml中server是没有必要配置的，至于为什么配置了，下文会作出说明。

7. 部署构件至Nexus

Nexus提供了两种方式来部署构件，你可以从UI直接上传，也可以配置Maven部署构件。

①. 通过Nexus UI部署

有时候有个jar文件你无法从公共Maven仓库找到，但是你能从其它得到这个jar文件（甚至是POM），那么你完全可以将这个文件部署到Nexus中，使其成为标准流程的一部分。步骤如下：  
点击左边导航栏的&#8221;Repository&#8221;，在右边的仓库列表中选择一个仓库，如“3rd Party”，然后会看到页面下方有7个tab，选择最后一个“Artifact Upload”，你会看到构件上传界面。选择你要上传的构件，并指定POM，（或者手工编写GAV等信息），之后选择add artifact，最后点击Upload Artifact，该构件就直接被部署到了Nexus的&#8221;3rd Party&#8221;仓库中，具体就不多说了，大家可以自己操作几次就明白了，可以参考下图（由于图片所限，最后这个button截图没有截出来）。

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/01/6-300x115.png" alt="6" width="300" height="115" class="alignnone size-medium wp-image-247" />][6]

②. 通过Maven部署

更常见的用例是：团队在开发一个项目的各个模块，为了让自己开发的模块能够快速让其他人使用，你会想要将snapshot版本的构件部署到Maven仓库中，其他人只需要在POM添加一个对于你开发模块的依赖，就能随时拿到最新的snapshot。  
以下的pom.xml配置就能让你通过Maven自动化部署构件：

```

<project>  
&#8230;  
<distributionManagement>  
<repository>  
<id>releases</id>  
<name>Nexus Release Repository</name>  
<url>http://127.0.0.1:8080/nexus/content/repositories/releases/</url>  
</repository>  
<snapshotRepository>  
<id>snapshots</id>  
<name>Nexus Snapshot Repository</name>  
<url>http://127.0.0.1:8080/nexus/content/repositories/snapshots/</url>  
</snapshotRepository>  
</distributionManagement>  
&#8230;  
</project>

```

这里我们配置所有的snapshot版本构件部署到Nexus的Snapshots仓库中，所有的release构件部署到Nexus的Releases仓库中。但是这么配置还没有完，因为部署文件到系统里面肯定需要权限验证，那么此时我们我们在settings.xml中配置的service就派上用场了，因为我们配置了对应Repository id的用户名和密码。  
然后，在项目目录中执行mvn deploy ，你会看到maven将项目构件部署到Nexus中，浏览Nexus对应的仓库，就可以看到刚才部署的构件。当其他人构建其项目时，Maven就会从Nexus寻找依赖并下载。

8. Maven常用插件

在实际使用中，maven有非常多的使用查看，老夫之前的blog曾介绍过一个[Maven项目如何生成测试报告][7]，今天在介绍一个打包的时候用到的plugin：maven-source-plugin，他的作用是，在上传到Nexus时，会同步上传一个-sorce.jar的源码包，配置如下：

```

<plugin>  
<artifactId>maven-source-plugin</artifactId>  
<executions>  
<execution>  
<id>attach-sources</id>  
<phase>deploy</phase>  
<goals>  
<goal>jar-no-fork</goal>  
</goals>  
</execution>  
</executions>  
</plugin>  
<plugin>

```

最后，其实Nexus还有很多其它的特性，如用户管理，角色权限管理等等，但因为本文仅仅是一个入门教程，所以就不对其做过多讲解了，用户可以在自己的使用中慢慢去了解探索，另外Nexus的OSS版本是完全开源的，如果你有兴趣，你可以学习其源码，甚至自己实现一个REST客户端。  
马上拥抱Nexus吧，它是免费的！

参考资料：  
1. http://juvenshun.iteye.com/blog/349534  
2. http://blog.csdn.net/fanyuna/article/details/40145827

 [1]: https://www.bridgeli.cn/wp-content/uploads/2016/01/1.png
 [2]: https://www.bridgeli.cn/wp-content/uploads/2016/01/2.png
 [3]: https://www.bridgeli.cn/wp-content/uploads/2016/01/3.png
 [4]: https://www.bridgeli.cn/wp-content/uploads/2016/01/4.png
 [5]: https://www.bridgeli.cn/wp-content/uploads/2016/01/5.png
 [6]: https://www.bridgeli.cn/wp-content/uploads/2016/01/6.png
 [7]: https://www.bridgeli.cn/archives/160 "Maven项目如何生成测试报告"