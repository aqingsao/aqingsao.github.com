---
layout: post
title: "在Ubuntu 8.10上安装Rails "
keywords: Ubuntu 8.10, Rails
description: "介绍了在Ubuntu 8.10上安装Rails的步骤"
category: Rails
tags: [Ubuntu, Rails, Ruby]
---
有了aptitude和gem，在ubuntu上安装Rails不是什么难事儿，但实际安装时没准碰到什么问题，比如漏掉了某些类库、必须更新版本等等。如果出现错误了去网上现查也可以，但是耗时耗力，搞不好1、2个小时搭进去了。我在昨天和今天装了3台机器，整理了一下在Ubuntu 8.10上安装Rails 2.2.2的步骤，第一台安装花了2个小时，最后1台只花了10分钟。步骤如下：
##0. 准备
#####0.0 Ubuntu 8.10 安装完毕，网络连接正常
 
#####0.1 修改apt的sourcelist： 
>sudo vi /etc/apt/source.list 
默认提供的source list比较慢，全部注释掉，然后改成比较快的。如果你手头没有，可以试试这个：
>deb http://ubuntu.cn99.com/ubuntu/ intrepid main restricted universe multiverse  
>deb-src http://ubuntu.cn99.com/ubuntu/ intrepid main restricted universe multiverse
 
#####0.2 更新apt package列表： 
>sudo apt-get update 

##1. 安装Ruby
#####1.1安装ruby
>sudo apt-get install ruby 
检查：`ruby -v =>ruby 1.8.7`
 
#####1.2安装ruby-dev
>sudo apt-get install ruby-dev 
如果不安装，后面某些步骤安装openssl或sqlite3的时候会出错

##2. 安装Rubygems 
#####2.1安装gem
>sudo apt-get install rubygems 
检查：`gem -v=>1.2.0`
默认安装的rubygems版本是1.2.0，而rake需要1.3.1，所以必须进行更新：
 
#####2.2 升级到1.3.1
>sudo gem install rubygems-update  
>sudo /var/lib/gems/1.8/bin/update_rubygems 
检查：`gem -v =>1.3.1`，版本已经是1.3.1了

##3. 安装Rails
>sudo gem install -v=2.2.2 rails 
检查：`rails -v =>Rails 2.2.2`
备注：如果结果是“The program 'rails' is currently not installed.  ”，可以这样修复：
在~/.bashrc文件最后添加一行：export PATH=/var/lib/gems/1.8/bin:$PATH 
然后用source刷新一下就可以了。

##4. 安装openssl和sqlite3 
#####4.1 安装openssl
>sudo apt-get install libopenssl-ruby
 
#####4.2安装sqlite3
>sudo apt-get install libsqlite3-dev  
>sudo gem install sqlite3-ruby 
如果你还想安装其它插件，可以在这里也可以以后安装。

##5. 创建一个例子测试 
#####5.1创建应用
>rails firstapp
 
#####5.2 测试数据库
>cd firstapp  
>rake db:create
 
#####5.3 启动
>script/server 

如果以上三步都没问题，说明安装没有问题。