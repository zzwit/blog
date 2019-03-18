### Mysql 分区

创建一个普通表

  ```
  CREATE TABLE `zr_sign_202901` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL COMMENT '用户ID',
  `t_day` int(2) NOT NULL COMMENT '天数',
  `appid` varchar(200) NOT NULL DEFAULT '' COMMENT 'app渠道',
  `coin` int(11) DEFAULT '0' COMMENT '金币',
  PRIMARY KEY (`id`,`user_id`),
  KEY `user_app_day` (`user_id`,`appid`,`t_day`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='签到log 只负责添加' 

  ```

创建一个分区表，按照user_id 进行取模分区 取模30个

```
CREATE TABLE `zr_sign_202901` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL COMMENT '用户ID',
  `t_day` int(2) NOT NULL COMMENT '天数',
  `appid` varchar(200) NOT NULL DEFAULT '' COMMENT 'app渠道',
  `coin` int(11) DEFAULT '0' COMMENT '金币',
  PRIMARY KEY (`id`,`user_id`),
  KEY `user_app_day` (`user_id`,`appid`,`t_day`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='签到log 只负责添加' 
partition BY HASH (`user_id`) PARTITIONS 30;
```

查看那个分区占用多少数据

```
SELECT PARTITION_NAME,PARTITION_METHOD,PARTITION_EXPRESSION,PARTITION_DESCRIPTION,TABLE_ROWS,SUBPARTITION_NAME,SUBPARTITION_METHOD,SUBPARTITION_EXPRESSION 
FROM information_schema.PARTITIONS WHERE TABLE_SCHEMA=SCHEMA() AND TABLE_NAME='zr_sign_202901';

```
