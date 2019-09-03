#### You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'order, org_id, start_time) values ('', '#409EFF', null, null, '11:30', null, nul' at line 1



```mysql
insert into yy_shift (code, color, create_time, create_user, order, end_time, last_update_time,
last_update_user, name, org_id, start_time) values (NULL, '#409EFF', '09/03/2019 14:56:44.829',
NULL, NULL, '14:56', '09/03/2019 14:56:44.829', NULL, '早班', NULL, '14:56')
```

表结构如下:

```
-- ----------------------------
-- Table structure for yy_shift
-- ----------------------------
DROP TABLE IF EXISTS `yy_shift`;
CREATE TABLE `yy_shift` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `org_id` bigint(20) DEFAULT NULL COMMENT '机构ID',
  `code` varchar(50) DEFAULT NULL COMMENT '班次代码',
  `name` varchar(50) NOT NULL COMMENT '班次名称',
  `start_time` varchar(50) NOT NULL COMMENT '班次开始时间',
  `end_time` varchar(50) NOT NULL COMMENT '班次结束时间',
  `color`  varchar(20) DEFAULT NULL COMMENT '显示颜色',
  `order`  varchar(20) DEFAULT NULL COMMENT '显示顺序',
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建用户',
  `last_update_time` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  `last_update_user` bigint(20) DEFAULT NULL COMMENT '最后更新用户',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```



这样的语句会报语法错误，为什么？

因为***order是保留字段！！！***

