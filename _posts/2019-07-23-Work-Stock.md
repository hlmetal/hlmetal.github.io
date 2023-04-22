---
layout: post
title:  "项目梳理-库存服务"
date:   2019-07-23 15:00:30 +0200
categories: work
---

STOCK项目梳理, 主要针对现有功能、库表等进行梳理

## 项目介绍
该项目为库存服务(库存即老师的时间段), 提供了库存的创建、占用、移除、取消、查询等功能, 以及对不同产线库存使用的支持
## 项目库表
### stock库
### 主要表结构
1. 老师时间库存
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>stock</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>库存id</td>
		</tr>
        <tr>
			<td>stock_status</td>
			<td>tinyint</td>
			<td>库存状态(正常、移除、过期)</td>
		</tr>
		<tr>
			<td>provider_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>shcedule_time</td>
			<td>datetime</td>
			<td>课程开始时间</td>
		</tr>
        <tr>
			<td>shcedule_time_end</td>
			<td>datetime</td>
			<td>课程结束时间</td>
		</tr>
        <tr>
			<td>duration</td>
			<td>datetime</td>
			<td>持续时间</td>
		</tr>
        <tr>
			<td>total_quantity</td>
			<td>int</td>
			<td>总库存</td>
		</tr>
        <tr>
			<td>left_quantity</td>
			<td>int</td>
			<td>剩余库存</td>
		</tr>
        <tr>
			<td>last_book_time</td>
			<td>datetime</td>
			<td>最后可约时间</td>
		</tr>
	</tbody>
</table>

2. 订单库存关系
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>order_stock_relation</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>订单id</td>
		</tr>
        <tr>
			<td>order_type</td>
			<td>tinyint</td>
			<td>订单类型(来自那个产线)</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>tinyint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>stock_num</td>
			<td>smallint</td>
			<td>库存扣减数量</td>
		</tr>
		<tr>
			<td>order_status</td>
			<td>tinyint</td>
			<td>订单状态(占用、取消)</td>
		</tr>
	</tbody>
</table>

3. 库存定向表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>stock_target</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>target_code</td>
			<td>tinyint</td>
			<td>目标码</td>
		</tr>
		<tr>
			<td>target_value</td>
			<td>bigint</td>
			<td>目标值</td>
		</tr>
        <tr>
			<td>restore_action</td>
			<td>tinyint</td>
			<td>返还库存状态(关闭、转化常规、不变)</td>
		</tr>
        <tr>
			<td>expire_time</td>
			<td>datetime</td>
			<td>过期时间</td>
		</tr>
        <tr>
			<td>target_status</td>
			<td>tinyint</td>
			<td>定向状态(未使用、已使用、已过期)</td>
		</tr>
	</tbody>
</table>

4. 订单库存占用表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>order_occupation</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>order_type</td>
			<td>tinyint</td>
			<td>订单类型</td>
		</tr>
		<tr>
			<td>order_id</td>
			<td>bigint</td>
			<td>订单id</td>
		</tr>
		<tr>
			<td>provider_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>order_status</td>
			<td>tinyint</td>
			<td>订单状态</td>
		</tr>
	</tbody>
</table>

5. 库存属性表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>stock_prop</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>provider_tyoe</td>
			<td>tinyint</td>
			<td>供应商类别(对应不同产线老师)</td>
		</tr>
		<tr>
			<td>provider_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>allowed_reuse_occasion</td>
			<td>tinyint</td>
			<td>老师是否学生取消课后库存再利用场景</td>
		</tr>
	</tbody>
</table>

6. 库存快照表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>stock_snapshot</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>pre_snapshot_id</td>
			<td>bigint</td>
			<td>本次快照由哪次snapshot更新而来</td>
		</tr>
        <tr>
			<td>provider_type</td>
			<td>tinyint</td>
			<td>供应商类别</td>
		</tr>
		<tr>
			<td>provider_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>reuse_occasion</td>
			<td>tinyint</td>
			<td>老师是否学生取消课后库存再利用场景</td>
		</tr>
	</tbody>
</table>

7. 库存最新快照
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>stock_current_snapshot</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>snapshot_id</td>
			<td>bigint</td>
			<td>最新snapshotid</td>
		</tr>
	</tbody>
</table>

8. 事务回滚
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>rollback_action</caption>
	<thead>
		<tr>
			<th>主要字段</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>Bigint</td>
			<td>id</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>库存id</td>
		</tr>
		<tr>
			<td>action_type</td>
			<td>tinyint</td>
			<td>事件类型(占用时间、释放时间)</td>
		</tr>
        <tr>
			<td>action_status</td>
			<td>tinyint</td>
			<td>时间状态(准备、正常完成、mq补完、定时补完)</td>
		</tr>
		<tr>
			<td>action_body</td>
			<td>varchar</td>
			<td>事件相关信息</td>
		</tr>
        <tr>
			<td>reuse_occasion</td>
			<td>tinyint</td>
			<td>老师是否学生取消课后库存再利用场景</td>
		</tr>
	</tbody>
</table>

</br>

### 核心功能
1. 创建库存
* controller: 参数校验-> Build对象->调service->记录操作日志->返回信息
* service: 校验非冲突-> 数据组装-> 生成调用记录用于接口回滚-> try(占用时间(时间服务)->生成库存) catch(mq库存回补)-> 库存成功mq->返回信息
2. 扣减库存
* controller: 参数校验-> 调service->记录操作日志->返回信息
* service: 校验库存是否存在 -> 校验订单类型与库存类型是否一致 -> 校验是否定向 -> 校验库存数量 -> 校验订单状态 -> 实际扣减 -> 创建订单库存关系 -> 维护定向状态 -> 发送扣减mq-> 返回信息
3. 返还库存
* controller: 参数校验-> 调service->记录操作日志->返回信息
* service: 校验订单状态-> 校验是否24h之内返还-> 更新库存数量-> 更新订单状态 -> 发送释放库存mq -> 返回信息
4. 移除库存
* controller: 参数校验 -> 调service -> 记录操作日志 -> 返回信息
* service: 校验库存状态 -> 更新库存状态 -> 定向状态维护 -> 释放时间(时间服务调用失败则mq库存回补) -> 发送移除mq -> 返回信息
5. 库存过期
* 定时任务定时扫表 -> 检查库存状态正常,但已过时间的库存 -> 执行库存过期(更新状态)
6. 相应查询

### 相关技术
1. Spring Boot
2. Spring MVC
2. Maven
3. Mysql(Mybatis)
4. Redis(Jedis)
5. RestTemplate
6. cat
7. sentry-logback
8. 其他集成组件, 如配置中心, 限流组件, 连接池, 定时任务

