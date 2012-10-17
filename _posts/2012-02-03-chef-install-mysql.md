---
layout: post
title: '通过Chef安装MySQL无法访问的问题'
category: Tools
tags: [Chef, Tools, MySQL]

---

通过自动化的配置管理工具可以简化项目的部署，我们尝试通过Chef来开通一个测试环境，其中用到了mysql。
做为最经常用到的技术栈之一，MySQL已经有了现成的cookbook，其安装也较为简单。然后安装成功之后，发现Web应用程序无法连接，然而通过命令行可以！

所以可以肯定是一个配置的问题，首先怀疑端口，但是检查后发现都是默认的3306端口。
然后怀疑数据库的连接地址，Web应用程序中的配置：localhost:3306。通过netstat -atn查看一下：看到这样一条：

{%highlight console%}
tcp   0   0 <strong>10.29.1.25</strong>:3306    0.0.0.0:*   LISTEN 
{%endhighlight%}

问题就出在这里，10.29.1.25意味着只监听远程网络的端口调用，而不会监听本地端口。最简单的方法是把应用程序中的localhost:3306改成10.29.1.25:3306。

另外一个解决方法让mysql服务支持本地端口调用。查看/etc/mysql/my.cnf文件，打开可以看到这一行：

{%highlight console%}
bind-address：10.29.1.25
{%endhighlight%}

如果把后面的地址改为：0.0.0.0，这意味着服务会绑定本机上所有的ip地址。如果机器有多个ip地址，这尤其适用。重启mysql服务，发现应用程序工作正常。

所以在使用chef开通mysql时，要注意绑定正确的ip地址。

