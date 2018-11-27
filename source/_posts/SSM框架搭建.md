---
title: SSM框架搭建
date: 2018-11-20 09:49:17
tags: SSM
categories:
- SSM
---

## 1 写在前面

初学SSM(SpringMVC,Spring,MyBatis)框架的整合，并在整合后的框架上对数据库进行增删改查等操作，仅此记录自学过程，如有不正确的地方，欢迎指正，探讨！本文工程代码以及相关jar包在文末给出。

<!--more-->

## 2 SSM是干什么的

在搭建整体架构之前，我们必须清楚SSM框架各个部分的作用是什么。

**SpringMVC**
这是一个表现层的框架，SpringMVC在项目中拦截用户请求，它的核心Servlet即DispatcherServlet（前端控制器）承担中介或是前台这样的职责，将用户请求通过[HandlerMapping（处理器映射器）](https://baike.baidu.com/item/DispatcherServlet/12740507)去匹配Controller（控制器），Controller就是具体对应请求所执行的操作。简而言之，SpringMVC的作用就是从请求中接收传入的参数，将处理后的结果数据返回给页面展示，SpringMVC相当于SSH框架中struts。

**Spring**
Spring就像是整个项目中装配bean的大工厂，在配置文件中可以指定使用特定的参数去调用实体类的构造方法来实例化对象。Spring的核心思想是IoC（控制反转），即不再需要程序员去显式地`new`一个对象，而是让Spring框架帮你来完成这一切。


**MyBatis**
mybatis是对jdbc的封装，它让数据库底层操作变的透明。mybatis的操作都是围绕一个sqlSessionFactory实例展开的。mybatis通过配置文件关联到各实体类的Mapper文件，Mapper文件中配置了每个类对数据库所需进行的sql语句映射。在每次与数据库交互时，通过sqlSessionFactory拿到一个sqlSession，再执行sql命令。

页面发送请求给控制器，控制器调用业务层处理逻辑，逻辑层向持久层发送请求，持久层与数据库交互，后将结果返回给业务层，业务层将处理逻辑发送给控制器，控制器再调用视图展现数据。

## 3 整合思路

### 3.1 数据库
首先要做的是新建一个数据库CRM，并创建如下表单base_dict（这部分比较枯燥，可以粗略带过=。=）
``` bash
CREATE DATABASE CRM_practice;
USE CRM_practice;
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for base_dict
-- ----------------------------
DROP TABLE IF EXISTS `base_dict`;
CREATE TABLE `base_dict` (
  `dict_id` VARCHAR(32) NOT NULL COMMENT '数据字典id(主键)',
  `dict_type_code` VARCHAR(10) NOT NULL COMMENT '数据字典类别代码',
  `dict_type_name` VARCHAR(64) NOT NULL COMMENT '数据字典类别名称',
  `dict_item_name` VARCHAR(64) NOT NULL COMMENT '数据字典项目名称',
  `dict_item_code` VARCHAR(10) DEFAULT NULL COMMENT '数据字典项目代码(可为空)',
  `dict_sort` INT(10) DEFAULT NULL COMMENT '排序字段',
  `dict_enable` CHAR(1) NOT NULL COMMENT '1:使用 0:停用',
  `dict_memo` VARCHAR(64) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`dict_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```
给base_dict表单添加数据
``` bash
-- ----------------------------
-- Records of base_dict
-- ----------------------------
INSERT INTO `base_dict` VALUES ('1', '001', '客户行业', '教育培训 ', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('10', '003', '公司性质', '民企', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('12', '004', '年营业额', '1-10万', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('13', '004', '年营业额', '10-20万', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('14', '004', '年营业额', '20-50万', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('15', '004', '年营业额', '50-100万', NULL, '4', '1', NULL);
INSERT INTO `base_dict` VALUES ('16', '004', '年营业额', '100-500万', NULL, '5', '1', NULL);
INSERT INTO `base_dict` VALUES ('17', '004', '年营业额', '500-1000万', NULL, '6', '1', NULL);
INSERT INTO `base_dict` VALUES ('18', '005', '客户状态', '基础客户', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('19', '005', '客户状态', '潜在客户', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('2', '001', '客户行业', '电子商务', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('20', '005', '客户状态', '成功客户', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('21', '005', '客户状态', '无效客户', NULL, '4', '1', NULL);
INSERT INTO `base_dict` VALUES ('22', '006', '客户级别', '普通客户', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('23', '006', '客户级别', 'VIP客户', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('24', '007', '商机状态', '意向客户', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('25', '007', '商机状态', '初步沟通', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('26', '007', '商机状态', '深度沟通', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('27', '007', '商机状态', '签订合同', NULL, '4', '1', NULL);
INSERT INTO `base_dict` VALUES ('3', '001', '客户行业', '对外贸易', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('30', '008', '商机类型', '新业务', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('31', '008', '商机类型', '现有业务', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('32', '009', '商机来源', '电话营销', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('33', '009', '商机来源', '网络营销', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('34', '009', '商机来源', '推广活动', NULL, '3', '1', NULL);
INSERT INTO `base_dict` VALUES ('4', '001', '客户行业', '酒店旅游', NULL, '4', '1', NULL);
INSERT INTO `base_dict` VALUES ('5', '001', '客户行业', '房地产', NULL, '5', '1', NULL);
INSERT INTO `base_dict` VALUES ('6', '002', '客户信息来源', '电话营销', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('7', '002', '客户信息来源', '网络营销', NULL, '2', '1', NULL);
INSERT INTO `base_dict` VALUES ('8', '003', '公司性质', '合资', NULL, '1', '1', NULL);
INSERT INTO `base_dict` VALUES ('9', '003', '公司性质', '国企', NULL, '2', '1', NULL);

```
创建customer表单
``` bash
DROP TABLE IF EXISTS `customer`;
CREATE TABLE `customer` (
  `cust_id` BIGINT(32) NOT NULL AUTO_INCREMENT COMMENT '客户编号(主键)',
  `cust_name` VARCHAR(32) NOT NULL COMMENT '客户名称(公司名称)',
  `cust_user_id` BIGINT(32) DEFAULT NULL COMMENT '负责人id',
  `cust_create_id` BIGINT(32) DEFAULT NULL COMMENT '创建人id',
  `cust_source` VARCHAR(32) DEFAULT NULL COMMENT '客户信息来源',
  `cust_industry` VARCHAR(32) DEFAULT NULL COMMENT '客户所属行业',
  `cust_level` VARCHAR(32) DEFAULT NULL COMMENT '客户级别',
  `cust_linkman` VARCHAR(64) DEFAULT NULL COMMENT '联系人',
  `cust_phone` VARCHAR(64) DEFAULT NULL COMMENT '固定电话',
  `cust_mobile` VARCHAR(16) DEFAULT NULL COMMENT '移动电话',
  `cust_zipcode` VARCHAR(10) DEFAULT NULL,
  `cust_address` VARCHAR(100) DEFAULT NULL,
  `cust_createtime` DATETIME DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`cust_id`),
  KEY `FK_cst_customer_source` (`cust_source`),
  KEY `FK_cst_customer_industry` (`cust_industry`),
  KEY `FK_cst_customer_level` (`cust_level`),
  KEY `FK_cst_customer_user_id` (`cust_user_id`),
  KEY `FK_cst_customer_create_id` (`cust_create_id`)
) ENGINE=INNODB AUTO_INCREMENT=162 DEFAULT CHARSET=utf8;
```

给customer表单添加数据

``` bash
INSERT INTO `customer` VALUES ('14', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:01');
        INSERT INTO `customer` VALUES ('15', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:01');
        INSERT INTO `customer` VALUES ('16', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:01');
        INSERT INTO `customer` VALUES ('17', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:02');
        INSERT INTO `customer` VALUES ('22', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:03');
        INSERT INTO `customer` VALUES ('24', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:03');
        INSERT INTO `customer` VALUES ('25', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:03');
        INSERT INTO `customer` VALUES ('26', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:03');
        INSERT INTO `customer` VALUES ('28', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:04');
        INSERT INTO `customer` VALUES ('29', '令狐冲', NULL, NULL, '7', '1', '23', '任盈盈', '0108888886', '13888888886', '6123456', '电子科技大学6', '2016-04-08 16:32:04');
        INSERT INTO `customer` VALUES ('30', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:04');
        INSERT INTO `customer` VALUES ('31', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:04');
        INSERT INTO `customer` VALUES ('33', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:04');
        INSERT INTO `customer` VALUES ('34', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:05');
        INSERT INTO `customer` VALUES ('35', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:05');
        INSERT INTO `customer` VALUES ('36', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:05');
        INSERT INTO `customer` VALUES ('37', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:05');
        INSERT INTO `customer` VALUES ('38', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:05');
        INSERT INTO `customer` VALUES ('39', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:06');
        INSERT INTO `customer` VALUES ('40', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:06');
        INSERT INTO `customer` VALUES ('41', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:06');
        INSERT INTO `customer` VALUES ('42', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:06');
        INSERT INTO `customer` VALUES ('43', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:06');
        INSERT INTO `customer` VALUES ('44', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('45', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('46', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('47', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('48', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('49', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:07');
        INSERT INTO `customer` VALUES ('50', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('51', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('52', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('53', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('54', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('55', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:08');
        INSERT INTO `customer` VALUES ('56', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:09');
        INSERT INTO `customer` VALUES ('57', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:09');
        INSERT INTO `customer` VALUES ('58', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:09');
        INSERT INTO `customer` VALUES ('59', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:29');
        INSERT INTO `customer` VALUES ('60', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:29');
        INSERT INTO `customer` VALUES ('61', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:29');
        INSERT INTO `customer` VALUES ('62', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:29');
        INSERT INTO `customer` VALUES ('63', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('64', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('65', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('66', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('67', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('68', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:30');
        INSERT INTO `customer` VALUES ('69', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:31');
        INSERT INTO `customer` VALUES ('70', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:31');
        INSERT INTO `customer` VALUES ('71', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:31');
        INSERT INTO `customer` VALUES ('72', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:31');
        INSERT INTO `customer` VALUES ('73', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:31');
        INSERT INTO `customer` VALUES ('74', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('75', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('76', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('77', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('78', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('79', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:32');
        INSERT INTO `customer` VALUES ('80', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('81', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('82', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('83', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('84', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('85', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:33');
        INSERT INTO `customer` VALUES ('86', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('87', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('88', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('89', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('90', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('91', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:34');
        INSERT INTO `customer` VALUES ('92', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:35');
        INSERT INTO `customer` VALUES ('93', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:35');
        INSERT INTO `customer` VALUES ('94', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:35');
        INSERT INTO `customer` VALUES ('95', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:35');
        INSERT INTO `customer` VALUES ('96', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:35');
        INSERT INTO `customer` VALUES ('97', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('98', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('99', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('100', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('101', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('102', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:36');
        INSERT INTO `customer` VALUES ('103', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:37');
        INSERT INTO `customer` VALUES ('104', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:37');
        INSERT INTO `customer` VALUES ('105', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:37');
        INSERT INTO `customer` VALUES ('106', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:37');
        INSERT INTO `customer` VALUES ('107', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:37');
        INSERT INTO `customer` VALUES ('108', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('109', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('110', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('111', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('112', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('113', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:38');
        INSERT INTO `customer` VALUES ('114', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:39');
        INSERT INTO `customer` VALUES ('115', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:39');
        INSERT INTO `customer` VALUES ('116', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:39');
        INSERT INTO `customer` VALUES ('117', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:39');
        INSERT INTO `customer` VALUES ('118', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:39');
        INSERT INTO `customer` VALUES ('119', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('120', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('121', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('122', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('123', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('124', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:40');
        INSERT INTO `customer` VALUES ('125', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:41');
        INSERT INTO `customer` VALUES ('126', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:41');
        INSERT INTO `customer` VALUES ('127', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:41');
        INSERT INTO `customer` VALUES ('128', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:41');
        INSERT INTO `customer` VALUES ('129', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:41');
        INSERT INTO `customer` VALUES ('130', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('131', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('132', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('133', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('134', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('135', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:42');
        INSERT INTO `customer` VALUES ('136', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:43');
        INSERT INTO `customer` VALUES ('137', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:43');
        INSERT INTO `customer` VALUES ('138', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:43');
        INSERT INTO `customer` VALUES ('139', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:43');
        INSERT INTO `customer` VALUES ('140', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:43');
        INSERT INTO `customer` VALUES ('141', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('142', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('143', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('144', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('145', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('146', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:44');
        INSERT INTO `customer` VALUES ('147', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:45');
        INSERT INTO `customer` VALUES ('148', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:45');
        INSERT INTO `customer` VALUES ('149', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:45');
        INSERT INTO `customer` VALUES ('150', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:45');
        INSERT INTO `customer` VALUES ('151', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:45');
        INSERT INTO `customer` VALUES ('152', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('153', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('154', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('155', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('156', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('157', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:46');
        INSERT INTO `customer` VALUES ('158', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('159', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('160', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('161', '周敏鹤', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('162', '??', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('173', '你好', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('164', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('165', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('166', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('167', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('168', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('169', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('170', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('171', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('172', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('173', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('174', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('175', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('176', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('177', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('178', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
        INSERT INTO `customer` VALUES ('179', 'aa', NULL, NULL, '6', '2', '22', 'ZAP', '0108888887', '13888888888', '123456', '电子科技大学', '2016-04-08 16:32:47');
```

### 3.2 创建工程
笔者使用的开发工具是InteliJ IDEA，首先，新建一个工程，如果你想在已有工程下搭建的话，就新建一个module，如图
![](https://i.imgur.com/iDJjBYQ.png)
新建好了之后接下来就是对ProjectStructure进行配置，详细的内容可以参考笔者之前的文章[Setup your Tomcat server on IDEA](https://minhez.github.io/2018/10/26/Setup-your-Tomcat-server-on-IDEA/),接下来导入相关的jar包，如下图所示

![](https://i.imgur.com/ilt3B8J.png)

接下来创建工程的目录结构，其中config文件夹需要标注为源文件目录，如下图
![](https://i.imgur.com/cT7JvIP.png)

然后在config文件夹中添加配置文件，其中db.properties是数据库连接的配置文件，log4j.properties是Apache组织给出的开源日志配置文件，这个能让你更好地去观察程序运行的日志。

在config目录下新建MyBatis的全局配置文件SqlMapConfig.xml
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="false"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="5"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>


</configuration>
```
由下往上，接下来配置Spring相关的文件，新建ApplicationContext-dao.xml，这里面需要配置数据库连接文件，数据库连接池，管理MyBatis的会话工厂sqlSessionFactory，加载MyBatis全局配置文件，最后一个是配置Mapper扫描器，作用是建立起dao目录下的接口与xml配置文件之间的映射关系。
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!--配置数据库连接文件db.properties-->
    <context:property-placeholder location="classpath:db.properties"/>
    <!--配置数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}?useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxActive" value="10"/>
        <property name="maxIdle" value="5"/>
    </bean>

    <!--会话工厂-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--引入数据源-->
        <property name="dataSource" ref="dataSource"/>
        <!--加载MyBatis全局配置文件-->
        <property name="configLocation" value="classpath:SqlMapConfig.xml"/>
    </bean>

    <!--配置Mapper扫描器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="minhe.dao"/>
    </bean>
</beans>
```
新建ApplicationContext-service.xml，配置注解扫描，因为在service层会使用到注解注入的方式
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <context:component-scan base-package="minhe.service"/>

</beans>
```
新建ApplicationContext-trans.xml，配置事务管理器，并定义通知以及配置切面
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--引用数据源-->
        <property name="dataSource" ref="dataSource"/>

    </bean>
    <!--通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--传播行为-->
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置切面-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* minhe.service.*.*(..))"/>
    </aop:config>

</beans>
```
新建ApplicationContext-SpringMVC.xml，这里分别引入了字典源文件，controller层的注解扫描，配置注解驱动，视图解析器，自定义日期转换器。
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
       http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">
    <!--引入字典资源文件-->
    <context:property-placeholder location="classpath:source.properties"/>

    <!--配置controller注解扫描-->
    <context:component-scan base-package="minhe.controller"/>

    <!--配置注解驱动-->
    <mvc:annotation-driven conversion-service="conversionService"/>

    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--配置自定义的转换器-->
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="minhe.controller.converter.CustomGlobalStrToDateConverter"/>
            </set>
        </property>
    </bean>
</beans>
```
最后配置web.xml文件
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--解决post请求乱码-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--spring监听-->
    <display-name>springmvc-web</display-name>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>

    <!--加载spring容器-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:ApplicationContext-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--SpringMvc的前端控制器-->
    <servlet>
        <servlet-name>SpringMVC.xml</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC.xml</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>


</web-app>
```

以上所有文件配置完成如下图
![](https://i.imgur.com/xifVrK2.png)

在src目录下的工程结构中，新建以下文件
![](https://i.imgur.com/2Lv28Qe.png)

添加静态网页文件至web文件夹下，以及在WEB-INF下添加customer.jsp文件
![](https://i.imgur.com/BFMHGqS.png)

至此，整个工程的配置文件以及目录文件结构全部搭建完成。[本文工程请看]（https://github.com/MinheZ/SSM）


	本文作者：MinheZ
	版权声明：转载请注明出处！

	





