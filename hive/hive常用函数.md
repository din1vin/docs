##  一、数值比较

### 1. 等值比较 : =与<=>

**说明**: = 和 <=> 都是 判断A,B两个值是否相等,在AB都不为空的时候二者效果一样

**不同点**: AB都为null的时候 <=>返回True, = 返回False

### 2. 不等值比较: != 与 <>

**说明**: 与传统数据库语言一致

### 3. 大于、小于、大于等于、小于等于: > ,< ,>=,<=

**说明**: 与传统数据库语言一致

### 4. 区间比较:

**语法**: A [not] between B and C

**说明**: 等价于 B<=A<C

### 5. 空值判断: is null , is not null

 **说明**: 与传统数据库语言一致

### 6. like

**描述:** 如果字符串 A 或者字符串 B 为 NULL，则返回 NULL;如果字符串 A 符合表达式 B 的正则语法，则为 TRUE;否则为 FALSE。B 中字符”_”表示任意单个字符，而字 符”%”表示任意数量的字符。

**举例**:

```sql
hive> select 1 from lxw1234 where 'football' like 'foot%';
 1
hive> select 1 from lxw1234 where 'football' like 'foot__'; 
1
```

> 注意: 否定时候用 NOT A like B.

### 7. 正则like: rlike 和 regexp

**说明**: 符合java 正则表达式的写法

**举例**: 判断一个字符串是否全为数字:

```sql
 hive> select 1 from lxw1234 where '123456' rlike '^\\d+$'; 

1

hive> select 1 from lxw1234 where '123456' regexp '^\\d+$'; 

1
```

## 二、数学操作

数学操作的 AB两个都必须是数值类型

### 1. 加减乘除: +,-,*,/

**说明**: 加减类型 int ± int = int,  int ± double = double

乘法结果类型是AB的最小父类型(详情见数据类型的继承关系)

除法的结果是 double.

> 注意:hive 中最高精度的数据类型是 double,只精确到小数点后 16 位，在做除法运算的 时候要特别注意

### 2. 取余数: %

**说明**: 取余结果类型是AB的最小父类型(详情见数据类型的继承关系)

### 3. 位操作与、或、非、异或: &,|,~,^

**说明**: 与数学逻辑一致.

## 三、 逻辑操作

### AND(&&), OR(||),NOT(!)

**说明:**与主流数据库一致

## 四、复合类型构造函数

### 1. array

**语法**: array(v1,v2,v3...)

**操作类型**: array

```sql
hive> select array(1,2,3,4) from test;
OK
[1,2,3,4]
```

### 2. map

**语法 **: map(k1,v1,k2,v2,...)
**操作类型**: map
**说明**:使用给定的 key-value 对，构造一个 map 数据结构 

**举例**:

```sql
 hive> select map('k1','v1','k2','v2') from test;
 OK
 {"k2":"v2","k1":"v1"}
```

### 3. struct

**语法 **: struct(v1,v2,v3,...)
**操作类型**:struct

**举例**:

```sql
-- 默认构造是 col1,col2...
hive> select struct(1,'aaa',FALSE) from test; 
OK
{"col1":1,"col2":"aaa","col3":false}
-- 如果是已经定义的struct
hive> create table t(x struct<name:string,age:int> );
OK
hive> insert into t values(struct('tom',22) );
OK
hive> select * from t;
OK
{name:'tom', age: 22}
```

### 4. named_struct

**语法**: named_struct(k1,v1,k2,v2,...)

**操作类型**: sturct

**说明**: 类似map构造指定列名的struct

```sql
 hive> select map('k1','v1','k2','v2') from test;
 OK
 {"k2":"v2","k1":"v1"}
```

## 五、复合类型操作函数

### 1.  Array取值

**语法:** A[n]

**操作类型**: array

**说明**: 与Java一样从下标0开始取第n+1个元素.

**举例:**

```sql
hive> select array(1,2,3)[0] from test;
OK
1
```

### 2. Map取值

**语法**: A[k]

**操作类型:** Map

**说明**: 与Java一样按key取值,如果key不存在,返回null

**举例**:

```sql
hive> select map('k1','v1')['k1'] from test;
OK
v1
```

### 3. Struct取值

**语法:** A.k

**操作类型**: struct

