# 实时算子数据源--clickhouse

## 玩家数据

### 表结构说明

> 我们希望有一张大宽表,包含玩家尽可能多的信息,以**每个玩家**,**每天**为基础维度,玩家数据表规范如下:

| 列名            | 类型   | 是否可空 | 默认值  | 描述                     |
| --------------- | ------ | -------- | ------- | ------------------------ |
| game            | String | No       | 无      | 游戏名                   |
| unique_tag      | String | No       | 无      | 用户唯一标识             |
| dt              | String | No       | 无      | yyyy-MM-dd所有日期格式   |
| day_num         | int    | No       | 1       | 账号年龄/天数            |
| yys             | String | No       | 无      | 大区                     |
| server_name     | int    | No       | 无      | 服务器                   |
| player_id       | int    | No       | 无      | 玩家id                   |
| player_name     | String | No       | 无      | 角色名                   |
| yx              | String | No       | 无      | 联运                     |
| user_id         | String | No       | 无      | uid                      |
| source          | String | No       | default | 来源                     |
| min_lv          | int    | No       | 无      | 当日最低等级             |
| max_lv          | int    | No       | 无      | 当日最高等级             |
| vip             | int    | No       | 0       | vip等级,当天有变化取最大 |
| total_pay       | int    | No       | 0       | 玩家总充值               |
| create_dt       | String | No       | 无      | 玩家创建日期             |
| pay_dt          | String | Yes      | 无      | 首次付费日期             |
| high_dt         | String | Yes      | 无      | 玩家首次到达高质的日期   |
| pay             | int    | No       | 0       | 当天充值元宝数           |
| use             | int    | No       | 0       | 当天消费元宝数           |
| use_sys         | int    | No       | 0       | 当天消费系统金           |
| use_user        | int    | No       | 0       | 当天消费用户金           |
| max_power       | int    | No       | 无      | 当天最大关卡id           |
| active          | int    | No       | 0       | 当天活跃时长/分钟数      |
| online          | int    | No       | 0       | 当天在线时长/分钟数      |
| pay_first_month | int    | No       | 0       | 前30日充值               |
| pay_first_week  | int    | No       | 0       | 前7天充值                |
| last_pay_time   | String | Yes      | 无      | 最后付费时间             |
| last_login_time | String | Yes      | 无      | 最后登录时间             |