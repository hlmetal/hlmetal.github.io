---
layout: post
title:  "项目梳理-库存营销服务"
date:   2020-02-05 15:00:30 +0200
categories: work
---

stock-promotion项目简单梳理

## 项目介绍
库存营销服务主要用于提高老师的曝光度, 通过各种活动来增加老师放课和曝光, 提高学生约课量
## 项目库表
### stock_promotion库
### 主要表结构
1. 库存营销表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>promotion</caption>
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
			<td>营销活动id</td>
		</tr>
        <tr>
			<td>source_type</td>
			<td>tinyint</td>
			<td>营销来源类型(能量石活动、约课券活动)</td>
		</tr>
		<tr>
			<td>source_id</td>
			<td>bigint</td>
			<td>营销来源识别id</td>
		</tr>
		<tr>
			<td>start_time</td>
			<td>datetime</td>
			<td>营销开始时间</td>
		</tr>
        <tr>
			<td>end_time</td>
			<td>datetime</td>
			<td>营销结束时间</td>
		</tr>
        <tr>
			<td>title</td>
			<td>varchar</td>
			<td>营销展示标题</td>
		</tr>
        <tr>
			<td>parent_desc</td>
			<td>varchar</td>
			<td>家长端展示文案</td>
		</tr>
        <tr>
			<td>cost_department_code</td>
			<td>varchar</td>
			<td>成本部门代码</td>
		</tr>
        <tr>
			<td>parent_channel_type</td>
			<td>tinyint</td>
			<td>家长入口渠道类型(不限制或或app的banner页)</td>
		</tr>
        <tr>
			<td>parent_channel_id</td>
			<td>bigint</td>
			<td>家长入口渠道id</td>
		</tr>
        <tr>
			<td>promotion_status</td>
			<td>tinyint</td>
			<td>营销活动状态(草稿、废弃、上架、下架、过期)</td>
		</tr>
        <tr>
			<td>last_book_time</td>
			<td>datetime</td>
			<td>最后可约时间</td>
		</tr>
        <tr>
			<td>enabled_props_flag</td>
			<td>bigint</td>
			<td>启用属性标识位,由N个二进制位表征的数字,对应的二进制位如果为0,说明该属性无限制,对应的二进制位如果为1,说明该属性有限制</td>
		</tr>
        <tr>
			<td>props_declare_flag</td>
			<td>bigint</td>
			<td>属性声明标识位,由N个二进制位表征的数字,对应的二进制位如果为0,说明该属性以包含方式定义,对应的二进制位如果位1,说明该属性以排除方式定义</td>
		</tr>
	</tbody>
</table>

* 假设活动属性包括枚举值1,2,3,4,5,6,7,8,9
* 使用位运算来记录enabled_props_flag是否开启该属性
* 初始化Long = enabled_props_flag = 0L
* 假设新建活动中属性包括int value = 1,2,7,9
* 执行位运算将开启的属性转为二进制表示: enabled_props_flag = enabled_props_flag | (1 << (value - 1))
* 解析: isEnabled = (flag >> (value - 1) & 1) == 1L; (即上述计算反向操作)

2. 库存属性表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>

<table class="table table-bordered table-striped">
	<caption>stock_property</caption>
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
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销活动id</td>
		</tr>
		<tr>
			<td>property_type</td>
			<td>tinyint</td>
			<td>属性类型</td>
		</tr>
		<tr>
			<td>property_value</td>
			<td>bigint</td>
			<td>属性值</td>
		</tr>
        <tr>
			<td>property_status</td>
			<td>tinyint</td>
			<td>状态(正常、删除)</td>
		</tr>
	</tbody>
</table>

3. 营销奖励表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>

<table class="table table-bordered table-striped">
	<caption>promotion_reward</caption>
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
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销活动id</td>
		</tr>
		<tr>
			<td>reward_type</td>
			<td>tinyint</td>
			<td>奖励类型(能量石、抵扣课)</td>
		</tr>
		<tr>
			<td>reward_value</td>
			<td>bigint</td>
			<td>奖励值</td>
		</tr>
	</tbody>
