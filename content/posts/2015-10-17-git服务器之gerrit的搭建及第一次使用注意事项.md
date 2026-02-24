---
title: Git服务器之Gerrit的搭建及第一次使用注意事项
author: Bridge Li
type: post
date: 2015-10-17T09:03:35+00:00

duoshuo_thread_id:
  - 6.206539630361E+18
categories:
  - SCM
tags:
  - code review
  - Gerrit
  - Git
  - 代码审查

---
公司的代码托管打算由SVN迁移到Git，刚好老大让老夫全权负责这个事(感谢老大信任)，老夫根据自己使用Git的经验，选择了Gerrit作为服务器，下面介绍一下老夫搭建Gerrit服务器的过程及第一次使用时需要注意的事项，如果以前没有用过Git可以参考老夫之前写的[这篇文章][1]和[这篇文章][2]。

1. 环境准备

①. Linux，Gerrit需要Linux环境，至于是哪个发行版本就不重要了，ubuntu还是centos随意；  
②. JDK，这个怎么安装就不说了，Java程序猿都会，就是不会网上一搜一堆，不做赘述；  
③. MySQL，其实这个非必须，Gerrit自带的有H2数据库，但没法老夫就是喜欢MySQL；  
④. nginx，作为认证和反向代理服务器；  
⑤. Maven, 在安装的过程中会下载一些jar文件；  
⑥. Git，这个忘了是不是必须的了，但还是装上吧，反正也不多，大家可以自己试一下需不需要（欢迎留言指出）；

2. 数据准备

①. 自己去Gerrit的官网：http://gerrit-releases.storage.googleapis.com/index.html，下载一个合适的版本，就是一个war包，这也就是我们为什么需要先装JDK的原因，需要说明的是Gerrit的1.X版本是Python写的，2.X改成了Java，变成了一个war包；  
②. 为Gerrit创建一个数据库，库名您随意，最好有意义，例如就叫reviewdb

3. 开始安装

开始安装之前，有人建议专门为Gerrit创建一个用户用户运行Gerrit，并且禁止登陆，这里从简，如果你非要这么做可以这么做：

```

adduser gerrit2  
passwd &#8211;delete gerrit2

```

然后以gerrit2运行安装，老夫就从简了，直接敲一下命令：

```

java -jar gerrit-xxx.war init -d review

```

gerrit-xxx.war就是你之前下载的war包，review是一个路径，就是安装到这下面，你可以随意定义，敲完这个命令，就进入一下交互(你的可能不太一样)，在里面我加了一些注释：

```

Gerrit Code Review xxx  
选项中大写字母为默认选项,如使用默认选项回车即可

Create &#8216;/home/review&#8217; [Y/n]? 

Git Repositories  
gerrit用于存储git仓库的目录,相对于根目录review #就是之前-d后面的路径

Location of Git repositories [git]: 

SQL Database

Database server type [h2]: mysql #数据库选mysql  
Server hostname [localhost]:  
Server port [(mysql default)]:  
Database name [reviewdb]:  
Database username [root]:  
root&#8217;s password :  
confirm password : 

User Authentication  
使用HTTP认证,OPENID需要服务器连接互联网,还可以使用LDAP认证服务

Authentication method [OPENID/?]: http #这里建议选http  
Get username from custom HTTP header [y/N]?  
SSO logout URL : 

Email Delivery  
gerrit发送邮件设置,可以使用本地或远程SMTP服务器,  
只要在smtp服务器上有帐号即可。

SMTP server hostname [localhost]: #这里我并没有选择邮件发送服务器和其他的配置，不知道为什么也可以发邮件  
SMTP server port [(default)]: #事实上发邮件是必须的，有知道为什么，可以告知，谢谢  
SMTP encryption [NONE/?]:  
SMTP username [root]:  
root@localhost.localdomain&#8217;s password :  
confirm password : 

Container Process  
使用root用户运行gerrit

Run as [root]:  
Java runtime [/usr/lib/jvm/java-7-openjdk-amd64/jre]:  
Copy gerrit-2.8.1.war to /home/review/bin/gerrit-xxx.war [Y/n]?  
Copying gerrit-2.8.1.war to /home/review/bin/gerrit-xxx.war

SSH Daemon  
gerrit自带的ssh服务,与服务器自身的ssh服务无关,监听默认端口即可  
注意:如要使用低于1024的特权端口,需authbind授权,否则ssh会绑定端口失败  
Listen on address [*]:  
Listen on port [29418]: 

Gerrit Code Review is not shipped with Bouncy Castle Crypto v144  
If available, Gerrit can take advantage of features  
in the library, but will also function without it.  
Download and install it now [Y/n]?  
Downloading http://www.bouncycastle.org/download/bcprov-jdk16-144.jar &#8230; OK  
Checksum bcprov-jdk16-144.jar OK  
Generating SSH host key &#8230; rsa&#8230; dsa&#8230; done

HTTP Daemon  
这里使用nginx反向代理gerrit,所以只在loop接口监听即可。  
如果使用域名访问gerrit,最好将规范URL设置为域名形式,发送校验邮件时会使用到

Behind reverse proxy [y/N]? y  
Proxy uses SSL (https://) [y/N]?  
Subdirectory on proxy server [/]:  
Listen on address [*]: 127.0.0.1  
Listen on port [8081]:  
Canonical URL [http://127.0.0.1/]:

Plugins  
选装插件

Install plugin download-commands version v2.8.1 [y/N]?  
Install plugin reviewnotes version v2.8.1 [y/N]?  
Install plugin replication version v2.8.1 [y/N]?  
Install plugin commit-message-length-validator version v2.8.1 [y/N]? 

Initialized /home/review  
Executing /home/review/bin/gerrit.sh start  
Starting Gerrit Code Review:  
因为为ssh服务选在了低于1024的端口,且没有authbind端口授权,所以会出现如下错误,高于1024端口不会。  
FAILED  
error: cannot start Gerrit: exit status 1  
Waiting for server on 127.0.0.1:80 &#8230; OK  
服务器上没有X,所以使用浏览器打开连接失败  
Opening http://127.0.0.1/#/admin/projects/ &#8230;FAILED  
Open Gerrit with a JavaScript capable browser:  
http://127.0.0.1/#/admin/projects/

```  
到了这里只能说好了一半，如果你直接通过域名访问，会报一个认证失败的错误，错误就不贴了，大家自己看看就知道了  
需要说明的是，以上这些配置，也可以通过修改：review/etc/gerrit.config 进行修改，修改之后重启就好了

