---
layout:     post
title:      "Freemarker中如何避免xss漏洞"
category:   Freemarker
date:       2019-06-20 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Freemarker
---

## 什么是XSS漏洞
试想一下，如果我们开发一个订单系统，订单名称如果没有做限制，允许用户输入任意字符，那么就有产生XSS的危险。攻击者可以很容易编写一个恶意JS脚本,然后将当前登录用户的cookie或者其他敏感信息抓取到，发送给攻击者自己，这就是XSS（跨站脚本）攻击。如何解决这个问题，首先我们想到的是在用户输入订单的时候，我们对订单名称做限制，不允许输入特殊字符。这样是可以避免的，不过对于一个大的系统来说，用户可以输入的字段太多了，如果能够全部校验，是最好的。如果不能做的话，还可以让前端在做展示的时候进行html转义处理一下，这样原本scirpt标签以及当中的内容就被当做一个字符串展示出来，而不是当做代码执行了。一般现在的reactjs等前端框架已经默认支持防xss了。今天我遇到的问题是，有一部分freemarker写的页面存在XSS的问题。

## FreeMarker中解决XSS
可以通过对用户输入的字段进行html转义，来有效的避免XSS问题。freemarker模板中的变量通过${value}这样的形式引入，但是挨个修改成`${value?html}`的形式未免工作量太大；查阅官方文档，通过escape标签可以对整个模板中$号引入的变量全部进行一次html转义。比如：
```
<#assign x = "<test>">
<#macro m1>
  m1: ${x}
</#macro>
<#escape x as x?html>
  <#macro m2>m2: ${x}</#macro>
  ${x}
  <@m1/>
</#escape>
${x}
<@m2/>
```
会输出：
```
  &lt;test&gt;
  m1: <test>
<test>
m2: &lt;test&gt;
```

这种方式完全满足现在的改造需求。在这个项目中，freemarker模板是存储在数据库中，所有用到freemarker渲染的地方均是用的同一个入口；这样工作量就不大了。我们知道，freemarker加载模板是通过TemplateLoader这个接口来实现的；只需要在加载模板的时候在模板的头部加上`<#escape x as x?html>`在尾部加上`</#escape`就可以对模板中所有的变量进行html转义了。通过这种方式数据库中的数据也不用修改，将来生成模板，也不需要考虑做html转义的问题。

```
public interface TemplateLoader {
	
    public Object findTemplateSource(String name)
    throws IOException;

    public long getLastModified(Object templateSource);
  
    public Reader getReader(Object templateSource, String encoding) throws IOException;
 
    public void closeTemplateSource(Object templateSource) throws IOException;
    
}
```
由于实现逻辑还是从外部读取字符串加载，所以TemplateLoader接口支持html转义的实现和StringTemplateLoader的逻辑差不多。

```
public class HtmlEscapeTemplateLoader implements TemplateLoader {

    private static final String HTML_ESCAPE_PREFIX = "<#escape x as x?html>";
    private static final String HTML_ESCAPE_SUFFIX = "</#escape>";

    // 为了支持并发，这里采用ConcurrentMap
    private final Map<String, StringTemplateSource> templates = Maps.newConcurrentMap();

    public void putTemplate(String name, String templateContent) {
        putTemplate(name, templateContent, System.currentTimeMillis());
    }

    public void putTemplate(String name, String templateContent, long lastModified) {
        templates.put(name, new HtmlEscapeTemplateLoader.StringTemplateSource(name, templateContent, lastModified));
    }

    public boolean removeTemplate(String name) {
        return templates.remove(name) != null;
    }

    public void closeTemplateSource(Object templateSource) {
    }

    public Object findTemplateSource(String name) {
        return templates.get(name);
    }

    public long getLastModified(Object templateSource) {
        return ((HtmlEscapeTemplateLoader.StringTemplateSource) templateSource).lastModified;
    }

    public Reader getReader(Object templateSource, String encoding) throws IOException {
        Reader reader = new StringReader(((StringTemplateSource) templateSource).templateContent);
        String templateText = IOUtils.toString(reader);
        return new StringReader(HTML_ESCAPE_PREFIX + templateText + HTML_ESCAPE_SUFFIX);
    }

    private static class StringTemplateSource {
        private final String name;
        private final String templateContent;
        private final long lastModified;

        StringTemplateSource(String name, String templateContent, long lastModified) {
            if (name == null) {
                throw new IllegalArgumentException("name == null");
            }
            if (templateContent == null) {
                throw new IllegalArgumentException("source == null");
            }
            if (lastModified < -1L) {
                throw new IllegalArgumentException("lastModified < -1L");
            }
            this.name = name;
            this.templateContent = templateContent;
            this.lastModified = lastModified;
        }

        @Override
        public int hashCode() {
            final int prime = 31;
            int result = 1;
            result = prime * result + ((name == null) ? 0 : name.hashCode());
            return result;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            HtmlEscapeTemplateLoader.StringTemplateSource other = (HtmlEscapeTemplateLoader.StringTemplateSource) obj;
            if (name == null) {
                if (other.name != null)
                    return false;
            } else if (!name.equals(other.name))
                return false;
            return true;
        }


        @Override
        public String toString() {
            return name;
        }

    }

    /**
     * Show class name and some details that are useful in template-not-found errors.
     *
     * @since 2.3.21
     */
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClassNameForToString(this));
        sb.append("(Map { ");
        int cnt = 0;
        for (String name : templates.keySet()) {
            cnt++;
            if (cnt != 1) {
                sb.append(", ");
            }
            if (cnt > 10) {
                sb.append("...");
                break;
            }
            sb.append(StringUtil.jQuote(name));
            sb.append("=...");
        }
        if (cnt != 0) {
            sb.append(' ');
        }
        sb.append("})");
        return sb.toString();
    }

    private static String getClassNameForToString(TemplateLoader templateLoader) {
        final Class tlClass = templateLoader.getClass();
        final Package tlPackage = tlClass.getPackage();
        return tlPackage == Configuration.class.getPackage() || tlPackage == TemplateLoader.class.getPackage()
                ? tlClass.getSimpleName() : tlClass.getName();
    }
}
```
至此实现了在freemarker中自动进行html转义避免XSS问题的过程。参考网上有些资料可以修改freemarker的源码，对其在进行`$`号解析的时候，自动加上html转义的逻辑，也是可以的。如果某些特殊情况下，就是需要展示html形式的内容而不需要转义，或者有人错误的将变量进行了一次转义比如写成`${!(value)?html}`的形式，会怎样？

如果就是需要展示html形式的内容而不需要转义，可以用`<#noescape>`标签将不需要转义的变量包裹起来，这样就算外层有`<#escape>`也不会进行转义了。如果在外部有`<#escape>`的情况下，变量自身又做了一次转义，那么该变量会被转义两次。正常字符不会有影响，含有特殊字符的话，可能比较难看了。

## 参考
https://blog.csdn.net/shadowsick/article/details/80768868
https://my.oschina.net/greki/blog/83246?p=1
