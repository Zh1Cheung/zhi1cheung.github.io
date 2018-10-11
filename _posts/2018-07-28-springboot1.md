---
title: SpringBoot 2.x | 第二十篇：轻松搞定数据验证（二） 
categories:
- SpringBoot
tags:
- SpringBoot

---


当系统自带的注解无法满足我们的要求时候应该咋办呢？这就是本章将给各位介绍的**自定义 Validator 注解**

[](#为何要自定义 "为何要自定义")为何要自定义
==========================

`javax.validation` 包与 `hibernate-validator` 包中存在的注解几乎可以满足大部分的要求，又拥有基于正则表达式的`@Pattern`，为什么还需要自己去定义呢？

> 原因如下

*   正则效率不高
*   正则可读性不好
*   正则门槛较高，很多开发者并不会编写正则表达式

[](#本章目标 "本章目标")本章目标
====================

熟悉 `ConstraintValidator` 接口并且编写自己的数据验证注解

[](#具体代码 "具体代码")具体代码
====================

非常简单…

[](#导入依赖 "导入依赖")导入依赖
--------------------

在 `pom.xml` 中添加上 `spring-boot-starter-web` 的依赖即可

    <dependencies>  
     <dependency>  
     <groupId>org.springframework.boot</groupId>  
     <artifactId>spring-boot-starter-web</artifactId>  
     </dependency>  
     <dependency>  
     <groupId>org.springframework.boot</groupId>  
     <artifactId>spring-boot-starter-test</artifactId>  
     <scope>test</scope>  
     </dependency>  
    </dependencies>  

[](#自定义注解 "自定义注解")自定义注解
-----------------------

这里定义了一个 `@DateTime` 注解，在该注解上标注了 `@Constraint` 注解，它的作用就是指定一个具体的校验器类

> 关键字段（强制性）

*   **message：** 验证失败提示的消息内容
*   **groups：** 为约束指定验证组（非常不错的一个功能，下一章介绍）
*   **payload：** 不太清楚（欢迎留言交流）

    package com.battcn.annotation;  
      
      
    import com.battcn.validator.DateTimeValidator;  
      
    import javax.validation.Constraint;  
    import javax.validation.Payload;  
    import java.lang.annotation.Retention;  
    import java.lang.annotation.Target;  
      
    import static java.lang.annotation.ElementType.FIELD;  
    import static java.lang.annotation.ElementType.PARAMETER;  
    import static java.lang.annotation.RetentionPolicy.RUNTIME;  
      
    /**  
     * @author Levin  
     * @since 2018/6/6 0006  
     */  
    @Target({FIELD, PARAMETER})  
    @Retention(RUNTIME)  
    @Constraint(validatedBy = DateTimeValidator.class)  
    public @interface DateTime {  
      
     String message() default "格式错误";  
      
     String format() default "yyyy-MM-dd";  
      
     Class<?>[] groups() default {};  
      
     Class<? extends Payload>[] payload() default {};  
    }  

[](#具体验证 "具体验证")具体验证
--------------------

定义校验器类 `DateTimeValidator` 实现 `ConstraintValidator` 接口，实现接口后需要实现它里面的 `initialize：` 与 `isValid：` 方法。

> 方法介绍

*   **initialize：** 主要用于初始化，它可以获得当前注解的所有属性
*   **isValid：** 进行约束验证的主体方法，其中 `value` 就是验证参数的具体实例，`context` 代表约束执行的上下文环境。

这里的验证方式虽然简单，但职责明确；**为空验证可以使用 @NotBlank、@NotNull、@NotEmpty 等注解来进行控制，而不是在一个注解中做各种各样的规则判断，应该职责分离**

    package com.battcn.validator;  
      
    import com.battcn.annotation.DateTime;  
      
    import javax.validation.ConstraintValidator;  
    import javax.validation.ConstraintValidatorContext;  
    import java.text.ParseException;  
    import java.text.SimpleDateFormat;  
      
    /**  
     * 日期格式验证  
     *  
     * @author Levin  
     * @version 1.0.0  
     * @since 2018-06-06  
     */  
    public class DateTimeValidator implements ConstraintValidator<DateTime, String\> {  
      
     private DateTime dateTime;  
      
     @Override  
     public void initialize(DateTime dateTime) {  
     this.dateTime = dateTime;  
     }  
      
     @Override  
     public boolean isValid(String value, ConstraintValidatorContext context) {  
     // 如果 value 为空则不进行格式验证，为空验证可以使用 @NotBlank @NotNull @NotEmpty 等注解来进行控制，职责分离  
     if (value == null) {  
     return true;  
     }  
     String format = dateTime.format();  
     if (value.length() != format.length()) {  
     return false;  
     }  
     SimpleDateFormat simpleDateFormat = new SimpleDateFormat(format);  
     try {  
     simpleDateFormat.parse(value);  
     } catch (ParseException e) {  
     return false;  
     }  
     return true;  
     }  
    }  

[](#控制层 "控制层")控制层
-----------------

    package com.battcn.controller;  
      
    import com.battcn.annotation.DateTime;  
    import org.springframework.validation.annotation.Validated;  
    import org.springframework.web.bind.annotation.GetMapping;  
    import org.springframework.web.bind.annotation.RestController;  
      
    /**  
     * 参数校验  
     *  
     * @author Levin  
     * @since 2018/6/04 0031  
     */  
    @Validated  
    @RestController  
    public class ValidateController {  
      
     @GetMapping("/test")  
     public String test(@DateTime(message = "您输入的格式错误，正确的格式为：{format}", format = "yyyy-MM-dd HH:mm") String date) {  
     return "success";  
     }  
    }  

[](#主函数 "主函数")主函数
-----------------

    package com.battcn;  
      
    import org.springframework.boot.SpringApplication;  
    import org.springframework.boot.autoconfigure.SpringBootApplication;  
      
      
    /**  
     * @author Levin  
     */  
    @SpringBootApplication  
    public class Chapter19Application {  
      
     public static void main(String[] args) {  
     SpringApplication.run(Chapter19Application.class, args);  
     }  
    }  

[](#测试 "测试")测试
--------------

完成准备事项后，启动 `Chapter19Application` 自行测试即可，测试手段相信大伙都不陌生了，如 `浏览器`、`postman`、`junit`、`swagger`，此处基于 `postman`，如果你觉得自带的异常信息不够友好，那么配上[一起来学SpringBoot | 第十八篇：轻松搞定全局异常](http://www.iocoder.cn/Spring-Boot/battcn/v2-other-exception/) 可以轻松搞定…

> 错误格式

[![错误格式](http://image.battcn.com/article/images/20180606/springboot/v2-other-validate/3.png)](http://image.battcn.com/article/images/20180606/springboot/v2-other-validate/3.png)

> 正确格式

[![正确格式](http://image.battcn.com/article/images/20180606/springboot/v2-other-validate/4.png)](http://image.battcn.com/article/images/20180606/springboot/v2-other-validate/4.png)



