---
layout: post
title: "Spring Security intercept url"
description: ""
category: Java
tags: [Java, Spring]
---
客户遗留系统的权限部分使用了Spring security中的FilterSecurityInterceptor，可以通过设置intercept-url及对应的权限进行验证。
其典型用法是在配置文件的Bean中定义：
 
{% highlight xml%}
<bean id="filterSecurityInterceptor" class="org.springframework.security.intercept.web.FilterSecurityInterceptor">
    <property name="objectDefinitionSource">
        <security:filter-invocation-definition-source path-type="ant">
            <security:intercept-url pattern="/books/*" access="PRIVILEGE_BOOKS"/>
        </security:filter-invocation-definition-source>
    </property>
</bean>
{%endhighlight%} 

但由于我们新使用了RESTful的url，所以上面的配置无法验证“/books”这样的url。
 
**\[尝试一\]**
把pattern修改成了"/books\*”，希望它可以拦截所有以“books”开头的url。
但结果发现它只拦截“/books”或者“/booksa”，而不会拦截“/books/1”。
 
究其原因，是path-type采用Ant方式。所以“\*”只能匹配任意多个字符，但是对分隔符“/”无效。
 
**\[尝试二\]**
既然采用了Ant匹配方式，写就比较容易了。但是犹豫程序部署及测试时间较长，于是写了一个快速的单元测试：

{% highlight java linenos%}
import org.springframework.util.AntPathMatcher;  
import java.util.Arrays;  
  
public class QuickTest {  
    public static void main(String[] args) {  
        AntPathMatcher matcher = new AntPathMatcher();  
        String[] pattern = {"/em*", "/em**", "/em/*", "/em/**", "/em/**/*"};  
        String[] str = {"/em", "/ems", "/em/a", "/em/a/b", "/em/a/b/c/"};  
        for (String s : str) {  
            for (String p : pattern) {  
                System.out.println("result:" + matcher.match(p, s));  
            }  
        }  
    }  
}  
{%endhighlight%} 
 
经过测试，结果如下：
{% highlight console%}
            /book     /books	/books/a	/books/a/b	/books/a/b/c
EXPECTED	false	  true	    true	    true	    true
/books*	    false	  true	    false	    false	    false
/books**	false     true	    false	    false	    false
/books/*	false	  false	    true	    false	    false
/books/\*\*	false	  true	    true	    true	    true
/books/\*\*/*	false	  false	    true	    true	    true
{%endhighlight%} 

根据与期望的结果比较，知道正确的模式是“/books/\*\*”

当然，如果你不想使用ant格式，也可以采用正则表达式，将bean定义中的path-type改成“regex”即可。