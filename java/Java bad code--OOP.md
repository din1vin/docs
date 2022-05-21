在实际开发过程中,我们有很多看似''面向对象'的把所有代码塞到一个类里边,实则"面向过程"的编程.

如果有意为之,并无不妥,而有些无意为之,则会影响到代码风格,希望以后多加注意吧.

## 违背面向对象原则的代码

### 1. 滥用getter,setter方法

在实际业务过程中,很多时候定义好一个类属性之后,顺手就把getter和setter方法全部定义上,甚至直接用IDE生成或者Lombok插件.

原以为无伤大雅的做法, 其实违背了面向对象的**封装**特性.

举例

```java
public class ShoppingCart{
    private int itemsCount; 
    private double totalPrice; 
    private List<ShoppingCartItem> items = new ArrayList<>();     
    
    public int getItemsCount() { return this.itemsCount; } 
    public void setItemsCount(int itemsCount) { this.itemsCount = itemsCount; } 
    public double getTotalPrice() { return this.totalPrice; } 
    public void setTotalPrice(double totalPrice) { this.totalPrice = totalPrice; } 
    public List getItems() { return this.items; } 
    public void addItem(ShoppingCartItem item) { 
        items.add(item); 
        itemsCount++; 
        totalPrice += item.getPrice(); 
    }
    
    // ....省略其他方法...
}
```

上边的类是一个购物车的抽象,购物车有商品总价,商品总件数,商品详情三个基本属性.

根据实际经验,用户对于购物车的操作,只有添加商品和移除商品,但是在暴露了所有属性的方法之后,itemsCount和totalPrice也是被外部随意修改的属性,跟使用public就没有区别了.而且如果任何代码都可以随意修改其中的某个值,也会导致跟items的属性不一致.

而面向对象封装的定义是: <u>通过访问权限控制,隐藏内部数据,外部仅能通过类提供有限的接口访问、修改内部数据</u>所以数据没有访问权限控制任何代码都可以随意修改,代码就退化成了面向过程风格的了.

除了上述的itemsCount和totalPrice外,items虽然没有暴露set方法,但是因为items本身是集合容器的引用,其他代码通过get方法获取到items,还是可以随意修改实际数据,这也是集合容器不安全的特点带来的隐患.

要解决这个问题,只需要用Collections.unmodifiableList来返回即可,然后列表内部的对象仍然可以修改,这个时候便可以clone一个对象返回了.

### 2. 滥用全局变量和全局方法

在面向对象编程中，常见的全局变量有单例类对象、静态成员变量、常量等，常见的全局方法有静态方法。单例类对象在全局代码中只有一份，所以，它相当于一个全局变量。静态成员变量归属于类上的数据，被所有的实例化对象所共享，也相当于一定程度上的全局变量。而常量是一种非常常见的全局变量，比如一些代码中的配置参数，一般都设置为常量，放到一个 Constants 类中。静态方法一般用来操作静态变量或者外部数据。你可以联想一下我们常用的各种 Utils 类，里面的方法一般都会定义成静态方法，可以在不用创建对象的情况下，直接拿来使用。静态方法将方法与数据分离，破坏了封装特性，是典型的面向过程风格。

#### 2.1 定义一个大而全的Constant类

```java

public class Constants {
  public static final String MYSQL_ADDR_KEY = "mysql_addr";
  public static final String MYSQL_DB_NAME_KEY = "db_name";
  public static final String MYSQL_USERNAME_KEY = "mysql_username";
  public static final String MYSQL_PASSWORD_KEY = "mysql_password";
  
  public static final String REDIS_DEFAULT_ADDR = "192.168.7.2:7234";
  public static final int REDIS_DEFAULT_MAX_TOTAL = 50;
  public static final int REDIS_DEFAULT_MAX_IDLE = 50;
  public static final int REDIS_DEFAULT_MIN_IDLE = 20;
  public static final String REDIS_DEFAULT_KEY_PREFIX = "rt:";
  
  // ...省略更多的常量定义...
}
```

会有什么问题呢?

1. 影响可维护性

   如果参与开发同一个项目的工程师很多,开发过程中的常量都往这个类里边添加, 容易造成代码提交冲突的概率,而且随着这个类越来越大,查找某个变量会变得费事,

2. 增加代码的编译时间

​		很多类都依赖Constant的话, 每次修改Constant会造成大量的类重新编译,浪费很多不必要的编译时间,进而可能影响开发效率.

3. 影响代码的可复用性

   如果我们要在另一个项目中，复用本项目开发的某个类，而这个类又依赖 Constants 类。即便这个类只依赖 Constants 类中的一小部分常量，我们仍然需要把整个 Constants 类也一并引入，也就引入了很多无关的常量到新的项目中。



如何改进这种设计呢?

1. 将大而全的Constant类拆解成功能单一的多个类,比如mysql相关放到MysqlConstant,redis相关放到RedisConstant
2. 不设计常量类,哪个类用到了某个常量,就把常量定义到这个类中,提高了类设计的内聚性和代码的复用性.



#### 2.2 对于Utils类的思考

Utils 类的出现是基于这样一个问题背景：如果我们有两个类 A 和 B，它们要用到一块相同的功能逻辑，为了避免代码重复，我们不应该在两个类中，将这个相同的功能逻辑，重复地实现两遍。这个时候我们该怎么办呢？

解决代码复用的方法,在java语言中很容易想到继承,只要定义一个父类,让AB都去继承它就可以了,但是java本身就是单继承,如果本身继承了其他类,也就没办法定义公共父类(java8 接口的静态方法能解决这个问题),而且AB两个类本身就不是一种类型的抽象,很难定义继承关系,比如Crawler和PageAnalyzer都需要URL的拼接和分割,但是爬虫跟页面解析明显很难定义一个父类,如果仅仅是为了代码复用,硬生生抽出一个父类.会影响到代码的可读性.

既然继承不能解决这个问题，我们可以定义一个新的类，实现 URL 拼接和分割的方法。而拼接和分割两个方法，不需要共享任何数据，所以新的类不需要定义任何属性，这个时候，我们就可以把它定义为只包含静态方法的 Utils 类了。实际上，只包含静态方法不包含任何属性的 Utils 类，是彻彻底底的面向过程的编程风格。但这并不是说，我们就要杜绝使用 Utils 类了。实际上，从刚刚讲的 Utils 类存在的目的来看，它在软件开发中还是挺有用的，能解决代码复用问题。所以，这里并不是说完全不能用 Utils 类，而是说，要尽量避免滥用，不要不加思考地随意去定义 Utils 类。

们设计 Utils 类的时候，最好也能细化一下，针对不同的功能，设计不同的 Utils 类，比如 FileUtils、IOUtils、StringUtils、UrlUtils 等，不要设计一个过于大而全的 Utils 类。