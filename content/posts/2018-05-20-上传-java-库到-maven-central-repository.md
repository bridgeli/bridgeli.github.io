---
title: 上传 Java 库到 Maven central repository
author: Bridge Li
type: post
date: 2018-05-20T08:17:54+00:00

categories:
  - Java
tags:
  - 401
  - 'Access denied to ReasonPhrase: Forbidden'
  - maven
  - Maven central repository

---
之前看过 Trinea 写过一篇文章，如果上传 Java 库到 Maven central repository，前一段时间感觉公司封装的 mybatis-generator 不好用，完全没有解决原生的 mybatis-generator 的问题，所以就重新做了一次封装，主要是加了查询分页，然后就想到是不是可以上传到 Maven central repository 玩玩，看了一下 Trinea 的这篇文章感觉挺简单的（原文见后面参考资料），但实际上还是有一些坑，具体的可以看 Trinea 的这篇文章，我主要写一下遇到的一些坑。

先说明一下，pom 文件请参考我的配置：https://github.com/bridgeli/mybatis-generator-plugin/blob/master/pom.xml，这里面所有的配置都是必须的，也是最少的了吧，自己写的时候修改一些内容，适配自己的就好了，Trinea 写的那个 pom 还是比较适合 Android。

发布 release 版本的时候，使用命令

```

mvn release:clean release:prepare release:perform

```

执行这个命令的时候，可能会遇到三个问题。

1. Git 报错，使用 Git 几年了，进入会遇到这样的错误。错误信息：

```

Unable to commit files  
[ERROR] Provider message:  
[ERROR] The git-add command failed.  
[ERROR] Command output:

```

实在不知道咋回事，可能是我有些文件没提交，反正提交之后就好了，看到有人说他报错是因为他不是 Git 管理的项目，我作为 Git 脑残粉，肯定是的，可能就是因为没有提交吧，反正最后就这么莫名其妙的解决了。

2. 提示没有 snapshot 版本，所以执行这个命令的时候，版本号和发布 snapshot 一样就好，而不是自己手动改成 release 版。  
3. 因为此时会发布一个 release 版本，会有 tag 推送到 GitHub 有可能会遇到 401，看到这个 error code，应该都清楚咋回事，没权限。不知道，病急乱投医，我是在 GitHub 上添加了一个 GPG key 并且在 pom 文件中新增一个 plugin。这个问题有人说是账号和密码错误，但是我很肯定我的账号和密码一点也没有错。

```

<plugin>  
<groupId>org.apache.maven.plugins</groupId>  
<artifactId>maven-gpg-plugin</artifactId>  
<version>1.5</version>  
<executions>  
<execution>  
<id>sign-artifacts</id>  
<phase>verify</phase>  
<goals>  
<goal>sign</goal>  
</goals>  
</execution>  
</executions>  
</plugin>

```

最终解决了这个问题，关于这个问题，可能是我操作的问题没有看到过别人说过，另外在 GitHub 上他的颜色一直还是：灰的，没有显示使用过，所以可能是我操作的有问题，不需要在 GitHub 上添加这个 GPG key，具体大家可以测一下，我后期也会测试。  
另，需要说明的是，生成 GPG key 的时候用到的邮箱和我在 GitHub 上的邮箱，我用的不是同一个邮箱，建议还是用同一个吧，省得出莫名其妙的问题。

上面问题都解决之后，可以发布 release 版本了，但是在此时我又遇到了三个问题，一步一坑啊。

4. 还是 401，依然没有权限，我的神啊，咋回事，说我没有权限上传到：https://oss.sonatype.org/service/local/staging/deploy/maven2 咋回事？咋回事？咋回事？因为在 https://issues.sonatype.org/ 上已经说我有权限了啊，实在解决不了了，试了各种方法都不行，当时也没有想到会是他们的问题，所以很郁闷，最终没有办法，然后就用我拙劣的蹩脚的英文，去询问一些咋回事，不多说了，见下面沟通截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/05/sonatype_chat-768x715.jpeg" alt="" width="768" height="715" class="alignnone size-medium wp-image-534" /> 

server-side error 被我遇到了，这是 https://issues.sonatype.org/ 啊，这错误想不到啊，为什么给我分权限的时候出错了，这运气也是没谁了，不过解决了就好。（不知道是 https://issues.sonatype.org/ 有 bug，还是咋回事，不知道大家会不会遇到这个问题，最起码 Trinea 的文章没有提到，反正帮 https://issues.sonatype.org/ 做了一把测试，哎）

5. 错误忘了截图了，反正就是不能调用 javadoc 命令生成 javadoc 文档，说我 JAVA_HOME 设置的有问题，这这这这莫名其妙的问题，我要是 JAVA_HOME 设置的有问题，javac 和 java 这俩命令可以用？但是没办法，虽然感觉有点莫名其妙，但是得解决，解决方法：maven-javadoc-plugin 新增如下配置：

```

<configuration>  
<javadocExecutable>/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/bin/javadoc</javadocExecutable>  
</configuration>

```

6. 正式发布，也就是同步到 Maven 主仓库使得其他人可以使用。包括 Close 和 Release 两步，先 Close 后 Release，Trinea 的图破了，我补上吧，close 截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/05/close-768x346.jpeg" alt="" width="768" height="346" class="alignnone size-medium wp-image-532" /> 

release 截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/05/release-768x360.jpeg" alt="" width="768" height="360" class="alignnone size-medium wp-image-533" /> 

但是此时，都最后一步了，胜利的曙光在望，我又又又又又又遇到了问题，Close 不了，为啥？还是看图吧

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/05/upload_error-768x509.jpeg" alt="" width="768" height="509" class="alignnone size-medium wp-image-535" /> 

少了两个签名文件，中央仓库要求必须有 class、sources、doc 和他们的签名等等（主要是为了安全，防止有人篡改），为什么 class 文件有 asc 文件，sources 和 doc 文件却没有，坑啊，百度了半天没解决，stack overflow 上搜了一下，第一条问答就解决了，很简单，修改一些配置，把 <phase>deploy</phase> 删了就好了。stack overflow 牛 X。  
其实这也暴漏了一些问题，对 Maven 的某些配置还是不熟啊，虽然用了好几年了。

最后希望大家都能把自己写的 Java 库其他的组件上传到 Maven central repository，拒绝做伸手党，造福我们程序员群体，丰富 Java 生态。

参考资料：  
Trinea 的文章：http://www.trinea.cn/dev-tools/upload-java-jar-or-android-aar-to-maven-center-repository/