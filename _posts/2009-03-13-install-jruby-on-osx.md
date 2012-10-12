---
layout: post
title: "Mac OS X上安装JRuby on Rails和sqlite3"
description: "本文介绍了如何在Mac OSX 10.5上安装Jruby on Rails和Sqlite3"
category: Rails
tags: [OSX, Rails, Sqlite3]
---
项目开发从Ruby换成了JRuby，于是在自己的Mac上安装JRuby和Rails。过程大同小异，不过有一些稍微值得注意的地方。

我的系统是Mac OS X 10.5.5，安装JRuby 1.1.6，开发使用Sqlite3。
 
##0. 准备
已经安装JDK 5或以上版本  
已经安装Ant  
(其实Mac上已经装好了)  

##1. 安装JRuby
#####1.1安装JRuby
下载[jruby-bin-1.1.6.tar.gz](http://dist.codehaus.org/jruby/) ，解压缩后把目录jruby-1.1.6放到到合适的地方，比如/usr/lib/目录下。
然后把jruby添加到PATH中：  
修改~/.bash_profile文件，最后增加两行RUBY_HOME，并修改PATH如下：
>export RUBY_HOME=/usr/lib/jruby-1.1.6  
>export PATH=$RUBY_HOME/bin:/Users/user/.gem/ruby/1.8/bin:$PATH  
然后刷新profile文件：
>source ~/.bash_profile

##2. 安装Rails 
#####2.1安装rails
>jgem install rails --no-rdoc --no-ri
>或：jruby -S gem install rails --no-rdoc --no-ri (下同)
 
#####2.2 安装rake
>jgem install rake
 
#####2.3 安装activerecord-jdbc
使用sqlite3，所以用：
>jgem install activerecord-jdbcsqlite3-adapter --no-rdoc --no-ri
如果使用mysql，则用：
>jgem install activerecord-jdbcmysql-adapter –-no-rdoc –-no-ri
 
注：上面的安装方式比较简单。如果分别安装activerecord-jdbc-adapter 和sqlite3-ruby ，会有以下错误：
>Building native extensions.  This could take a while...
>/usr/lib/jruby-1.1.6/lib/ruby/1.8/mkmf.rb:7: JRuby does not support native extensions. Check wiki.jruby.org for alternatives. (NotImplementedError)
>    from /usr/lib/jruby-1.1.6/lib/ruby/1.8/mkmf.rb:1:in `require'
>    from extconf.rb:1
>ERROR:  Error installing sqlite3-ruby:
>    ERROR: Failed to build gem native extension.
 
#####2.3 安装openssl
上面安装时一直有错误提示：
>JRuby limited openssl loaded. gem install jruby-openssl for full support.
>http://wiki.jruby.org/wiki/JRuby_Builtin_OpenSSL
这样安装一下openssl就可以了：
>jruby -S gem install jruby-openssl
 
使用时注意修改database.yml中的adapter：
 
>development:
>  adapter: jdbcsqlite3 
>  database: db/development.sqlite3
>  timeout: 5000

下面你就可以开始使用JRuby on Rails了。
参考文章：
*Get JRuby onto the Rails on Mac OS X (注：此文有一些错误，后面的评论给了纠正)
*TOTD #37: SQLite3 with Ruby-on-Rails on GlassFish Gem