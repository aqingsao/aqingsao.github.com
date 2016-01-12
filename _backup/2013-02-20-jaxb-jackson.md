---
layout: post
title: "JaxB Jackson实战"
description: ""
category: 
tags: []
---

Jersy是一个优秀的RESTful框架，当我们使用Jersey为RESTful应用提供JSON格式的数据时，Jackson是个很好的选择。只需要在程序中配置FEATURE_POJO_MAPPING为true，并把jackson的jar包加入到classpath，就可以使用了。
return Guice.createInjector(new ServletModule() {
            @Override
            protected void configureServlets() {
                serve("/api/*").with(GuiceContainer.class, new ImmutableMap.Builder<String, String>()
                        .put(FEATURE_POJO_MAPPING, "true").build());
            }
        });

应用向客户端提供数据，常用的格式有JSON或XML，而有时需要同时提供多种格式。JAXB是Java对象和XML文档映射的规范，通过注解的方式配置映射关系，可以完成二者间的序列化和反序列化。

在RESTful框架的参考实现Jersey中，

虽然XML与JSON格式不同，但是其在序列化和反序列化方面有诸多相似之处，这使得JAXB的元数据可用于序列化成JSON数据。

要想让Jackson支持JAXB注解，在classpath中加入jackson-module-jaxb-annotations-2.*.jar文件即可。

目前Jackson2.0支持JAXB的大多数注解，但是由于二者概念有一致的地方等原因，仍然有少数不能支持，详细列表请参考[官方网站](http://wiki.fasterxml.com/JacksonJAXBAnnotations)，不过对于我们的应用来说，这已经足够了。


http://www.360doc.com/content/09/0331/21/59141_2980566.shtml
http://www.ibm.com/developerworks/cn/webservices/1003_sunzg_jaxb/index.html
http://qify.iteye.com/blog/248205
http://zh.wikipedia.org/wiki/JAXB