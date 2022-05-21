## 核心概念

IOC

AOP

Bean的生命周期





### Bean容器

如果容器太抽象,就理解成一个Map<beanName,bean对象>







### Bean 的生产工序

UserService类 ---> 构造方法 --->实例化对象 ---> **依赖注入**--->初始化前(@PostConstruct) ---> 初始化(InitializingBean) ----> 初始化后(**AOP**) --->UserService代理对象---> 放入容器  ---> Bean

构造方法推断,bean取值: byType->byName



### AOP

```sql
grant Select , Describe on table tap_db.tapdb_taplink_projects to USER RAM$xindong@aliyun.com:tapdb_oss_for_tapad;
```

