---
layout:     post
title:      "记一次重写RequestMappingHandlerMapping的经历"
category:   security
date:       2019-06-17 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Spring
---

>近期公司的产品做了一次安全审查，发现后端提供的接口有**不安全的Http方法**漏洞。不安全的HTTP方法一般包括：TRACE、PUT、DELETE、COPY 等。其中最常见的为TRACE方法可以回显服务器收到的请求，主要用于测试或诊断，恶意攻击者可以利用该方法进行跨站跟踪攻击（即XST攻击），从而进行网站钓鱼、盗取管理员cookie等。

## 原因分析
引起这个问题的原因其实很简单，因为开发人员开发接口的时候偷懒没有指定RequestMapping中的method属性导致的。没有指定method则系统会默认支持除了TRACE之外的其他７中方式，所以，这就是我要重写的RequestMappingHandlerMapping原因。

当然还有其他的方式，比如搜索出所有采用RequestMapping而没有指定method的地方，然后在代码中明确指定method。这种方式思路最简单，但是毕竟人是懒惰的。而且将来代码开发中如果任然有人不指定method，那么这个问题任然会存在。所以我们需要改动一下，框架加载http接口映射关系的逻辑：

在没有指定method的情况下，只支持GET和POST；在明确指定method的情况下，保留指定的method。

## 为什么重写的是RequestMappingHandlerMapping
首先后端提供的Http请求均是通过RestController和RequestMapping(或者衍生的GetMapping/PostMapping等)来实现的。那在接收http请求之后，如何将http请求映射到具体的Controller中的方法上呢？答案在HandlerMapping这个接口。这个具体映射过程细节以后再说，而HandlerMapping接口的映射方法getHandler是在AbstractHandlerMapping中实现的，而AbstractHandlerMapping的一个非抽象子类就是RequestMappingHandlerMapping。

## 重写RequestMappingHandlerMapping的哪个方法
从RequestMappingHandlerMapping中的getMappingForMethod方法可以看出，接口的映射信息，是由两部分组成的。一部分是来自于Controller类上的RequestMapping注解，一部分是来自于方法上的RequestMapping注解。所以，在这个方法中createRequestMappingInfo调用了两次，然后再组合。
```
@Override
	protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
		RequestMappingInfo info = createRequestMappingInfo(method);
		if (info != null) {
			RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
			if (typeInfo != null) {
				info = typeInfo.combine(info);
			}
		}
		return info;
	}
```

真正的createRequestMappingInfo有一个代理方法。

```
private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
		RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
		RequestCondition<?> condition = (element instanceof Class ?
				getCustomTypeCondition((Class<?>) element) : getCustomMethodCondition((Method) element));
		return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);
	}
```

在createRequestMappingInfo中，找到我们想要改动的地方了。在RequestMappingInfo设置method的时候增加一段自己的逻辑即可。

```
protected RequestMappingInfo createRequestMappingInfo(
			RequestMapping requestMapping, RequestCondition<?> customCondition) {

		return RequestMappingInfo
				.paths(resolveEmbeddedValuesInPatterns(requestMapping.path()))
				.methods(requestMapping.method())
				.params(requestMapping.params())
				.headers(requestMapping.headers())
				.consumes(requestMapping.consumes())
				.produces(requestMapping.produces())
				.mappingName(requestMapping.name())
				.customCondition(customCondition)
				.options(this.config)
				.build();
	}
```

至此,可以重写RequestMappingHandlerMapping如下：

