---
layout: post
title: "正确处理CentOS上Nginx部署的静态资源403问题"
date: 2020-02-24 09:40:13 +0800
categories:
---

## 前言

接着上一篇博客[低成本在家部署一个Server](../23/deploy-server-at-home.html)，这台服务器在我家里充当了一个服务总网关的作用，我在这台服务器上安装了Nginx，在Nginx上部署了HTTPS，然后当我在我工作使用的MacBook上启动一个测试Web服务的时候，我可以通过在Nginx上配置反向代理来对外提供这个Web服务。

我同样在Nginx上配置了一个静态目录，我在这个静态目录里放了一些学习资料，这样我就可以随时随地访问这些学习资料了。因为macOS上自带了Apache，所以我之前一直用Apache做我的静态文件服务器，Nginx还是属于刚开始接触。我在Nginx的配置文件里配置好静态目录，也给静态目录及其子目录设置了所有用户可以读取的权限，但是启动Nginx并访问，403 Forbidden。于是上网查找解决方案，在我查到的解决这种问题的博客里，绝大多数都是通过在Nginx的配置文件里把user改为root，虽然我对Linux也不是很懂，但随便给程序以root权限运行的方式绝对是下下之策。

后来发现这个问题是SELinux的限制导致的，在Nginx官网上找到了正确的解决方案。