</table>

4. 订单营销占用表
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
			<td>order_id</td>
			<td>bigint</td>
			<td>订单id</td>
		</tr>
		<tr>
			<td>order_type</td>
			<td>tinyint</td>
			<td>订单类型</td>
		</tr>
		<tr>
			<td>order_status</td>
			<td>tinyint</td>
			<td>订单状态(占用、取消)</td>
		</tr>
	</tbody>
</table>

5. 营销订单奖励生成记录表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>order_promotion_reward</caption>
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
			<td>order_id</td>
			<td>bigint</td>
			<td>订单id</td>
		</tr>
		<tr>
			<td>order_type</td>
			<td>tinyint</td>
			<td>订单类型</td>
		</tr>
		<tr>
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>student_id</td>
			<td>bigint</td>
			<td>学生id</td>
		</tr>
        <tr>
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销活动id</td>
		</tr>
        <tr>
			<td>reward_type</td>
			<td>tinyint</td>
			<td>营销奖励类型</td>
		</tr>
        <tr>
			<td>reward_value</td>
			<td>tinyint</td>
			<td>营销奖励值</td>
		</tr>
        <tr>
			<td>reward_status</td>
			<td>tinyint</td>
			<td>营销奖励状态(已关联、已失效、不发放、待发放、已发放)</td>
		</tr>
        <tr>
			<td>stock_id</td>
			<td>bigint</td>
			<td>营销对应的库存id</td>
		</tr>
        <tr>
			<td>source_type</td>
			<td>tinyint</td>
			<td>营销来源类型</td>
		</tr>
        <tr>
			<td>source_id</td>
			<td>bigint</td>
			<td>营销来源识别id </td>
		</tr>
	</tbody>
</table>

6. 活动名单基本信息表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>activity_user_group</caption>
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
			<td>source_id</td>
			<td>bigint</td>
			<td>来源id</td>
		</tr>
		<tr>
			<td>user_group_id</td>
			<td>bigint</td>
			<td>人群id(活动名单来源于那个人群)</td>
		</tr>
		<tr>
			<td>group_type</td>
			<td>tinyint</td>
			<td>名单类型(老师曝光名单、老师参与名单、学生参与名单)</td>
		</tr>
        <tr>
			<td>need_sync</td>
			<td>tinyint</td>
			<td>是否同步数据</td>
		</tr>
	</tbody>
</table>

7. 活动名单成员信息表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>activity_user_group_member</caption>
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
			<td>activiy_user_group_id</td>
			<td>bigint</td>
			<td>活动人群id</td>
		</tr>
		<tr>
			<td>user_id</td>
			<td>bigint</td>
			<td>用户id</td>
		</tr>
		<tr>
			<td>group_type</td>
			<td>tinyint</td>
			<td>名单类型(老师曝光名单、老师参与名单、学生参与名单)</td>
		</tr>
        <tr>
			<td>member_status</td>
			<td>tinyint</td>
			<td>成员状态(正常、无效)</td>
		</tr>
	</tbody>
</table>

8. 用户人群表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>user_group</caption>
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
			<td>group_source_type</td>
			<td>tinyint</td>
			<td>人群来源类型</td>
		</tr>
		<tr>
			<td>group_source_id</td>
			<td>bigint</td>
			<td>人群来源id</td>
		</tr>
		<tr>
			<td>user_type</td>
			<td>tinyint</td>
			<td>用户类型(学生、老师)</td>
		</tr>
        <tr>
			<td>group_status</td>
			<td>tinyint</td>
			<td>人群状态(草稿、有效、无效、删除)</td>
		</tr>
        <tr>
			<td>name</td>
			<td>varchar</td>
			<td>人群名称</td>
		</tr>
        <tr>
			<td>is_sync</td>
			<td>tinyint</td>
			<td>是否同步</td>
		</tr>
        <tr>
			<td>last_sync_time</td>
			<td>datetime</td>
			<td>上次同步时间</td>
		</tr>
        <tr>
			<td>del_flag</td>
			<td>tinyint</td>
			<td>是否删除</td>
		</tr>
	</tbody>
