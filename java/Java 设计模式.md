# Java 设计模式

## OOP七大原则

1. **开闭原则**: 对扩展开放,对修改关闭;
2. **里式替换原则**:继承必须保证超类所拥有的性质在子类中仍然成立;
3. **依赖倒置原则**:面向接口编程,不要面向实现编程;
4. **单一职责原则**:控制类的粒度大小,将对象解耦,提高内聚性;
5. **接口隔离原则**:为各个类提供它们需要的接口;
6. **迪米特原则**:只与朋友交谈,不跟"陌生人"说话;(缺点是容易建立很多中间类,增加系统复杂性)
7. **合成复用原则**:尽量先用组合或聚合等关联关系实现,其次再考虑继承;

### 单例模式

单例模式的重要思想是:==构造器私有化==,使得新建对象的时候无法立刻获得实例对象;

#### 饿汉式

> 饿汉式可以立刻获得对象实例,如同"饿汉"一样;
>
> ```java
> public class HungrySingleton{
>   	//构造方法私有化
>   	private HungrySingleton(){     
>     }
>   
>   	//保证对象实例唯一性,类加载的时候立即新建一个静态类
>   	private final static HungrySingleton HUNGRY = new HungrySingleton();
>   	
>   	public static HungrySingleton getInstance(){
>       	return HungrySingleton;
>     }
> }
> ```
>
> 由于饿汉式类是在编译完成后加载的(还没创建类实例的时候已经为其分配资源),所以可能会造成*内存浪费*,为了避免内存浪费,所以有了懒汉式;

#### 懒汉式

>懒汉式是为了避免饿汉式造成的内存浪费,可以理解成饿汉式的优化:
>
>```java
>public class LazySingleton{
>  //构造方法私有化
>  	private LazySingleton(){ 
>    }
>  
>  	//不分配资源
>  	private static LazySingleton lazy;
>  	
>  	public static LazySingleton getInstance(){
>      	if(null==lazy){
>        	lazy = new LazySingleton();
>      	}
>      	return lazy;
>    }
>}
>```
>
>