老夫把自己gerrit.config的配置贴出来如下（部分敏感信息用xxx进行了隐藏，该文件中\*就是\*，不是用于隐藏信息）：

```

[gerrit]  
basePath = git  
canonicalWebUrl = http://gerrit.xxx.com/  
[database]  
type = mysql  
hostname = localhost  
database = reviewdb  
username = root  
[index]  
type = LUCENE  
[auth]  
type = HTTP  
[sendemail]  
smtpServer = smtp.qq.com  
smtpServerPort = 465  
smtpEncryption = ssl  
smtpUser = xxx@xxx.com  
smtpPass = xxx  
sslVerify = false  
from=CodeReview<xxx@xxx.com>  
[container]  
user = root  
javaHome = /data/apps/jdk/jre  
[sshd]  
listenAddress = *:29418  
[httpd]  
listenUrl = proxy-http://*:8081/  
[cache]  
directory = cache

```

4. nginx认证

①. 配置nginx

nginx反向代理gerrit,并且nginx承担http认证,gerrit不会对用户进行认证。gerrit将http认证成功后第一个登录的用户作为管理员,其他用户皆为普通用户。用户第一次http认证成功后,gerrit会为用户生成同名的gerrit用户,只要进一步完善账户即可。比如添加email和公钥等等。管理员为其他普通用户授权。

nginx反向代理配置：

```

user www;  
worker_processes 4;  
pid /run/nginx.pid;

events {  
worker_connentions 768;  
}

http {  
server {  
listen 80;  
server_name gerrit.bridgeli.cn;  
allow all;  
deny all;  
auth_basic "THSTACK INC. Review System Login";  
auth_basic_user_file /data/apps/nginx/passwords;

location / {  
proxy_pass http://127.0.0.1:8081;  
}  
}  
}

```

②. http认证文件

使用htpasswd命令为管理云用户生成http认证配置文件,如果没有htpasswd文件需要安装apache2-utils包。  
ubuntu用户的命令为：

```

sudo apt-get install apache2-utils

```

centos用户的命令是：

```

yum install httpd-tools

```

然后就可以用命令htpasswd设置密码和用户名了，其中passwords是nginx中配置的auth_basic_user_file的认证文件  
```

htpasswd -d passwords admin #admin是用户名

```

以后添加gerrit用户时,同样需要先为其配置http认证,然后用户登录后gerrit会为其自动生成用户帐号,名字与http认证名字一致。

5. 第一次使用要注意的问题

gerrit的权限管理，极其复杂，复杂到我也没有弄明白。。。尤其是第一次使用可以说问题多多，但只要注意一下问题就好了

①. git配置的email一定要和gerrit里注册的email一致，否者push会出错；  
②. 只有管理员和项目拥有者可以使用

```

git push origin master

```

进行代码提交，其他人都不行，但是即使是：管理员和项目拥有者，也不建议使用这个命令，因为这个命令会导致代码不经过review直接被合并进去，失去了Gerrit的最优秀的功能，推荐使用的命令：

```

git push origin HEAD:refs/for/master

```

但是你以为这样就能成功了，还是不会的，因为你看到 remote: ERROR: missing Change-Id in commit message footer 的错误，所以还需要第三步，加入一个文件