</table>

9. 用户人群成员表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>user_group_member</caption>
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
			<td>user_group_id</td>
			<td>bigint</td>
			<td>人群id</td>
		</tr>
		<tr>
			<td>user_type</td>
			<td>tinyint</td>
			<td>用户类别</td>
		</tr>
		<tr>
			<td>user_id</td>
			<td>bigint</td>
			<td>用户id</td>
		</tr>
        <tr>
			<td>member_status</td>
			<td>tinyint</td>
			<td>成员状态(正常、无效)</td>
		</tr>
	</tbody>
</table>

10. 用户人群标签表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>user_group_tag</caption>
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
			<td>user_group_id</td>
			<td>bigint</td>
			<td>人群id</td>
		</tr>
		<tr>
			<td>group_source_id</td>
			<td>bigint</td>
			<td>人群来源id</td>
		</tr>
		<tr>
			<td>user_tag_type</td>
			<td>tinyint</td>
			<td>用户标签类型</td>
		</tr>
        <tr>
			<td>user_tag_meta_id</td>
			<td>bigint</td>
			<td>标签元数据id</td>
		</tr>
	</tbody>
</table>

11. 用户标签元数据表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>user_tag_meta</caption>
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
			<td>meta_id</td>
			<td>bigint</td>
			<td>标签唯一标识id</td>
		</tr>
		<tr>
			<td>user_tag_type</td>
			<td>tinyint</td>
			<td>用户标签类型</td>
		</tr>
		<tr>
			<td>user_tag_value1</td>
			<td>int</td>
			<td>标签值1</td>
		</tr>
        <tr>
			<td>user_tag_value2</td>
			<td>int</td>
			<td>标签值2</td>
		</tr>
        <tr>
			<td>user_tag_value_desc</td>
			<td>varchar</td>
			<td>标签对应文本描述</td>
		</tr>
	</tbody>
</table>

12. 老师标签信息表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>teacher_tag</caption>
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
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>user_tag_type</td>
			<td>tinyint</td>
			<td>用户标签类型</td>
		</tr>
		<tr>
			<td>user_tag_meta_id</td>
			<td>bigint</td>
			<td>标签元数据id</td>
		</tr>
        <tr>
			<td>last_edit_time</td>
			<td>datetime</td>
			<td>最后修改时间</td>
		</tr>
	</tbody>
</table>

13. 老师原生数据表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>raw_teacher</caption>
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
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>cursor</td>
			<td>varchar</td>
			<td>时间PT</td>
		</tr>
		<tr>
			<td>classify</td>
			<td>tinyint</td>
			<td>类型</td>
		</tr>
        <tr>
			<td>booked_rate_at_peak_time</td>
			<td>smallint</td>
			<td>PT时间订课率</td>
		</tr>
        <tr>
			<td>booked_rate_at_peak_peak_time</td>
			<td>smalltime</td>
			<td>PPT时间订课率</td>
		</tr>
        <tr>
			<td>slots_opened_in_future</td>
			<td>int</td>
			<td>未来30天放课量</td>
		</tr>
        <tr>
			<td>slots_opened_in_future_at_peak_time</td>
			<td>int</td>
			<td>未来30天PT时间放课量</td>
		</tr>
        <tr>
			<td>slots_opened_in_future_at_peak_peak_time</td>
			<td>int</td>
			<td>未来30天PPT时间放课量</td>
		</tr>
        <tr>
			<td>entry_days</td>
			<td>int</td>
			<td>入职天数</td>
		</tr>
        <tr>
			<td>exposure_uv</td>
			<td>int</td>
			<td>近60天广告位曝光uv数量</td>
		</tr>
	</tbody>
</table>

