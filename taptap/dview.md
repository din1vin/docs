```sql
 CREATE TABLE `dview_user_tags` (
  `id` int unsigned NOT NULL AUTO_INCREMENT COMMENT '标签id',
  `created_at` timestamp default current_timestamp,
  `updated_at` timestamp default current_timestamp on update current_timestamp,
  `project` tinyint DEFAULT '1' COMMENT '1: 国内, 2:海外 ',
  `start_time` char(10) NOT NULL COMMENT '时间区间开始',
  `end_time` char(10) NOT NULL COMMENT '时间区间结束',
  `recent_day` varchar(15) DEFAULT NULL COMMENT '动态区间',
  `user_id` int DEFAULT NULL COMMENT '创建人id',
  `creator_name` varchar(127) DEFAULT NULL COMMENT '创建人名',
  `display_name` varchar(255) DEFAULT NULL COMMENT '标签名',
  `subject` varchar(127) DEFAULT 'user' COMMENT '标签类型 device: 设备标签 user: 用户标签',
  `cluster_name` varchar(255) DEFAULT NULL COMMENT '标签代码',
  `process` int DEFAULT NULL COMMENT '处理进度 0~100',
  `refresh_time` timestamp NULL DEFAULT NULL COMMENT '数据更新时间',
  `refresh_type` tinyint DEFAULT NULL COMMENT '更新方式 0:手动更新 1:自动更新',
  `tag_type` varchar(128) DEFAULT NULL COMMENT '标签类型',
  `remarks` varchar(512) DEFAULT NULL COMMENT '备注',
  `event` json DEFAULT NULL COMMENT '标签事件',
  `major` varchar(1024) DEFAULT NULL COMMENT '平台列表',
  `summary_result` json DEFAULT NULL COMMENT '标签统计(离散map)',
  `users_num` int DEFAULT NULL COMMENT '用户总人数',
  `range_type` tinyint DEFAULT '0' COMMENT '0:离散值 1:默认区间 2: 自定义区间',
  `range_prop` varchar(512) DEFAULT NULL COMMENT '自定义区间,有序数字序列用逗号分割',
  `range_result` json DEFAULT NULL COMMENT '用户设置的区间展示结果',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3
```

### 站内信息

```sql
    create table `dview_notifications`(
        `id` int auto_increment comment '主键ID',
        `project_id` tinyint not null comment '项目ID',
        `created_at` timestamp default current_timestamp,
        `updated_at` timestamp default current_timestamp on update current_timestamp,
        `user_id` int not null comment '创建人ID',
        `message_type` tinyint not null comment '消息类型[0-公告/1-更新消息/2-Dview使用Tips/3-功能说明/4-Features]',
        `title` varchar(1023) not null comment '消息标题',
        `content` text not null comment '消息主体',
        primay key(id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3

    create table `dview_user_notification`(
        `id` int auto_increment comment '主键ID',
        `user_id` int not null comment '用户id',
        `notification_id` int not null comment '通知Id',
        `status` tinyint not null default 0 comment '0:未读,1:已读,2:已删除',
        primary key(id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3
```



### mysqldump

```shell
mysqldump -t features -u bigdata -hrm-2zenz3ip6o87880sq.mysql.rds.aliyuncs.com -pU97DHtddvmoU3Vgu --tables  dview_users dview_roles dview_user_role
```