**说明**: 返回A中k列的值

**举例**:

```sql
hive> select struct('a1','b2','c3').col1 from test;
OK
a1
```

### 4. Array长度

**语法:** size(Array a)

**说明:** 返回Array类型的长度

**举例:**

```sql
hive> select size(array(1,2,3,4)) from test;
OK
4
```

### 5. Map大小

**语法:** size(Map a)

**说明:** 返回map类型的a的长度

**举例:**

```sql
hive> select size(map('a',1,'b',2)) from test;
OK
2
```

### 6. 数组中是否包含元素

**语法:** array_contains(Array a, e)

**返回值:** boolean

**说明:** 返回a数组中是否包含e元素.

**举例:**

```sql
hive> select array_contains(array(1,2,3,4),5) from test;
OK
false
```

### 7. 获取map中的所有key

**语法:** map_keys(map a)

**返回类型:** array

**举例:**

```sql
hive> select map_keys(map('a',1,'b',2)) from test;
OK
['a','b']
```

### 8. 获取map中的所有value

语法: map_values(map a)

**返回类型:** array

**举例:**

```sql
hive> select map_values(map('a',1,'b',2)) from test;
OK
[1,2]
```

### 9. 数组排序

**语法**: sort_array(array a)

**说明:** 返回数组的增序排序结果

**返回类型: **array

**举例:**

```sql
select sort_array(array(3,2,4,1)) from test;
OK
[1,2,3,4]
```

## 六、数学计算函数

### 1. 四舍五入

**语法:** round(double A, scale=0)=bigint/double

**说明:** 对double的A类型进行四舍五入,scale是小数点后保留的位数,不指定时为0.scale不指定时返回bigint类型,指定精度时返回double类型.

**举例:**

```sql
hive> select round(3.14) from test;
OK
3

hive> select round(3.1415926,3) from test;
OK
3.142
```

### 2. 向下取整

**语法:** floor(double A)

**操作类型**: double

**返回值类型**: bigint

**举例**

```sql
hive> select floor(3.14) from test;
OK
3
```

### 3. 向上取整

**语法**: ceil(double A) 或 ceiling(double A)

**操作类型:** double

**返回值类型:** bigint

**举例:**

```sql
hive> select ceil(3.14) from test;
OK
4

hive> select ceiling(3.14) from test;
OK
4
```

### 4.  一以内随机数

**语法**: rand(),rand(int seed)

**返回值类型**: double

**说明**: 类似Java的random,如果指定seed,将得到稳定随机数

**举例**:

```sql
hive> select rand() from test;
0.7853598658412376
hive> select rand() from test;
0.6785479262413191
hive> select rand(100) from test;
0.7220096548596434
hive> select rand(100) from test;
0.7220096548596434
```

### 5. 自然指数e^x^

**语法:** exp(double x)

**返回类型:** double

**举例**:

```sql
hive> select exp(2) from test;
7.38905609893065
```

### 6. 自然对数 lnX

**语法:** ln(double x)

**返回类型:** double

**举例:**

```sql
hive> select ln(7.38905609893065) from test;
2.0
```

### 7. 常用对数 log~10~X

**语法**: log10(double x)

**返回类型:** double

**举例:**

```sql
hive> select log10(100) from test;
2.0
```

### 8. 以2为底对数 log~2~X

**语法:** log2(double x)

**返回类型:** double

**举例:**

```sql
hive> select log2(8) from test;
3.0
```

### 9. 指数函数 a^x^

语法 pow(double a,double x) 或者power(double a, double x)

返回值: double

举例:

```sql
hive> select pow(2,3) from test;
8.0
hive> select power(2,10) from test;
1024.0
```

### 10. 对数函数 log~n~a

**语法:** log(double n, double a)

**返回值:** double

**举例:**

```sql
hive> select log(3,27) from test;
3.0
```

### 11. 开方函数 ^n^√a

**语法:**  开根号(2次方)用sqrt(n),开n次方用 pow(a,1/n) 

**返回值:** double

**举例:** 

```sql
hive> select sqrt(16) from test;
4
```

### 12. 取余函数

**语法:** pmod(double x,double y)

**说明:** 返回 x%y的正余数

### 13. 取反函数

**语法:** negative( double x)