14. 分类老师
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>in_classify_teacher</caption>
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
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>classify</td>
			<td>tinyint</td>
			<td>类型</td>
		</tr>
        <tr>
			<td>last_edit_time</td>
			<td>最后修改时间</td>
			<td>PT时间订课率</td>
		</tr>
	</tbody>
</table>

15. 老师能量石活动表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>taecher_activity</caption>
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
			<td>title</td>
			<td>varchar</td>
			<td>活动标题</td>
		</tr>
		<tr>
			<td>inner_title</td>
			<td>varchar</td>
			<td>内部标题</td>
		</tr>
		<tr>
			<td>teacher_activiy_type</td>
			<td>tinyint</td>
			<td>活动类型(常规、新老师)</td>
		</tr>
        <tr>
			<td>exposure_position</td>
			<td>bigint</td>
			<td>曝光位置</td>
		</tr>
        <tr>
			<td>show_start_time</td>
			<td>datetime</td>
			<td>活动展示开始时间</td>
		</tr>
        <tr>
			<td>show_end_time</td>
			<td>datetime</td>
			<td>活动展示截止时间</td>
		</tr>
        <tr>
			<td>activiy_tag</td>
			<td>varchar</td>
			<td>活动标签</td>
		</tr>
        <tr>
			<td>tag_desc</td>
			<td>varchar</td>
			<td>html形式的标签说明</td>
		</tr>
        <tr>
			<td>landing_page_image</td>
			<td>varchar</td>
			<td>落地页头图</td>
		</tr>
        <tr>
			<td>landing_page_desc</td>
			<td>varchar</td>
			<td>落地页说明</td>
		</tr>
        <tr>
			<td>student_declare_way</td>
			<td>tinyint</td>
			<td>学生指定方式(全量、导入名单、人群指定)</td>
		</tr>
        <tr>
			<td>student_user_group_ids</td>
			<td>varchar</td>
			<td>学生人群id</td>
		</tr>
        <tr>
			<td>student_upload_batch_id</td>
			<td>bigint</td>
			<td>导入名单批次id</td>
		</tr>
        <tr>
			<td>book_start_time</td>
			<td>datetime</td>
			<td>活动约课开始时间</td>
		</tr>
        <tr>
			<td>book_end_time</td>
			<td>datetime</td>
			<td>活动约课截止时间</td>
		</tr>
        <tr>
			<td>schedule_start_time</td>
			<td>datetime</td>
			<td>活动上课开始时间</td>
		</tr>
        <tr>
			<td>schedule_end_time</td>
			<td>datetime</td>
			<td>活动上课截止时间</td>
		</tr>
        <tr>
			<td>enroll_start_time</td>
			<td>datetime</td>
			<td>报名开始时间</td>
		</tr>
        <tr>
			<td>enroll_end_time</td>
			<td>datetime</td>
			<td>报名结束时间</td>
		</tr>
        <tr>
			<td>times</td>
			<td>varchar</td>
			<td>限定时间段</td>
		</tr>
        <tr>
			<td>is_multi_session</td>
			<td>tinyint</td>
			<td>是否多个子活动</td>
		</tr>
        <tr>
			<td>activity_status</td>
			<td>tinyint</td>
			<td>活动状态(草稿、未上架、已上架、已结束)</td>
		</tr>
        <tr>
			<td>activity_time_status</td>
			<td>tinyint</td>
			<td>活动时间状态(未开始、报名中、待订课、订课中、订课结束、已结束)</td>
		</tr>
        <tr>
			<td>del_flag</td>
			<td>tinyint</td>
			<td>是否删除</td>
		</tr>
	</tbody>
</table>