③. 把第七部分的附录，copy下来，保存为：commit-msg，然后放到：.git/hooks/下面，如果是Linux系统需要添加x权限，windows直接添加就好了，现在您在提交就没问题了  
④. 以上之后你就可以提交了，提交了之后，进行代码审查就好了

6. 补充说明：

①. 使用http认证登录gerrit后,并无法通过点击&#8221;Sign Out&#8221;退出登录,只能通过直接关闭浏览器窗口来退出当前会话。  
②. 如果需要重新安装gerrit,记得将数据库drop掉再重新创建。

7. 附录

```

#!/bin/sh  
\# From Gerrit Code Review 2.1.2.4  
#  
\# Part of Gerrit Code Review (http://code.google.com/p/gerrit/)  
#  
\# Copyright (C) 2009 The Android Open Source Project  
#  
\# Licensed under the Apache License, Version 2.0 (the "License");  
\# you may not use this file except in compliance with the License.  
\# You may obtain a copy of the License at  
#  
\# http://www.apache.org/licenses/LICENSE-2.0  
#  
\# Unless required by applicable law or agreed to in writing, software  
\# distributed under the License is distributed on an "AS IS" BASIS,  
\# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
\# See the License for the specific language governing permissions and  
\# limitations under the License.  
#

CHANGE_ID_AFTER="Bug|Issue"  
MSG="$1"

\# Check for, and add if missing, a unique Change-Id  
#  
add_ChangeId() {  
clean_message=$(sed -e &#8216;  
/^diff &#8211;git a/.*/{  
s///  
q  
}  
/^Signed-off-by:/d  
/^#/d  
&#8216; "$MSG" | git stripspace)  
if test -z "$clean_message"  
then  
return  
fi

if grep -i &#8216;^Change-Id:&#8217; "$MSG" >/dev/null  
then  
return  
fi

id=$(_gen_ChangeId)  
perl -e &#8216;  
$MSG = shift;  
$id = shift;  
$CHANGE_ID_AFTER = shift;

undef $/;  
open(I, $MSG); $_ = <I>; close I;  
s|^diff &#8211;git a/.*||ms;  
s|^#.*$||mg;  
exit unless $_;

@message = split /n/;  
$haveFooter = 0;  
$startFooter = @message;  
for($line = @message &#8211; 1; $line >= 0; $line&#8211;) {  
$_ = $message[$line];

if (/^[a-zA-Z0-9-]+:/ && !m,^[a-z0-9-]+://,) {  
$haveFooter++;  
next;  
}  
next if /^[ []/;  
$startFooter = $line if ($haveFooter && /^r?$/);  
last;  
}

@footer = @message[$startFooter+1..@message];  
@message = @message[0..$startFooter];  
push(@footer, "") unless @footer;

for ($line = 0; $line < @footer; $line++) {  
$_ = $footer[$line];  
next if /^($CHANGE_ID_AFTER):/i;  
last;  
}  
splice(@footer, $line, 0, "Change-Id: I$id");

$_ = join("n", @message, @footer);  
open(O, ">$MSG"); print O; close O;  
&#8216; "$MSG" "$id" "$CHANGE_ID_AFTER"  
}  
_gen_ChangeIdInput() {  
echo "tree $(git write-tree)"  
if parent=$(git rev-parse HEAD^0 2>/dev/null)  
then  
echo "parent $parent"  
fi  
echo "author $(git var GIT_AUTHOR_IDENT)"  
echo "committer $(git var GIT_COMMITTER_IDENT)"  
echo  
printf &#8216;%s&#8217; "$clean_message"  
}  
_gen_ChangeId() {  
_gen_ChangeIdInput |  
git hash-object -t commit &#8211;stdin  
}

export LC_ALL=C  
add_ChangeId

```

**2015.10.27 补充**

1. 经后续测试，Git是必须安装的；  
2. 邮箱好像是必须的，如果配置信息如下：

```

[sendemail]  
smtpServer = smtp.qq.com  
smtpServerPort = 465  
smtpEncryption = ssl  
smtpUser = xxx@xxx.com  
smtpPass = xxxx  
sslVerify = false  
from=CodeReview<xxx@xxx.com>

```

3. clone项目的时候可以选择：clone with commit-msg hook，这样的话在正常的clone命令后面会添加如下命令，就不会忘记添加commit-msg文件了，但要注意Linux系统是否需要修改添加X权限

```

&& scp -p -P 29418 xxx@xxx.com:hooks/commit-msg demo/.git/hooks/

```

参考资料：  
1. http://openwares.net/linux/gerrit2_setup.html  
2. http://blog.csdn.net/ljchlx/article/details/22277235

 [1]: https://www.bridgeli.cn/archives/153 "世界最大同性交友网站(GitHub)入门使用秘籍"
 [2]: https://www.bridgeli.cn/archives/200 "Git开发最佳实践"