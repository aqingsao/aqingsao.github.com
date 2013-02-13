---
layout: post
title: 使用Expect创建VPN自动登陆脚本
keywords: Expect, VPN, Automation, Scripts
description: Write automation scripts with expect
category: Tools
tags: [Tools, Automation]
---

平时工作，需要经常在切换到VPN，每次都得面对烦人的登录框，重复输入用户名、密码：
  
[![VPN login page](/assets/images/vpnLoginPage.png "VPN login page")](/assets/images/vpnLoginPage.png)
  
很明显，需要一个脚本搞定它。
Cisco VPN提供了命令行，可以完成这一工作：
<em> vpnclient connect $profile user $username pwd $password</em>
可以写一个简单的shell脚本，但在执行时，它会跳出一个提示：Do you wish to continue? (y/n):，此时你需要输入“y”。

这样的场景，很适合交互式的自动化脚本工具来完成，我选择了EXPECT。

{%highlight sh linenos%}
#!/usr/bin/expect -f
set user user_name
set pwd password
set profile my_profile
spawn vpnclient connect $profile user $user pwd $pwd
expect "* to continue*"    
send "y\r"
interact
{%endhighlight%}

运行一下，已经可以工作。但是如果vpn已经连接的话，该脚本会报错误，我们需要连接之前检查一下状态：
<em>vpnclient stat</em>

{%highlight sh linenos%}
#!/usr/bin/expect -f
set user user_name
set pwd password
set profile my_profile
spawn vpnclient stat
expect {
  "*No connection exists*" {
    spawn vpnclient connect $profile user $user pwd $pwd
    expect "* to continue*"
    send "y\r"
  }
  "*Connection Entry*" {
    puts "VPN already connected"
  }
}
interact
{%endhighlight%}

试一试，OK。接下来，就是把该脚本加到QuickSilver里面，以后，只需要在QuickSilver中输入vpnConnect.sh，回车，就可以自动连接VPN：

[![VPN login script](/assets/images/vpnLoginScript.png "VPN login script")](/assets/images/vpnLoginScript.png)

搞定！