16. 子活动信息表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>teacher_activity_session</caption>
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
			<td>activity_id</td>
			<td>bigint</td>
			<td>所属活动id</td>
		</tr>
		<tr>
			<td>teacher_declare_way</td>
			<td>tinyint</td>
			<td>老师定义方式(全量、导入名单、人群指定)</td>
		</tr>
		<tr>
			<td>teacher_user_group_ids</td>
			<td>varchar</td>
			<td>老师人群id</td>
		</tr>
        <tr>
			<td>expose_group_id</td>
			<td>bigint</td>
			<td>曝光人群id</td>
		</tr>
        <tr>
			<td>group_dynamic</td>
			<td>tinyint</td>
			<td>是否动态人群</td>
		</tr>
        <tr>
			<td>teacher_upload_batch_id</td>
			<td>bigint</td>
			<td>导入名单id</td>
		</tr>
        <tr>
			<td>teacher_session_limit</td>
			<td>tinyint</td>
			<td>活动期间扶植课程上上限</td>
		</tr>
        <tr>
			<td>session_schedule_start_time</td>
			<td>datetime</td>
			<td>子活动上课开始时间</td>
		</tr>
        <tr>
			<td>session_schedule_end_time</td>
			<td>datetime</td>
			<td>子活动上课截止时间</td>
		</tr>
        <tr>
			<td>teacher_page_image</td>
			<td>varchar</td>
			<td>教师端活动页头图</td>
		</tr>
        <tr>
			<td>teacher_page_desc</td>
			<td>varchar</td>
			<td>教师端活动页说明</td>
		</tr>
        <tr>
			<td>teacher_page_introduction_url</td>
			<td>varchar</td>
			<td>教师端说明页链接</td>
		</tr>
        <tr>
			<td>times</td>
			<td>varchar</td>
			<td>时间点限制</td>
		</tr>
        <tr>
			<td>reward_rule_type</td>
			<td>tinyint</td>
			<td>奖励额度规则(固定额度)</td>
		</tr>
        <tr>
			<td>reward_type</td>
			<td>tinyint</td>
			<td>奖励类型(能量石)</td>
		</tr>
        <tr>
			<td>reward_value</td>
			<td>bigint</td>
			<td>奖励值</td>
		</tr>
        <tr>
			<td>promotion_id</td>
			<td>bigint</td>
			<td>对应营销id</td>
		</tr>
        <tr>
			<td>settle_type</td>
			<td>tinyint</td>
			<td>结算类型(平台支付、老师支付、联合支付)</td>
		</tr>
        <tr>
			<td>settle_value_teacher</td>
			<td>bigint</td>
			<td>老师支付数量</td>
		</tr>
        <tr>
			<td>settle_value_vk</td>
			<td>bigint</td>
			<td>平台支付数量</td>
		</tr>
        <tr>
			<td>announcement</td>
			<td>varchar</td>
			<td>通知文案</td>
		</tr>
        <tr>
			<td>activity_status</td>
			<td>tinyint</td>
			<td>活动状态(草稿、未上架、已上架、已结束)</td>
		</tr>
        <tr>
			<td>activity_time_status</td>
			<td>tinyint</td>
			<td>活动时间状态(未开始、报名中、待订课、订课中、订课结束、已结束)</td>
		</tr>
        <tr>
			<td>del_flag</td>
			<td>tinyint</td>
			<td>是否删除</td>
		</tr>
	</tbody>
</table>

17. 营销额度表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>promotion_limit</caption>
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
			<td>promotion_id</td>
			<td>bigint</td>
			<td>所属营销id</td>
		</tr>
		<tr>
			<td>limit_type</td>
			<td>tinyint</td>
			<td>额度类型</td>
		</tr>
		<tr>
			<td>limit_value</td>
			<td>int</td>
			<td>额度数值</td>
		</tr>
	</tbody>
</table>

18. 营销满额老师表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>promotion_limited_teacher</caption>
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
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>limit_type</td>
			<td>tinyint</td>
			<td>额度类型</td>
		</tr>
		<tr>
			<td>limit_value</td>
			<td>int</td>
			<td>额度大小</td>
		</tr>
        <tr>
			<td>current_value</td>
			<td>int</td>
			<td>已消耗额度大小</td>
		</tr>
        <tr>
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销id</td>
		</tr>
        <tr>
			<td>limit_status</td>
			<td>tinyint</td>
			<td>额度状态(满额、未满额)</td>
		</tr>
	</tbody>
