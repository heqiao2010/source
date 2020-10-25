---
layout:     post
title:      "Freemarker中如何进行Json转化"
category: Freemarker
date:       2019-07-26 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Freemarker
---

### 缘起
之前在开发过程中,freemarker的解析经常遇见一些问题,这里将这些问题做一下记录.

比如,如下错误,是因为freemarker模板中出现了一个全角的中文空格,导致解析出现失败.
```
Error:(1, 1) java: 非法字符: '\ufeff'
```

### 需求
需求是,从freemarker中构造出一个Json,然后将这个json进行url编码,放到一个url后面,用户点击这个url,则立即可以跳转到指定的页面,页面再根据这个参数做相应的解析.

### 实现
在freemarker中,是通过拼凑Json字段进行处理的.在拼凑Json字段的过程中,会涉及字段的转义:
```
event.params.group?j_string
```
以及urlencode:
```
<#assign url_param="search=${search?url}" />
```

还有特别重要的是,有些字段需要判空,判空有好几种形式:
```
<#--条件判空-->
<#if event.params.processName?? >
       		     	<#assign processName = event.params.processName?j_string /> 
</#if> 

<#--感叹号判空-->
<#assign filePath = (event.params.filePath!"")?j_string />

<#--为空,则展示为空串-->
${(grouplist)!}
```


整体例子:
```
<#macro detailLinkParams aggregateAgentEvents>
        <#setting url_escaping_charset='utf8'/>
        <#if (aggregateAgentEvents?size>0) > 
		<#--连接进程-->
		<#assign processName="" />
		<#--业务组-->
		<#assign group=[] />
		<#--主机名-->
		<#assign hostname="" />
		<#--端口-->
		<#assign targetPort="" />
		<#--目标主机-->
		<#assign targetIpLike="" />
		<#--主机IP-->
		<#assign ip="" />
		<#--时间区间-->
		<#assign time_max = aggregateAgentEvents[0].params.dataTime/>
		<#assign time_min = aggregateAgentEvents[0].params.dataTime/>
                <#list aggregateAgentEvents as event> 
		     <#if event.params.group?? && !(group?seq_contains(event.params.group?j_string)) > 
		     	<#assign group = group + [event.params.group?j_string] />
		     </#if> 
		     <#if event.params.processName?? >
       		     	<#assign processName = event.params.processName?j_string /> 
		     </#if>         
		     <#if event.params.hostname?? >              
		     	<#assign hostname = event.params.hostname?j_string />  
		     </#if>
		     <#if event.params.targetPort?? >    
		     	<#assign targetPort = event.params.targetPort?j_string />
		     </#if>
		     <#if event.params.targetIp?? >    
		     	<#assign targetIpLike = event.params.targetIp?j_string />
		     </#if>
		     <#if event.params.displayIp?? >    
		     	<#assign ip = event.params.displayIp?j_string />
		     </#if>
		     <#if (time_max < event.params.dataTime) >
                            <#assign time_max = event.params.dataTime />
                     </#if>
                     <#if (time_min > event.params.dataTime) >
                            <#assign time_min = event.params.dataTime />
                     </#if>
                </#list>
                <#assign time_max = time_max + 1/>
		<#assign time_min = time_min - 1/>
                <#assign time="{\"min\":\"${(time_min*1000)?number_to_datetime?string('yyyy-MM-dd HH:mm:ss')}\",\"max\":\"${(time_max*1000)?number_to_datetime?string('yyyy-MM-dd HH:mm:ss')}\"}"/>
                <#assign grouplist = group?join(",") />
		<#if (aggregateAgentEvents?size==1) >
			<#assign search="{\"processName\":\"${(processName)!}\",\"createTime\":${time},\"group\":[${(grouplist)!}],\"ip\":\"${(ip)!}\",\"targetIpLike\":\"${(targetIpLike)!}\",\"targetPort\":${(targetPort)!},\"hostname\":\"${(hostname)!}\"}"/>
		<#else>
			<#assign search="{\"processName\":\"${(processName)!}\",\"createTime\":${time},\"group\":[${(grouplist)!}],\"ip\":\"${(ip)!}\",\"targetIpLike\":\"${(targetIpLike)!}\",\"targetPort\":${(targetPort)!},\"hostname\":\"${(hostname)!}\"}"/>
		</#if>
                <#assign url_param="search=${search?url}" />
	<#else>
		<#assign url_param="a=1"/>
	</#if>
${url_param}</#macro>
```

### 额外的问题

FreeMarker遍历集合集合时貌似有个要求，集合本身必须是Collection的子类；如果将jackson的JsonNode对象，扔给Freemarker则可能会有问题，例如：

```
	{
    "vuls": [
        {
            "vuln": 1,
            "port": 6379,
            "title": "xxx",
            "checkid": 7,
            "msg": "xxx!",
            "hacked_file": [
                {
                    "info": "xxx",
                    "path": "xxx"
                }
            ],
            "pid": 1351
        }
		]
}
```

如果通过如下模板进行解析，则会出现问题，因为json对象中的数组，无法用freemarker中的list遍历，这点很奇怪。

```
XXX ：
<#list vuls as vul>
 <#list vul.hacked_file as item>
  xx：${item.path},xxx：${item.info}
 </#list>
</#list>
```

但是可以通过把JsonNode当做字符串处理，在freemarker中用eval进行处理，则没问题：

```
<#function parseJSON json>
  <#local null = 'null'> <#-- null is not a keyword in FTL -->
  <#return json?eval>
</#function>
<#assign jsonObj=parseJSON(vuls)/>
XXX ：
<#list jsonObj as vul>
 <#list vul.hacked_file as item>
  xx：${item.path},xxx：${item.info}
 </#list>
</#list>
```

### 参考资料
https://stackoverflow.com/questions/46154391/how-can-i-iterate-a-collection-twice-in-freemarker#

https://stackoverflow.com/questions/17778844/evaluate-json-with-null-value-using-freemarker

https://stackoverflow.com/questions/12708162/how-to-get-json-into-a-freemarker-template-ftl