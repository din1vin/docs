# Spring Boot ASP 使用Kotlin自定义注解校验

## 背景概述

Spring Boot项目中需要自定义参数校验,自定义IP白名单等,如果在Controller的方法前面都加上这么一层,不仅很多方法用到同样的逻辑,代码显得冗余,不太优雅,即使用java写自定义注解,写ASP也有大量的代码工作,搜到一种Kotlin解决的思路,Kotlin的写法类似java,却少了很多代码量,是一个"相对优雅"的解决方案.



### POM

需要在pom中添加Kotlin的支持:

```xml

				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-reflect</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib-jdk8</artifactId>
            <version>${kotlin.version}</version>
        </dependency>

				<!-- 测试 支持-->
				<dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-test-junit</artifactId>
            <version>${kotlin.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-test</artifactId>
            <version>${kotlin.version}</version>
            <scope>test</scope>
        </dependency>
```





### 重点kotlin文件 HttpSercurityInterceptor.kt

```kotlin
package com.demo.component.filter

import org.aspectj.lang.annotation.Aspect
import org.aspectj.lang.annotation.Before
import org.springframework.stereotype.Component
import org.springframework.core.annotation.Order
import org.aspectj.lang.JoinPoint
import org.springframework.web.context.request.RequestAttributes
import org.springframework.web.context.request.RequestContextHolder

/**
* 自定义校验接口
*/
@Component
@Aspect
open class HttpSercirityInterceptor{
  companoin object{
    val sercretKey:String = PropertiesUtils.get("api.key","init")
  }
  
  @Before("execution(public * com.demo.component..*Controller.*(..)) && @annotation(org.springframework.web.bind.annotation.RequestMapping)")
  @Order(-1)
  open fun before(joiPoint:JoinPoint){
    val methodSignature = joinPoint.signature as MethodSignature
    val method = methodSignature.method
    
    val requestAttributes = RequestContextHolder.getRequestAttributes()
    val httpServletRequest: HttpServletRequest = requestAttributes?.resolveReference(RequestAttributes.REFRENCE_REQUEST) as HttpServletRequest
    limitIp(method, httpServletRequest)
    checkParamas(method, httpServletRequest)
  }
  
  //IP检测方法
  private fun limitIp(method: Method,httpServletRequest: HttpServletRequest){
    val ip: String = WebUtil.getIpAddr(httpServletRequest)
    //没有@Iplimit的方法直接return
    val ipLimit = method.getAnnotation(IpLimit::class.java)?:return
    
    //检测配置文件是否开启了ip限制
    if("false" == PropertiesUtil.get("ip.limit.enable","init")) return
    ipLimit.whiteList.map{
      PropertiesUtils.get("ip.rule.while_list.$it","init").split(",").map{_ip -> if(_ip==ip) return}
    }
    throw IllegalIpException("访问被拒绝 cause: ${ip}不在白名单内")
  }
  
  private fun checkParams(method:Method, httpServletRequest: HttpServletRequest){
    val paramChecker = method.getAnnotation(ParamsChecker::class.java) ? : return
    if (!paramsChecker.checker.check(httpServletRequest)) throw IllegalParamsException("访问被拒绝,参数不合法")
  }
}


/**
* IpLimit 注解
*/
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class IpLimit(val whiteList:Array<String>=["default"])

/**
* ParamChecker 注解
*/
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class ParamsChecker(val checker: Checker = Checker.DEFAULT)


/**
* 检查策略枚举类
*/
enum class Checker{
  DEFAULT{
    override fun check(httpServletRequest: HttpServletRequest): Boolean {
            return true
        }
  },
  CLASSICAL{
    override fun check(httpServletRequest: HttpServletRequest): Boolean {
            return !(!SecretConnectUtil.md5StrIsValid(secretKey, httpServletRequest) || !SecretConnectUtil.timeIsValid(httpServletRequest))
        }
  };
  
  abstract fun check(httpServletRequest: HttpServletRequest): Boolean
}
```

### 它带来了什么?

```java
@RestController
@RequestMapping("/demo")
public class DemoController{
  
  @RequestMapping("/simple")
  @ParamsChecker(check=Checker.SIMPLE)
  @Iplimit
  public Result simpleChecker(){
    Result result = new Result();
    result.setMsg("this method check with simple");
  }
}
```

只用添加注解就实现了检测ip和参数的逻辑,是不是很spring boot?