</table>

19. 奖励结算规则表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>promotion_reward_settlement</caption>
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
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销id</td>
		</tr>
		<tr>
			<td>settle_finish_type</td>
			<td>tinyint</td>
			<td>结算类型</td>
		</tr>
		<tr>
			<td>settle_source</td>
			<td>tinyint</td>
			<td>结算来源(平台、老师)</td>
		</tr>
        <tr>
			<td>settle_value</td>
			<td>bigint</td>
			<td>结算类型</td>
		</tr>
	</tbody>
</table>

20. 奖励结算记录表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>order_promotion_reward_settlement</caption>
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
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>student_id</td>
			<td>bigint</td>
			<td>学生id</td>
		</tr>
        <tr>
			<td>promotion_id</td>
			<td>bigint</td>
			<td>营销id</td>
		</tr>
        <tr>
			<td>settle_source</td>
			<td>tinyint</td>
			<td>结算来源</td>
		</tr>
        <tr>
			<td>settle_value</td>
			<td>bigint</td>
			<td>结算数额</td>
		</tr>
        <tr>
			<td>settle_finish_type</td>
			<td>varchar</td>
			<td>完课FT</td>
		</tr>
	</tbody>
</table>

21. 活动报名表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>teacher_activity_enrollment</caption>
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
			<td>activity_id</td>
			<td>bigint</td>
			<td>活动id</td>
		</tr>
		<tr>
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
		<tr>
			<td>session_id</td>
			<td>bigint</td>
			<td>活动群体规则id</td>
		</tr>
        <tr>
			<td>enroll_time</td>
			<td>datetime</td>
			<td>报名时间</td>
		</tr>
	</tbody>
</table>

22. 用户id上传记录表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>upload_record</caption>
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
			<td>batch_id</td>
			<td>bigint</td>
			<td>批次id</td>
		</tr>
		<tr>
			<td>user_type</td>
			<td>tinyint</td>
			<td>用户类型(学生、老师)</td>
		</tr>
		<tr>
			<td>user_id</td>
			<td>bigint</td>
			<td>用户di</td>
		</tr>
	</tbody>
</table>

23. 券活动规则表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>coupon_rule</caption>
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
			<td>coupon_no</td>
			<td>varchar</td>
			<td>券模板id</td>
		</tr>
		<tr>
			<td>coupon_page_title</td>
			<td>varchar</td>
			<td>券落地页标题</td>
		</tr>
		<tr>
			<td>coupon_page_image</td>
			<td>varchar</td>
			<td>券落地页头图</td>
		</tr>
        <tr>
			<td>coupon_page_desc</td>
			<td>varchar</td>
			<td>券落地页说明</td>
		</tr>
        <tr>
			<td>coupon_tag_text</td>
			<td>varchar</td>
			<td>券标签文案</td>
		</tr>
        <tr>
			<td>coupon_tag_label</td>
			<td>varchar</td>
			<td>券标签右侧说明文案</td>
		</tr>
        <tr>
			<td>coupon_tag_page_image</td>
			<td>varchar</td>
			<td>券标签落地页头图</td>
		</tr>
        <tr>
			<td>coupon_tag_page_desc</td>
			<td>varchar</td>
			<td>券标签落地页说明</td>
		</tr>
        <tr>
			<td>coupon_rule_status</td>
			<td>tinyint</td>
			<td>券模版状态(绑定、审核通过、上架、下架、过期)</td>
		</tr>
        <tr>
			<td>enabled_props_flag</td>
			<td>bigint</td>
			<td>启用属性标识位,由N个二进制位表征的数字,对应的二进制位如果为0,说明该属性无限制,对应的二进制位如果为1,说明该属性有限制</td>
		</tr>
        <tr>
			<td>props_declare_flag</td>
			<td>bigint</td>
			<td>属性声明标识位,由N个二进制位表征的数字,对应的二进制位如果为0,说明该属性以包含方式定义,对应的二进制位如果位1,说明该属性以排除方式定义</td>
		</tr>
        <tr>
			<td>coupon_outer_name</td>
			<td>varchar</td>
			<td>券对外名称</td>
		</tr>
        <tr>
			<td>coupon_type</td>
			<td>tinyint</td>
			<td>券类型(约课券)</td>
		</tr>
        <tr>
			<td>book_class_count</td>
			<td>tinyint</td>
			<td>课时数</td>
		</tr>
        <tr>
			<td>receive_begin_time</td>
			<td>datetime</td>
			<td>券可领取开始时间</td>
		</tr>
        <tr>
			<td>receive_end_time</td>
			<td>datetime</td>
			<td>券可领取结束时间</td>
		</tr>
        <tr>
			<td>start_time</td>
			<td>datetime</td>
			<td>券可使用开始时间</td>
		</tr>
        <tr>
			<td>end_time</td>
			<td>datetime</td>
			<td>券可使用结束时间</td>
		</tr>
        <tr>
			<td>join_text</td>
			<td>varchar</td>
			<td>券说明页按钮文案</td>
		</tr>
        <tr>
			<td>join_link</td>
			<td>varchar</td>
			<td>券说明页按钮链接</td>
		</tr>
	</tbody>
