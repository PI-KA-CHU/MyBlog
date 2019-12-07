+++
author = "pikachu"
title = "SpringBoot实现全局异常的捕获"
date = "2019-02-13"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"spring-boot"
]
categories = [
    "IT"
]
+++


## 1. 创建一个简单类
- 使用`@RestControllerAdvice`注册组件，使用`@ExceptionHandler`捕获异常
- 该类`只捕获Controller层的异常`，其他方法体的异常不予捕获
- 捕获到异常后会返回相应方法体的返回值

```
/**
 * 全局异常处理，仅对Controller层的异常有效
 *
 * @author 曾博佳
 * @since 2019-02-01
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    private Logger log = LoggerFactory.getLogger(this.getClass());

    /**
     * 统一捕获运行时异常
     *
     * @author 曾博佳
     * @since 2019-02-01
     * @param e 异常信息
     * @return 返回异常错误码及信息
     */
    @ExceptionHandler(Exception.class)
    public BaseReturnDto handleError(Exception e){
        log.error("服务器异常",e);
        return new BaseReturnDto(ReturnCodeEnum.SERVER_ERROR.getCode(),e.getMessage());
    }

    /**
     * 统一捕获404未找到异常
     *
     * @author 曾博佳
     * @since 2019-02-01
     * @param e 异常信息
     * @return 返回404码及异常信息
     */
    @ExceptionHandler(NoHandlerFoundException.class)
    public BaseReturnDto handler404Error(Exception e){
        log.error("页面未找到",e);
        return new BaseReturnDto(ReturnCodeEnum.NOT_FOUND.getCode(),e.getMessage());
    }
}

```

&nbsp;

## 2. 全局异常捕获类的注解介绍
- **@RestControllerAdvice** = `@ResponseBody` + `@ControllerAdvice`：是SpringBoot整合后的注解
- **@ResponseBody**：将返回结果以JSON格式输出
- **@ControllerAdvice**：是一个组件注解，它允许实现类通过类路径扫描被自动检测到。`@ControllerAdvice`注解的类可以包含带有`@ExceptionHandler`、`@InitBinder`和`@ModelAttribute`注解的方法，`@ExceptionHandler`用于异常的捕获，可以指定捕获的异常类型。

&nbsp;