---
layout:     post
title:      "JUC共享锁之Semaphore"
category:   Java并发基础
date:       2019-06-24 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---

>Semaphore原意是指信号量,从API的注释:"Semaphores are often used to restrict the number of threads than can access some (physical or logical) resource"可以看出,Semaphore一般是用来限制线程能够使用的资源个数.

### 应用场景
在Web开发中,Semaphore可以用来限制某个接口的并发调用次数.可以在Sping的Context中维护一个Map,key可以是处理线程,value可以是一个Semaphore对象.通过这样的方式,就可以实现系统同时处理的线程数不会超过某个阈值.

### 代码实现
我们可以通过WebFilter注解,来实现一个过滤器,在过滤器中拦截所有的请求调用,然后通过Semaphore来进行计数,如果超过总的计数,则返回相应的提示信息.当然也可以对URL进行细化,针对每个API提供对应的限制.
```
/**
 * API并发控制过滤器
 * Created by qiaohe
 * Date: 19-7-1 上午11:54
 */
@Component
@WebFilter(urlPatterns = "/*", filterName = "concurrentRestrictFilter")
public class ConcurrentRestrictFilter implements Filter {
    private Log log = LogFactory.getLog(getClass());

    private static final Integer MAX_CONCURRENT_NUM = 1;

    private static final Semaphore semaphore = new Semaphore(MAX_CONCURRENT_NUM);

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("before acquire: " + semaphore.availablePermits());
        if(!semaphore.tryAcquire()){
            if(response instanceof HttpServletResponse){
                HttpServletResponse res = (HttpServletResponse)response;
                res.setContentType(MimeTypeUtils.APPLICATION_JSON_VALUE);
                res.sendError(HttpStatus.BAD_REQUEST.value(), "reach max current num");
            }
            return;
        }
        log.info("after acquire: " + semaphore.availablePermits());
        try{
            chain.doFilter(request, response);
        } finally {
            semaphore.release();
            log.info("release: " + semaphore.availablePermits());
        }
    }

    @Override
    public void destroy() {

    }
}
```