</table>

24. 券规则属性
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>coupon_rule_property</caption>
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
			<td>coupon_rule_id</td>
			<td>bigint</td>
			<td>券活动ID</td>
		</tr>
		<tr>
			<td>coupon_no</td>
			<td>varchar</td>
			<td>券模版ID</td>
		</tr>
		<tr>
			<td>property_type</td>
			<td>tinyint</td>
			<td>属性类型</td>
		</tr>
        <tr>
			<td>property_value</td>
			<td>bigint</td>
			<td>属性值</td>
		</tr>
        <tr>
			<td>parent_property_id</td>
			<td>bigint</td>
			<td>父属性id</td>
		</tr>
	</tbody>
</table>

25. 券活动奖励
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>coupon_rule_reward</caption>
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
			<td>coupon_rule_id</td>
			<td>bigint</td>
			<td>券活动ID</td>
		</tr>
		<tr>
			<td>coupon_no</td>
			<td>varchar</td>
			<td>券模版ID</td>
		</tr>
		<tr>
			<td>reward_type</td>
			<td>tinyint</td>
			<td>奖励类型</td>
		</tr>
        <tr>
			<td>reward_value</td>
			<td>bigint</td>
			<td>奖励值</td>
		</tr>
	</tbody>
</table>

26. 营销交易表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>coupon_rule_reward</caption>
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
			<td>订单ID</td>
		</tr>
		<tr>
			<td>teacher_id</td>
			<td>bigint</td>
			<td>老师id</td>
		</tr>
        <tr>
			<td>student_id</td>
			<td>bigint</td>
			<td>学生id</td>
		</tr>
		<tr>
			<td>deal_flag</td>
			<td>bigint</td>
			<td>冗余标识位</td>
		</tr>
		<tr>
			<td>deal_status</td>
			<td>tinyint</td>
			<td>交易状态(绑定、废弃、取消、完成)</td>
		</tr>
	</tbody>
</table>

27. 营销交易属性表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>deal_property</caption>
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
			<td>订单ID</td>
		</tr>
		<tr>
			<td>detail_property_type</td>
			<td>tinyint</td>
			<td>属性类型</td>
		</tr>
        <tr>
			<td>detail_property_value</td>
			<td>bigint</td>
			<td>属性值</td>
		</tr>
	</tbody>
</table>

28. 营销交易资源表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>deal_resource</caption>
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
			<td>订单ID</td>
		</tr>
		<tr>
			<td>coupon_nos</td>
			<td>varchar</td>
			<td>券模板id</td>
		</tr>
        <tr>
			<td>coupon_codes</td>
			<td>varchar</td>
			<td>券id</td>
		</tr>
		<tr>
			<td>resource_status</td>
			<td>tinyint</td>
			<td>资源状态(待占用券、已占用券、待返还券、已返还券)</td>
		</tr>
	</tbody>
