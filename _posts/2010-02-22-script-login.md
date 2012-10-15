---
layout: post
title: "提供用户名和密码的SSH自动登录脚本"
description: ""
category: Script
tags: [Script, Expect]
---
使用VPN，每次都要在Terminal上重复输入命令：

{% highlight sh%}
ssh -D port user@host
{% endhighlight%}

出来密码提示符后，把复杂的密码拷贝下来，然后粘贴到Terminal，敲回车...
 
终于忍受不了这样的重复了，于是用Shell写一个可以自动登录的脚本：

{% highlight sh linenos%}
#!/usr/bin/expect -f  
  
set port port_no  
set user user_name  
set host host_name  
set password my_password  
set timeout -1  
  
spawn ssh -D $port $user@$host  
expect "*assword:*"  
  
send "$password\r"  
expect eof  
{% endhighlight%}

把上面的代码命名成vpn，并设置755的权限之后，使用起来就方便了：./vpn。
 
上面脚本中的自动交互用到了expect，那么什么是expect呢？
expect是一个基于Tcl的用于自动交互操作的工具语言，它适合用来编写需要交互的自动化脚本，比如上面提到的SSH输入用户名密码，自动FTP等等场景。
 
除了具有Tcl的语法，expect提供了几个常用的命令：
1. send
用来发送一个字符串，比如 send "hello world"。
初始情况下，这个字符串会发送到标准输出。如果你用的是max OSX或者linux，可以在Terminal下直接输入expect命令并回车，就进入了expect交互环境，此时，输入send "hello world"就可以看到结果。
一旦你的程序已经与其他程序进行交互，字符串就会被发送到其他程序那里。如上面的例子脚本中，我们调用send ”$password\r"就是把密码发送给SSH连接的服务器端指定端口。
 
2. expect
与send相反，expect用来等待你所期望的字符串。比如expect "hello"
在expect后面跟的字符串中，你可以指定一个正则表达式。
expect会一直等待下去，除非收到的字符串与预期的格式匹配，或者到了超期时间。
 
3. spawn
spawn用来启动一个新的进程，比如上面的spawn ssh -D $port $user@$host，Expect会执行命令“ssh -D $port $user@$host”。
在交互式的场景中，当你输入命令后，可能服务器端会返回一些操作提示符，以让你输入命令。Expect提供了这样三个常用的命令，spawn, expect和send，恰好满足这种需要。把它们结合起来使用，可以实现很多简单的自动化脚本。
 
其它常用的命令还有：interact，比如你通过脚本自动连接到了某个ftp，并输入了用户名密码，此时需要人工输入一些命令，就可以使用interact命令，它会把脚本的控制权交给用户；sleep，等待多少秒等等。
 
由于expect是从Tcl继承下来的，所以也支持Tcl的语法和命令，比如变量声明、流程控制等等。
 
上面脚本的一些解释：
1. set timeout 300：设置超时时间300s。如果设为-1，代表永不超时。
2. expect eof：等待接受文件结束符。