**说明:** 返回 -x

### 12. 三角函数

| 函数名   | 用法      | 输入类型   | 输出类型   |
| ----- | ------- | ------ | ------ |
| 正弦函数  | sin(x)  | double | double |
| 反正弦函数 | asin(x) | double | double |
| 余弦函数  | cos(x)  | double | double |
| 反余弦函数 | acos(x) | double | double |
| 正切函数  | tan(x)  | double | double |
| 反正切函数 | atan(x) | double | double |

## 七、进制转换

### 1. 十进制转二进制

**语法:** bin(bigint n)

**返回值:** string

**举例:** 

```sql
hive> select bin(7) from test;
111
```

### 2. 十进制转十六进制

**语法:** hex(bigint a)

**返回值:** string

**举例:**

```sql
hive> select hex(18) from test;
12
```

### 3. 普通进制转换

> 记住这一个通用的就行,其他的可以不知道

**语法:** conv(num, int from_base,int to_base)

**说明:** num可以是数值类型也可以是合法的string类型. 把num 从 from_base进制转换为to_base进制

**举例:**

```sql
 -- 将18从10进制转换成16进制
hive> select conv(18,10,16) from test;
12
```

### 4. 类型转换 cast

**语法:** cast(a as int)

**说明:** 将 a 转换成 as后面的类型并返回

**返回类型:** 指定的类型

**举例:** 

```sql
hive> select cast('1' as int) from test;
1
```

## 八、 时间日期函数

### 1. 时间戳转日期字符串

**语法:** from_unixtime(bigint tms,fmt='yyyy-MM-dd HH:mm:ss')

**说明**: 把时间戳转换成日期字符串格式,不指定格式的时候为yyyy-MM-dd HH:mm:ss

**举例:**

```sql
hive> select from_unixtime(1786587545) from test;
OK
2026-08-13 10:19:05

hive> select from_unixtime(1323308943,'yyyyMMdd') from test; 
OK
20111208
```

### 2. 获取当前时间戳

**语法:** unix_timestamp()

**返回类型:** bigint

**举例:**

```sql
hive> select unix_timestamp() from test;
OK
1632814841
```

### 3.日期转时间戳

**语法:** unix_timestamp(string date)

**返回类型:** bigint

**举例:**

```sql
hive> select unix_timestamp('2021-01-01 00:00:00');
OK
1609430400
```

### 4. 指定格式日期转时间戳

**语法:**unix_timestamp(string date,string pattern)

**返回类型:**bigint

**举例:**

```sql
hive> select unix_timestamp('2021-01-01','yyyy-MM-dd') from test;
OK
1609430400
```

### 5. 获取时间字符串的日期

**语法:** to_date(string dateString)

**返回类型:**string

**举例:**

```sql
hive> select to_date('2021-01-01 19:01:01') from test;
OK
2021-01-01
```

### 6. 获取时间字符串中的年份

**语法:** year(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select year('2021-01-01 19:01:01') from test;
OK
2021

hive> select year('2021-09-11') from test;
OK
2021
```

### 7. 获取时间字符串中的月份

**语法:** month(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select month('2021-01-01 19:01:01') from test;
OK
1

hive> select month('2021-09-11') from test;
OK
9
```

### 8. 获取时间字符串中的天

**语法:** day(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select day('2021-01-01 19:01:01') from test;
OK
1

hive> select day('2021-09-11') from test;
OK
11
```

### 9. 获取时间字符串中的小时

**语法:** hour(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select hour('2021-01-01 19:01:01') from test;
OK
19
```

### 10. 获取时间字符串中的分钟

**语法:** minute(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select minute('2021-01-01 19:01:01') from test;
OK
1
```

### 11. 获取时间字符串中的秒

**语法:** second(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select second('2021-01-01 19:01:01') from test;
OK
1
```

### 12. 获取时间字符串中的周

**语法:** weekofyear(string dateString)

**返回类型:** int

**举例:**

```sql
hive> select weekofyear('2021-10-10 19:01:01') from test;
OK
39
```

### 13. 日期相减

**语法**: datediff(string enddate,string startdate)

**返回值:** int

```sql
hive> select datediff('2021-12-01','2021-11-01') from test;
OK
30
```

参考: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-LogicalOperators