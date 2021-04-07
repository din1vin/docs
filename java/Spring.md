# Spring

* Spring Boot  快速开发脚手架
* Spring Cloud 基于Spring  Boot微服务架构

## Spring MVC

### IOC: 控制翻转思想

### IOC创建对象的原理

1. 默认使用无参构造创建对象

2. 有参构造方式:
   * index  : `<constructor-arg index="0" value="str1"/>`
   * 类型匹配 : `<constructor-arg type="java.lang.String" value="str2"/>`
   * id引用: `<constructor-arg name="str" value="str3"/>`

### xml配置

* 别名Alias: id标签后面,添加alias使用别名,使用别名也可以取到对象.

* scope: bean的作用域,`singleton`是唯一对象,`prototype`是全局模式.

  

