---
layout: post
title: "解决Rails rake fixtures加载的顺序问题"
keywords: Rails, rake, fixtures
description: "解决Rails rake fixture的加载顺序问题"
category: Rails
tags: [Rails, Ruby]
---
Rails中有个rake rask，叫做 db:fixtures:load，可以帮你自动load指定目录下（text/fixtures）的yml或csv文件。然而，如果这些文件之间有依赖关系，这个task会失败。

比如有两个模型Image和Locations，依赖关系为：Image has_and_belongs_to_many Locations 的单向关联。
数据库中分别对应Images和Locations表，并通过image_category_rel表关联。

在test/fixtures目录下有这两个模型的yml文件，分别为：

{% highlight yaml %}
image1:  
  id: 1  
  locations: china  
{% endhighlight %}

{% highlight yaml %}
china:  
  name: china  
{% endhighlight %}

如果db:fixtures:load先加载category文件，然后加载images文件，肯定没问题。不幸的是，我们使用时出现了这样的错误：

{% highlight yaml %}
The INSERT statement conflicted with the FOREIGN key constraint ......  
{% endhighlight %}

经过进一步测试，发现就是load时首先加载了images文件，插入中间表时无法找到对Locations表的关联。
 
所以解决yml文件的加载顺序就可以。上网google了一下，已经有人遇到了同样的问题，而且incognito 给了一个这样的task：

{% highlight ruby %}
task :load_fixtures_ordered => :environment do  
  require 'active_record/fixtures'    
  ordered_fixtures = Hash.new  
  ENV["FIXTURE_ORDER"].split.each { |fx| ordered_fixtures[fx] = nil }  
  other_fixtures = Dir.glob(File.join(RAILS_ROOT, 'test', 'fixtures', '*.{yml,csv}')).collect { |file| File.basename(file, '.*') }.reject {|fx| ordered_fixtures.key? fx }  
  ActiveRecord::Base.establish_connection(ENV['RAILS_ENV'])  
  (ordered_fixtures.keys + other_fixtures).each do |fixture|  
    Fixtures.create_fixtures('test/fixtures',  fixture)  
  end unless :environment == 'production'   
end 
{% endhighlight %}

觉得写的有些麻烦了，于是和小龙一块整了一个简单一点的，如下：

{% highlight ruby %}
task :load_ordered => :environment do  
      require 'active_record/fixtures'  
      ActiveRecord::Base.establish_connection(RAILS_ENV.to_sym)  
      ordered = %w( location image )  
      (ordered + Dir.glob(File.join(RAILS_ROOT, 'test', 'fixtures', '*.{yml,csv}')).collect { |file| File.basename(file, '.*') }).uniq.each do |fixture_name|  
        Fixtures.create_fixtures('test/fixtures', fixture_name)  
      end unless :environment == 'production'  
 end  
{% endhighlight %}

测试发现，运行正常。
这个任务主要用到了数组的+、uniq方法，觉得比incognito上面更简单一些。