```
public class HttpSafetyRequestMappingHandlerMapping extends RequestMappingHandlerMapping{

    private RequestMappingInfo.BuilderConfiguration config = new RequestMappingInfo.BuilderConfiguration();

    @Override
    protected RequestMappingInfo createRequestMappingInfo(RequestMapping requestMapping,
                                                          RequestCondition<?> customCondition) {
        // 如果Controller的方法上RequestMapping没有指定Method，则只支持GET和POST
        RequestMethod[] methods = { RequestMethod.GET, RequestMethod.POST };

        if(requestMapping.method().length != 0){
            methods = requestMapping.method();
        }

        return RequestMappingInfo
                .paths(resolveEmbeddedValuesInPatterns(requestMapping.path()))
                .methods(methods)
                .params(requestMapping.params())
                .headers(requestMapping.headers())
                .consumes(requestMapping.consumes())
                .produces(requestMapping.produces())
                .mappingName(requestMapping.name())
                .customCondition(customCondition)
                .options(config)
                .build();
    }

    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo info = createRequestMappingInfo(method);
        if (info != null) {
            RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
            if (typeInfo != null) {
                info = typeInfo.combine(info);
            }
        }
        return info;
    }

    private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
        RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
        RequestCondition<?> condition = (element instanceof Class ?
                getCustomTypeCondition((Class<?>) element) : getCustomMethodCondition((Method) element));
        if(requestMapping == null){
            return null;
        }
        // 只需要处理方法上的RequestMapping，Controller类上的不需要处理
        if(element instanceof Class){
            return super.createRequestMappingInfo(requestMapping, condition);
        } else {
            return createRequestMappingInfo(requestMapping, condition);
        }
    }

    @Override
    public void afterPropertiesSet() {
        this.config = new RequestMappingInfo.BuilderConfiguration();
        this.config.setUrlPathHelper(getUrlPathHelper());
        this.config.setPathMatcher(getPathMatcher());
        this.config.setSuffixPatternMatch(useSuffixPatternMatch());
        this.config.setTrailingSlashMatch(useTrailingSlashMatch());
        this.config.setRegisteredSuffixPatternMatch(useRegisteredSuffixPatternMatch());
        this.config.setContentNegotiationManager(getContentNegotiationManager());
        super.afterPropertiesSet();
    }
}
```

## 如何替换
网关服务采用Spring boot开发，那么我们可以从`org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration`入手。此类中有一个内部类：EnableWebMvcConfiguration，它继承并重写了createRequestMappingHandlerMapping方法。我们知道，在WebMvcConfigurationSupport类中，RequestMappingHandlerMapping实例的获取是通过其：requestMappingHandlerMapping方法上增加Bean注解来实现的。

这几个类的关系：
```
WebMvcConfigurationSupport
^
| 继承
DelegatingWebMvcConfiguration
^
| 继承
EnableWebMvcConfiguration
^      ^ 
|      | @Import
|　WebMvcAutoConfigurationAdapter
|　　　　　　　 ^ 静态内部类
|             |
| 静态内部类　　|
WebMvcAutoConfiguration

```

在WebMvcConfigurationSupport的requestMappingHandlerMapping方法中，RequestMappingHanlderMapping是通过调用createRequestMappingHandlerMapping方法来实现的。所以要覆盖RequestMappingHanlderMapping的实现，只需要重写createRequestMappingHandlerMapping即可。而EnableWebMvcConfiguration已经将这个方法重写如下：

```
        @Override
		protected RequestMappingHandlerMapping createRequestMappingHandlerMapping() {
			if (this.mvcRegistrations != null
					&& this.mvcRegistrations.getRequestMappingHandlerMapping() != null) {
				return this.mvcRegistrations.getRequestMappingHandlerMapping();
			}
			return super.createRequestMappingHandlerMapping();
		}
```
从上面的代码可以看出，EnableWebMvcConfiguration已经提供了替换RequestMappingHandlerMapping的方式，那就是：WebMvcRegistrations。而WebMvcRegistrations是个接口，并有一个默认的空实现：WebMvcRegistrationsAdapter，只需要覆盖其中的getRequestMappingHandlerMapping方法即可。

```
@Component
public class SafetyWebMvcRegistrationsAdapter extends WebMvcRegistrationsAdapter{

    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new HttpSafetyRequestMappingHandlerMapping();
    }
}
```
至此Sping boot中替换RequestMappingHandlerMapping的实现就结束了。


不过我们另一个服务采用的是Spring mvc,上面的替换就不生效了。因为服务中WebApplicationContext是通过@EnableWebMvc注入的配置。而在这个注解中导入的实际上是DelegatingWebMvcConfiguration。

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```
所以把@EnableWebMvc替换成@Import(WebMvcAutoConfiguration.EnableWebMvcConfiguration.class)即可。或者重写DelegatingWebMvcConfiguration，覆盖createRequestMappingHandlerMapping方法也是可以的。
