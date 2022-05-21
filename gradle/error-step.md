### 1. Could not resolve all files for configuration ':common:compileClasspath'.

当项目里为显式指定repositories的时候,如果gradle配置也未指定,会报这个错误.

方法一:

项目的build.gradle 添加如下内容

```groovy
repositories{
    mavenCentral()
}
```



方法二:  

全局配置init.gradle: `vim ~/.gradle/init.gradle` 

```groovy


allprojects {                                                                                                                                                                                                                    
     repositories {
         mavenLocal()
         maven { url 'https://maven.aliyun.com/repository/central/' }
         maven { url 'https://maven.aliyun.com/repository/public/' }
         maven { url 'https://maven.aliyun.com/repository/google/' }
         maven { url 'https://maven.aliyun.com/repository/jcenter/'}
         maven { url 'https://maven.aliyun.com/repository/gradle-plugin/'}
         maven { url 'https://maven.aliyun.com/repository/spring/'}
         maven { url 'https://maven.aliyun.com/repository/spring-plugin/'}
         maven { url 'https://maven.aliyun.com/repository/grails-core/'}
         maven { url 'https://maven.aliyun.com/repository/apache-snapshots/'}
         mavenCentral()
     }
}
```

