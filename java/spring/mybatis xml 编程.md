## 前言

最近加入了一个很牛叉的平台项目,据说支持n多的属性查询,高级分析,兴致勃勃的clone代码想看一下巧夺天工的sql生成设计模式的时候,干净的java code一下子给我打回现实. 不可能,绝对不可能,没有sql怎么实现ck的查询呢? 一看xml文件,好家伙,我直接好家伙,接口5个方法,xml配置了900多行,原来该来的在这来了.



截取一段代码感受一下这妖娆的俄罗斯传统工艺(套娃):

```xml
<sql id="single_event_query">
        SELECT
        <include refid="OtherMapper.date_time"/>
        AS x,
        <include refid="OtherMapper.selects"/>
        <include refid="OtherMapper.group_by"/>
        FROM
        <include refid="OtherMapper.presetTableName"/>
        AS logic_table
        <include refid="OtherMapper.join_expression"/>
        WHERE (
        ( <include refid="OtherMapper.presetSimpleTableName"/>
        BETWEEN #{query.eventView.startTime} AND #{query.eventView.endTime})
        )
        <if test="query.presetTableName == null or query.presetTableName ==''">
            <include refid="OtherMapper.filter_major_pn"/>
        </if>
        <trim prefix="AND (" suffix=")" prefixOverrides="AND">
            <if test="query.presetTableName == null or query.presetTableName ==''">
                <include refid="OtherMapper.filter_join_types"/>
                <include refid="OtherMapper.filter_event"/>
            </if>
            <if test="event.joinConditions != null and event.joinConditions.size() > 0">
                <foreach collection="event.joinConditions" item="join">
                    <choose>
                        <when test="join.tableId == 7">
                            AND dws_bi_device_properties.device_id != ''
                        </when>
                        <when test="join.tableId == 8">
                            AND dws_bi_user_properties.user_id != 0
                        </when>
                        <when test="join.tableId == 10">
                            AND dws_bi_apps.app_id != ''
                        </when>
                        <when test="join.tableId == 13">
                            AND dws_bi_moments.source_id != '' AND dws_bi_moments.source_type != ''
                        </when>
                        <when test="join.tableId == 16">
                            AND dws_channels.channel_name != ''
                        </when>
                    </choose>
                </foreach>
            </if>

            <include
                    refid="OtherMapper.filter_global"/>
            <include
                    refid="OtherMapper.filter_part"/>

            <if test="query.myKanbanParams != null and query.myKanbanParams.hasEventDate0 != null and query.myKanbanParams.hasEventDate0 == 1">
                and dwd_bi_events.event_object_id global in (
                select distinct app_id from tap_dwd.dwd_bi_app_operation_events
                where event_type_desc = #{query.myKanbanParams.eventTypeDesc} and source = 'upcomings'
                and event_date0 >= #{query.eventView.startTime}
                and event_date0 &lt;= #{query.eventView.endTime}
                )
            </if>

        </trim>
        GROUP BY x
        <include
                refid="OtherMapper.group_by_only_column_name"/>
    </sql>
```

抛开**可维护性和可读性**不谈,xml的动态性属实让人大吃一惊,虽然我不建议把sql搞成这样,但是功能还是可以了解一下的.

熟悉之后,四舍五入相当于又掌握了一门编程语言.



## 本质

xml配置sql,其实就是各种引用替换, bind ,ref, foreach,when, if等这些熟悉的关键字让它看起来赋值,引用,分支判断,编程语言需要的都有了.但本质上还是各种替换最终返回一个string. `#`和 `$`的灵活使用,也赋予了xml更多的动态可能,所以理论上所有的sql逻辑,都可以用xml来实现.



## 'XML 语言'

### 1. 赋值bind

编程语言都有赋值操作,xml语言也有, 那就是bind. xml bind允许用户定义一个变量,指向bind的值.

example:

```xml
<select id="getByType">
	<bind name="event" value="query.events[0]"/>
	select * from test_table where type = #{event.type}
</select>
```

java classes:

```java
public class queryDao{
   List<?> getByType(@Param("query") QueryPojo query);
   ...
}

@Data
public class QueryPojo{
    private List<Event> events;
    ...
}

@Data
public class Event{
    private String type;
    private String name;
}
```

可以看到QueryPojo是一个java类,类里边有个List,bind可以让sql直接使用List中第一个对象的成员属性.



### 2. 引用refid



这个比较好理解,就是在xml里边把其他地方的一块给拼过来,像不像重构时extract method的做法?

<include refid="xxx"/ >就可以实现了, 使用绝对路径 refid="com.pack1.other.XXX" 可以导入其他包id为XXX的片段.

```xml
<sql id="definedColumns">
    col_name1, col_name2, col_name3, col_name4....
</sql>

<select id="getAll">
    select <include refid="definedColumns"/> from test;
</select>

<insert id="insertOrdinary">
    insert into test(<include refid="definedColumns">) values (#{value1},#{value2}....)
</insert>
```



### 3.  if 逻辑判断

if是一个没有else的卫语句写法,类似于 if(condition){do something}的形式.

```xml
<insert id="insertByTp" >
    insert into table (tp,tp_cn) values (#{tp},
    <if test="tp=='user'">'用户'</if>
    <if test="tp=='device'">'设备'</if>)
</insert>
```

当tp是user和device之外的值,sql就会报错.

* test 里的多个且条件用and连接 或条件用 or 连接.

### 4. choose 逻辑判断

if是一个没有else的卫语句,如果要写一个有default 的case分支选择呢? 这个时候需要用到choose

```xml
<insert id="insertByTp" >
    insert into table (tp,tp_cn) values (#{tp},
    <choose>
        <when test="tp=='user'">'用户'</when>
        <when test="tp=='device'">'设备'</when>
        <otherwise>'其他'</otherwise>
    </choose>
</insert>
```



### 5. foreach 循环

