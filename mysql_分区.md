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

书籍表
```
CREATE TABLE `zr_merchants_books` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `mch_id` int(11) NOT NULL COMMENT '商户ID',
  `book_id` int(11) NOT NULL COMMENT '书籍ID',
  `start_time` datetime NOT NULL COMMENT '开始时间',
  `end_time` datetime NOT NULL COMMENT '结束时间',
  `status` tinyint(1) DEFAULT '0' COMMENT ' 1 开启 0 删除',
  `book_name` varchar(100) DEFAULT '' COMMENT '书籍名称',
  `author_name` varchar(50) DEFAULT '' COMMENT '作者名称',
  `gmt_create` datetime DEFAULT NULL COMMENT '添加时间',
  PRIMARY KEY (`id`,`mch_id`),
  KEY `mch_id_book_index` (`mch_id`,`book_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='商户书籍表'
 partition BY HASH (`mch_id`) PARTITIONS 30;
```