</table>

29. 券活动规则设置表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>coupon_rule_setting</caption>
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
			<td>coupon_rule_id</td>
			<td>bigint</td>
			<td>券规则id</td>
		</tr>
		<tr>
			<td>coupon_no</td>
			<td>varchar</td>
			<td>券模版id</td>
		</tr>
		<tr>
			<td>setting_type</td>
			<td>tinyint</td>
			<td>设置类型</td>
		</tr>
        <tr>
			<td>setting_value</td>
			<td>bigint</td>
			<td>设置值</td>
		</tr>
		<tr>
			<td>parent_setting_id</td>
			<td>bigint</td>
			<td>父设置id</td>
		</tr>
	</tbody>
</table>

30. 广告位表
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>ad_slot</caption>
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
			<td>ad_title</td>
			<td>varchar</td>
			<td>广告位标题</td>
		</tr>
		<tr>
			<td>ad_type</td>
			<td>tinyint</td>
			<td>广告位类型(新老师广告位)</td>
		</tr>
		<tr>
			<td>ad_status</td>
			<td>tinyint</td>
			<td>广告位状态(草稿、上架、下架、结束)</td>
		</tr>
		<tr>
			<td>teacher_group_id</td>
			<td>bigint</td>
			<td>老师人群id</td>
		</tr>
        <tr>
			<td>launch_start_time</td>
			<td>datetime</td>
			<td>投放开始时间</td>
		</tr>
		<tr>
			<td>launch_end_time</td>
			<td>datetime</td>
			<td>投放结束时间</td>
		</tr>
		<tr>
			<td>exposure_amount</td>
			<td>bigint</td>
			<td>活动新增曝光量上限</td>
		</tr>
		<tr>
			<td>del_flag</td>
			<td>tinyint</td>
			<td>标志位(正常,已删除)</td>
		</tr>
	</tbody>
</table>

31. 其他表, 包括操作日志表、上传文件表、上传文件数据表等

</br>

## 核心功能
### 老师能量石活动
1. 创建/更新活动
* 创建/更新基本信息-> 创建/更新活动规则信息(学生人群选择、老师人群选择、时间段限定、额度设置、奖励设置、落地页数据)
2. 活动上架
* 上架前检查(id、状态、人群信息、活动时间等)-> 生成营销数据 -> 初始化活动相关属性 -> 记录上架事件 -> 活动节点事件加入redis延迟队列
3. 下架活动
* 下架前检查-> 更新活动状态 -> 记录下架事件 -> 关闭数据同步
4. 删除活动
* 删除前检查 -> 更新活动状态 -> 记录删除事件
5. 活动过期
* 由定时任务执行
6. 复制活动

### 约课券活动
相较于前者规则较为简单, 券模板由第三方服务提供, 同样包括创建、上架、下架、删除、过期等功能

### 核心逻辑
1. 老师能量石活动: 设置活动规则(哪些老师的课包含能量石标签, 哪些学生/老师能参与活动, 活动额度等) -> 学生参与并完课 -> 奖励结算 -> 能量石发放
  <img src= "/assets/files/stock-promotion.jpg" alt="加载错误" title="老师营销活动基本逻辑"/>
2. 约课券活动: 券服务负责提供券模板 -> 拿到券后对给券加上活动规则(哪些老师的课能够使用、哪些课程id能够使用等) -> 学生用券参与并完课 -> 扣减约课券
* 创建券活动流程
  <img src= "/assets/files/创建券活动流程.jpg" alt="加载错误" title="创建券活动流程"/>
* 课时券发放环节服务交互流程
  <img src= "/assets/files/课时券发放环节服务交互.jpg" alt="加载错误" title="课时券发放环节服务交互流程"/>
* 课时券使用环节服务交互流程
  <img src= "/assets/files/课时券使用环节服务交互.jpg" alt="加载错误" title="课时券使用环节服务交互流程"/>