---
layout:     post
title:      "ControllerAdvice注解"
category:   Spring
date:       2019-07-08 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Spring
---

### 基本功能

准确的来说,这个是springmvc中的一个注解.
从Advice后缀可以看出,这个注解是针对Controller的一个增强.在对外提供Http接口支持的时候,往往有很多通用的业务逻辑.最常见的是同样的异常处理逻辑.
为了避免大块的try-catch块的出现,可以通过此注解提供一个异常处理的增强来解决.

#### @ExceptionHandler

在ControllerAdvice类中标注某个方法,用于处理某个类型的异常.

例子:
```
/**
 * 统一处理网关接口异常
 *
 * Created by qiaohe on 19-6-27.
 */
@Component
@ControllerAdvice("com.*")
public class ExceptionAdvice {
    private final Logger logger = LoggerFactory.getLogger(ExceptionAdvice.class);

    /**
     * 处理AuthFailedException
     */
    @ExceptionHandler(AuthFailedException.class)
    @ResponseBody
    public BaseResult handleException(AuthFailedException ex){
        return BaseResult.failure(ex.getAuthFailedCode().getCode(),
                ex.getAuthFailedCode().getDesc());
    }
}

```

#### 其他两个不常用的注解

@InitBinder：用来设置WebDataBinder，用于自动绑定前台请求参数到Model中。

@ModelAttribute：本来作用是绑定键值对到Model中，此处让全局的@RequestMapping都能获得在此处设置的键值对。

列子:
```
@ControllerAdvice(basePackages = {"com.concretepage.controller"} )
public class GlobalControllerAdvice {
	@InitBinder
	public void dataBinding(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
		dateFormat.setLenient(false);
		binder.registerCustomEditor(Date.class, "dob", new CustomDateEditor(dateFormat, true));
	}
	@ModelAttribute
        public void globalAttributes(Model model) {
		model.addAttribute("msg", "Welcome to My World!");
        }
	@ExceptionHandler(FileNotFoundException.class)
        public ModelAndView myError(Exception exception) {
	    ModelAndView mav = new ModelAndView();
	    mav.addObject("exception", exception);
	    mav.setViewName("error");
	    return mav;
	}
} 
```

### 参考
https://www.concretepage.com/spring/spring-mvc/spring-mvc-controlleradvice-annotation-example