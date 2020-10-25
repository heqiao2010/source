---
layout:     post
title:      "Swagger"
category:   Swagger
date:       2019-06-06 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Api管理
---

## Swagger使用说明

### 功能简介
Swagger是一套治理API的工具，根据OAS（Open Api Specification）这个描述API的规则，实现API盘点、测试以及归档等功能。目前在wisteria功能中，可以在嵌入很少的侵入代码的情况下实现API的盘点功能——我们可以获取到一个最新的，随时和代码保持同步的API文档。

### 使用步骤 
1. 引入工程依赖

    compile "io.springfox:springfox-swagger2:2.9.2"
    
2. 添加API文档支持

在项目中，采用开开源的springfox支持API信息的采集。 

(1)非Spring MVC项目(可以跳过该步骤)
 
(2)Spring MVC项目

需要在MvcConfig中增加`@EnableSwagger2`注解，同时注入Bean`Docket`即可。

    @Configuration
    @EnableSwagger2
    @ComponentScan(basePackages = "com.xxx.frontend",
        nameGenerator = FullBeanNameGenerator.class)
    @EnableWebMvc
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public class MvcConfig extends WebMvcConfigurerAdapter{
        @Bean
        public Docket apiDocket() {
            return new Docket(DocumentationType.SWAGGER_2)
                    .enable(swaggerStatus)
                    .apiInfo(createDefaultApiInfo())
                    .groupName(appName + "frontend")
                    .select()
                    .apis(RequestHandlerSelectors.basePackage("com.xxx"))
                    .paths(PathSelectors.any())
                    .build();
        }
        private ApiInfo createDefaultApiInfo(){
            return new ApiInfoBuilder()
                    .title(appName)
                    .description(appName + " frontend api 文档")
                    .termsOfServiceUrl("http://www.xxx.cn")
                    .build();
        }
    }

(3)添加API描述

Springfox定义了一系列的注解，用于更好的描述各个API的内容，包括输入输出以及字段类型和限制等。如果不添加这些内容，API的信息也是可以获取到的，只是不是很全面而已。

描述Controller的注解：

  @Api describes the whole controller
  @ApiOperation is used for description on a methods level
  @ApiParam is used for method parameters

示例：

    @RestController
    @RequestMapping("/v2/persons/")
    @Api(description = "Set of endpoints for Creating, Retrieving, Updating and Deleting of Persons.")
    public class PersonController {
        private PersonService personService;
        @RequestMapping(method = RequestMethod.GET, path = "/{id}", produces = "application/json")
        @ApiOperation("Returns a specific person by their identifier. 404 if does not exist.")
        public Person getPersonById(@ApiParam("Id of the person to be obtained. Cannot be empty.")
                                        @PathVariable int id) {
            return personService.getPersonById(id);
        }
    }

描述Model的注解：

示例：

    {
        @ApiModel(description = "Class representing a person tracked by the application.")
        public class Person {
        @ApiModelProperty(notes = "Unique identifier of the person. No two persons can have the same id.", example = "1", required = true, position = 0)
        private int id;
        @ApiModelProperty(notes = "First name of the person.", example = "John", required = true, position = 1)
        private String firstName;
        @ApiModelProperty(notes = "Last name of the person.", example = "Doe", required = true, position = 2)
        private String lastName;
        @ApiModelProperty(notes = "Age of the person. Non-negative integer", example = "42", position = 3)
        private int age;
    
        // … Constructor, getters, setters, ...
    }
    

支持JSR-303

  @NotNull    不为空
  @NotBlank   不为空 不为空字符串
  @Size(min = 1, max = 20)  字符串长度范围
  @Min(0)       最小值最大值设置
  @Max(100)
  @Pattern(regexp = "[SOME REGULAR EXPRESSION]")   正则表达式设置  


做到上面几步就可以在api文档中看到效果了。类似如下：

![](https://www.vojtechruzicka.com/static/57c53482f716c4648ec5d4c860c2a0e2/30398/swagger-ui.png)

或者采用不同的展示模板：

![](https://static.oschina.net/uploads/space/2018/0716/075136_60JO_254762.png)


## Springfox原理

Springfox生成Api文档的原理是，通过获取Spring上下文中`@Controller`以及`@RequestMapping`注解的Bean的信息，然后通过层层解析，最终按照Swagger的标准生成Api文档而得来的。

从RequestMappingHandlerMapping的一段源码上面看，就能够理解在`@Controller`和`@ResquestMapping`注解到一个Bean或其属性上之后，Spring做了什么，是如何用HandlerMapping将Http请求映射到响应的Bean的方法上的，以及是如何采用HandlerAdapter来处理Http请求的。

RequestMappingHandlerMapping中的两个关键方法：

    /**
     * {@inheritDoc}
     * Expects a handler to have a type-level @{@link Controller} annotation.
     */
    @Override
    protected boolean isHandler(Class<?> beanType) {
    	return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
    			AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
    }
    
    private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
    	RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
    	RequestCondition<?> condition = (element instanceof Class ?
    			getCustomTypeCondition((Class<?>) element) : getCustomMethodCondition((Method) element));
    	return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);
    }
    

Springfox获取Controller Bean信息的类图：

![](https://raw.githubusercontent.com/heqiao2010/heqiao2010.github.io/master/img/2019/swagger.png )

参考：
https://www.vojtechruzicka.com/documenting-spring-boot-rest-api-swagger-springfox/

https://www.v2ex.com/t/493395