我这里用谷歌翻译后发到博客上，以供中文朋友参考，可以点击[原文](https://www.nginx.com/blog/using-nginx-plus-with-selinux/)查看英文原版。

# 将NGINX和NGINX Plus与SELinux结合使用

编者说–标题为“ NGINX：升级到RHEL 6.6 / CentOS 6.6时SELinux的更改”的博客文章重定向到此处。 本文提供了更新的通用信息。

现代Red Hat Enterprise Linux（RHEL）和相关发行版上的Security-Enhanced Linux（SELinux）的默认设置可能非常严格，这是出于安全性而不是便利性的考虑。 尽管默认设置在其默认配置中不限制NGINX Open Source和NGINX Plus的功能，但是除非您在SELinux中明确允许它们，否则可能会阻止您可能配置的其他功能。 本文介绍了可能的问题以及建议的解决方法。

[编者说–本文适用于NGINX开源和NGINX Plus。 为了便于阅读，始终使用术语“ NGINX”。

CentOS是最初衍生自RHEL的相关发行版，并受到NGINX和NGINX Plus的支持。 此外，NGINX Plus还支持相关的Amazon Linux和Oracle Linux发行版。 它们的默认SELinux设置可能与CentOS和RHEL不同。 请参阅供应商文档。]

## SELinux概述

默认情况下，现代RHEL和CentOS服务器上已启用SELinux。 每个操作系统对象（进程，文件描述符，文件等）都标记有SELinux上下文，该上下文定义了对象可以执行的权限和操作。 在RHEL 6.6 / CentOS 6.6和更高版本中，NGINX带有`httpd_t`上下文标记：

```Bash
# ps auZ | grep nginx
unconfined_u:system_r:httpd_t:s0 3234 ? Ss 0:00 nginx: master process /usr/sbin/nginx \
                                                -c /etc/nginx/nginx.conf
unconfined_u:system_r:httpd_t:s0 3236 ? Ss 0:00 nginx: worker process
```

`httpd_t`上下文允许NGINX侦听公共Web服务器端口，访问**/etc/nginx**中的配置文件，以及访问标准docroot位置（**/usr/share/nginx**）中的内容 。 它不允许许多其他操作，例如代理上游位置或通过套接字与其他进程通信。

## 暂时为NGINX禁用SELinux

要暂时禁用针对httpd_t上下文的SELinux限制，以便NGINX可以执行与非SELinux操作系统中相同的所有操作，请将httpd_t上下文分配给许可域。 有关详细信息，请参见下一部分。

```Bash
# semanage permissive -a httpd_t
```

## 更改SELinux模式

SELinux可以以“强制”，“允许”或“禁用”模式（也称为域）运行。 在进行可能会违反默认（严格）权限的NGINX配置更改之前，可以在测试环境（如果可用）或生产环境中将SELinux从“强制”模式更改为“允许”模式。 在“允许”模式下，SELinux允许所有操作，但在“强制”模式下记录可能违反安全策略的操作。

要将“ httpd_t”添加到“允许的”域列表中，请运行以下命令：

```Bash
# semanage permissive -a httpd_t
```

要从“允许的”域列表中删除“ httpd_t”，请运行：

```Bash
＃semanage permissive -d httpd_t
```

要将模式全局设置为“允许”，请运行：

```Bash
＃setenforce 0
```

要将模式全局设置为**执行**，请运行：

```Bash
＃setenforce 1
```

##解决SELinux安全异常

在“允许”模式下，安全例外记录到默认的Linux审核日志中，即**/var/log/audit/audit.log**。如果您遇到仅在NGINX以“强制”模式运行时发生的问题，请查看以“许可”模式记录的异常，并更新安全策略以允许它们。

### 问题1：禁止代理连接

默认情况下，SELinux配置不允许NGINX连接到远程HTTP，FastCGI或其他服务器，如审核日志消息所示，如下所示：

```
type=AVC msg=audit(1415714880.156:29): avc:  denied  { name_connect } for  pid=1349 \
  comm="nginx" dest=8080 scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=system_u:object_r:http_cache_port_t:s0 tclass=tcp_socket
type=SYSCALL msg=audit(1415714880.156:29): arch=c000003e syscall=42 success=no \
  exit=-115 a0=b \a1=16125f8 a2=10 a3=7fffc2bab440 items=0 ppid=1347 pid=1349 \
  auid=1000 uid=497 gid=496 euid=497 suid=497 fsuid=497 egid=496 sgid=496 fsgid=496 \
  tty=(none) ses=1 comm="nginx" exe="/usr/sbin/nginx" \
  subj=unconfined_u:system_r:httpd_t:s0 key=(null)
```

audit2why命令解释消息代码（1415714880.156：29）：

```
# grep 1415714880.156:29 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1415714880.156:29): avc:  denied  { name_connect } for  pid=1349 \
  comm="nginx" dest=8080 scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=system_u:object_r:http_cache_port_t:s0 tclass=tcp_socket
 
        Was caused by:
        One of the following booleans was set incorrectly.
        Description:
        Allow httpd to act as a relay
 
        Allow access by executing:
        # setsebool -P httpd_can_network_relay 1
        Description:
        Allow HTTPD scripts and modules to connect to the network using TCP.
 
        Allow access by executing:
        # setsebool -P httpd_can_network_connect 1
```

audit2why的输出表明，您可以通过启用**httpd_can_network_relay**和**httpd_can_network_connect**布尔选项中的一个或两个来允许NGINX建立代理连接。您可以临时或永久启用它们，后者可以通过添加**‑P**标志（如输出所示）来启用。

#### 了解布尔选项

sesearch命令提供有关布尔选项的更多信息，并且在安装**setools**软件包（yum install setools）时可用。在这里，我们显示**httpd_can_network_relay**和**httpd_can_network_connect**选项的输出。

##### httpd_can_network_relay布尔选项

以下是sesearch命令有关**httpd_can_network_relay**选项的输出：

```
# sesearch -A -s httpd_t -b httpd_can_network_relay
Found 10 semantic av rules:
   allow httpd_t gopher_port_t : tcp_socket name_connect ;
   allow httpd_t http_cache_client_packet_t : packet { send recv } ;
   allow httpd_t ftp_port_t : tcp_socket name_connect ;
   allow httpd_t ftp_client_packet_t : packet { send recv } ;
   allow httpd_t http_client_packet_t : packet { send recv } ;
   allow httpd_t squid_port_t : tcp_socket name_connect ;
   allow httpd_t http_cache_port_t : tcp_socket name_connect ;
   allow httpd_t http_port_t : tcp_socket name_connect ;
   allow httpd_t gopher_client_packet_t : packet { send recv } ;
   allow httpd_t memcache_port_t : tcp_socket name_connect ;
```

此输出表明**httpd_can_network_relay**允许标有**httpd_t**上下文的进程（例如NGINX）连接到各种类型的端口，包括**http_port_t**类型：

```
# semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
```

要将更多端口（此处为**8082**）添加到**http_port_t**允许的端口集中，请运行：

```Bash
# semanage port -a -t http_port_t -p tcp 8082
```

如果此命令的输出表明已经定义了端口，如以下示例所示，则意味着该端口已包含在另一个集中。不要将其重新分配给http_port_t，因为其他服务可能会受到负面影响。

```
# semanage port -a -t http_port_t -p tcp 8080
/usr/sbin/semanage: Port tcp/8080 already defined
# semanage port -l | grep 8080
http_cache_port_t              tcp      3128, 8080, 8118, 8123, 10001-10010
```

##### httpd_can_network_connect布尔选项

这是**sesearch**命令关于**httpd_can_network_connect**选项的输出：

```
# sesearch -A -s httpd_t -b httpd_can_network_connect
Found 1 semantic av rules:
   allow httpd_t port_type : tcp_socket name_connect ;
```

此输出表明**httpd_can_network_connect**允许标有**httpd_t**上下文的进程（例如NGINX）连接到所有具有**port_type**属性的TCP套接字类型。要列出它们，请运行：

```
＃seinfo -aport_type -x
```

### 问题2：禁止文件访问

默认情况下，SELinux配置不允许NGINX访问知名授权位置之外的文件，如以下审核日志消息所示：

```
type=AVC msg=audit(1415715270.766:31): avc:  denied  { getattr } for  pid=1380 \
  comm="nginx" path="/www/t.txt" dev=vda1 ino=1084 \
  scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=unconfined_u:object_r:default_t:s0 tclass=file
```

audit2why命令解释消息代码（**1415715270.766:31**）：

```
# grep 1415715270.766:31 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1415715270.766:31): avc:  denied  { getattr } for  pid=1380 \
  comm="nginx" path="/www/t.txt" dev=vda1 ino=1084 \
  scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=unconfined_u:object_r:default_t:s0 tclass=file
 
    Was caused by:
        Missing type enforcement (TE) allow rule.
 
        You can use audit2allow to generate a loadable module to allow this access.
```

禁止文件访问时，有两个选择。

#### 选项1：修改文件标签

修改文件标签，以便NGINX（作为带有**httpd_t**上下文的进程）可以访问文件：

```
＃chcon -v --type = httpd_sys_content_t /www/t.txt
```

默认情况下，当重新标记文件系统时，将删除此修改。要使更改永久生效，请运行：

```
＃semanage fcontext -a -t httpd_sys_content_t /www/t.txt
＃restorecon -v /www/t.txt
```

要修改文件组的文件标签，请运行：

```
＃semanage fcontext -a -t httpd_sys_content_t /www(/.*)?
＃restorecon -Rv / www
```

#### 选项2：扩展httpd_t域权限

扩展**httpd_t**的策略以允许访问其他文件位置：

```
# grep nginx /var/log/audit/audit.log | audit2allow -m nginx > nginx.te
# cat nginx.te
 
module nginx 1.0;
 
require {
        type httpd_t;
        type default_t;
        type http_cache_port_t;
        class tcp_socket name_connect;
        class file { read getattr open };
}
 
#============= httpd_t ==============
allow httpd_t default_t:file { read getattr open };
 
#!!!! This avc can be allowed using one of these booleans:
#     httpd_can_network_relay, httpd_can_network_connect
allow httpd_t http_cache_port_t:tcp_socket name_connect;
```

要生成已编译的策略，请包含`-M`选项：

```
＃grep nginx /var/log/audit/audit.log | audit2allow -M nginx
```

要加载该策略，请运行`semodule -i`，然后使用`semodule -l`验证成功：

```
＃semodule -i nginx.pp
＃semodule -l | grep nginx
Nginx 1.0
```

此更改在重新启动后仍然存在。

### 问题3：NGINX无法绑定到其他端口

默认情况下，SELinux配置不允许NGINX侦听（**bind()**）除**http_port_t**类型白名单中的默认端口外的TCP或UDP端口：

```
# semanage  port -l | grep http_port_t
http_port_t                    tcp      80, 443, 488, 8008, 8009, 8443
```

如果尝试将NGINX配置为在未列入白名单的端口上侦听（在NGINX配置中的http，流或邮件上下文中使用listen指令），则在验证（nginx -t）或重新加载NGINX时会收到错误消息配置，如以下NGINX日志条目所示：

YYYY/MM/DD hh:mm:ss [emerg] 46123#0: bind()到0.0.0.0:8001失败（13：权限被拒绝）
您可以使用semanage将所需的端口（此处为8001）添加到http_port_t类型：

```
# semanage port -a -t http_port_t -p tcp 8001
```

用新配置重新加载NGINX。

```
# nginx -s reload
```

## 其他资源

SELinux是用于管理操作系统权限的复杂而强大的工具。其他信息在以下文档中提供。

* 增强安全性的Linux，RHEL 6或RHEL 7
* SELinux（CentOS使用方法）
* 增强安全性的Linux用户指南（Fedora项目）
* SELinux项目主页
* 安全性增强的Linux（美国国家安全局）
