---
layout:     post
title:      "Spring学习之Profile和Lookup注解"
category:   Spring
date:       2019-07-02 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Spring
---

>在编码过程中遇到一个Profile和Lookup注解的问题,这里记录一下.Profile的原意是剖面的意思,Profile在官方的API文档中,是如此描述的:

>A profile is a named logical grouping that may be activated programmatically via ConfigurableEnvironment.setActiveProfiles(java.lang.String...) or declaratively by setting the spring.profiles.active property as a JVM system property, as an environment variable, or as a Servlet context parameter in web.xml for web applications. Profiles may also be activated declaratively in integration tests via the @ActiveProfiles annotation.

>通过Profile可以从逻辑上对Bean做一些分组,然后应用中通过配置来选中某一组Bean进行实例化.Lookup注解是用于实现,注入prototype-scoped类型的Bean,一般我们可能习惯用new的方式去实例化非单例模式的Bean.

### Profile注解示例
实现开发环境和生产环境采用不同的数据源.
```
// 开发环境采用mysql数据库
@Profile("Development")
@Configuration
public class DevDatabaseConfig implements DatabaseConfig {
 
    @Override
    @Bean
    public DataSource createDataSource() {
        System.out.println("Creating DEV database");
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        /*
         * Set MySQL specific properties for Development Environment
         */
        return dataSource;
    }
 
}

// 生产环境采用Oracle数据库
@Profile("Production")
@Configuration
public class ProductionDatabaseConfig implements DatabaseConfig {
 
    @Override
    @Bean
    public DataSource createDataSource() {
        System.out.println("Creating Production database");
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        /*
         * Set ORACLE specific properties for Production environment
         */
        return dataSource;
    }
 
}
```

除了采用注解,使用配置文件也可以达到同样的效果.
```
    <beans profile="Development">
        <import resource="dev-config-context.xml"/>
    </beans>
 
    <beans profile="Production">
        <import resource="prod-config-context.xml"/>
    </beans>
```

测试:

```
public class AppMain {
     
    public static void main(String args[]){
        AnnotationConfigApplicationContext  context = new AnnotationConfigApplicationContext();
        //Sets the active profiles
        context.getEnvironment().setActiveProfiles("Development");
        //Scans the mentioned package[s] and register all the @Component available to Spring
        context.scan("com.websystique.spring"); 
        context.refresh();
        context.close();
    }
 
}
```

### Lookup注解
lookup可以实现方法注入,即利用lookup可以让某个单例Bean的某个方法每次都返回一个新的Bean,即prototype类型的Bean.这是依赖CGlib技术实现的,在运行时,修改字节码.

利用xml文件实现:
```
<bean id="car" class="com.smart.injectfun.Car" p:brand="红旗CA72", p:price="2000" scope="prototype" />

<bean id="magicBoss" class="com.smart.injectfun.MagicBoss">
    <look-method name="getCar" bean="car">
<bean>
```

也可以通过注解来实现:

```
@Component
public class MagicBossIMpl implements MagicBoss {
    @Lookup
    public Car getCar(){
        return null;
    }
}
```

### 参考
https://www.baeldung.com/spring-lookup

http://websystique.com/spring/spring-profile-example/

