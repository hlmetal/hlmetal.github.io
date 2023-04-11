---
layout: post
title: Echarts的基本使用
category: web echarts
tags: [web]
---

Echarts的基本知识与简单实践

## [Echarts快速上手](https://echarts.apache.org/handbook/zh/basics/import)
### 安装: `npm install echarts`
### 引入echarts: `import echarts from 'echarts';`
### 基本概念
* 图表容器及大小：通常需要在HTML中先定义一个`div`节点，并且通过CSS使得该节点具有宽度和高度。初始化的时候，传入该节点，图表的大小默认即为该节点的大小
* 坐标轴: x轴和 y轴都由轴线、刻度、刻度标签、轴标题四个部分组成。部分图表中还会有网格线来帮助查看和计算数据
* 系列: 即series, 图标中数据管理的组件
* 图例: 图例是图表中对内容区元素的注释、用不同形状、颜色、文字等来标示不同数据列，通过点击对应数据列的标记，可以显示或隐藏该数据列。

### 基本API
* `this.chartLine = echarts.init(dom);` 初始化创建图表实例, 返回echartsInstance  , dom是实例容器，一般是一个具有高宽的DIV元素
* `this.chartLine.setOption({})`设置图表实例的配置项以及数据，万能接口，所有参数和数据的修改都可以通过`setOption`完成，ECharts 会合并新的参数和数据，然后刷新图表。如果开启动画的话，ECharts 找到两组数据之间的差异然后通过合适的动画去表现数据的变化。

### setOption中的配置项
* `title{}` 标题组件,包括text、subtext、textStyle、subtextStyle(color、fontStyle、fontWeight、fontFamily、fontSize)等属性
* `legend{}` 图例组件, 包括selectedMode(图例选择的模式)、data(图例的数据数组)等属性
* `tooltip{}` 提示框组件, 包括trigger(触发类型)等一系列属性
* `grid{}` 直角坐标系内绘图网格, 包括top、left、right、bottom(gird组件与容器上下左右距离)、containLabel(grid 区域是否包含坐标轴的刻度标签)
* `toolbox{}` 工具栏, 包括feature(工具匹配值项, 如内置工具saveAsImage)等属性
* `xAxis{}` 直角坐标系grid中x轴, 包括name、type、boundaryGap、axisLabel、data等属性 
* `yAxis{}` 直角坐标系grid中y轴
* `series[{}]` 图标内容, 包括设置图表类型(折线、柱状等)、设置图表数据、设计标记类型

## 折线图实践(Vue项目)
```js
<template>
    <section class="chart-container">
        <div align="center">
          选择时间范围：
          <el-date-picker
            @change="getCustomizeItems"
            v-model="dateValue"
            type="daterange"
            align="center"
            format="yyyy 年 MM 月 dd 日"
            value-format="yyyy-MM-dd 00:00:00"
            unlink-panels
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :picker-options="pickerOptions">
          </el-date-picker>
          <h7></h7>
        </div>
        
        <el-row>
            <el-col :span="20">
                <div id="chartLine" style="width:120%; height:400px;"></div>
            </el-col>
        </el-row>
    </section>
</template>
<script>
import echarts from 'echarts'
import axios from 'axios'
	import { dateFormat } from '../../js/dateutil';
	import queryApi from '../../api/queryApi';
    export default {
        data() {
            return {
                pickerOptions: {
                    shortcuts: [{
                        text: '最近一周',
                        onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                        picker.$emit('pick', [start, end]);
                        }
                    }, {
                        text: '最近一个月',
                        onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                        picker.$emit('pick', [start, end]);
                        }
                    }, {
                        text: '最近三个月',
                        onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                        picker.$emit('pick', [start, end]);
                        }
                    }]
                },
                dateValue: '',
                chartLine: null,
                statTypes: '1,2,3,4,5',
                statTimes: [],
                createCount: [],
                targetCreateCount: [],
                studentTargetCreateCount: [],
                productTargetCreateCount: [],
                removeCount: []
            }
        },

        methods: {
            getItem: function(items) {
                items.forEach( item => {
                        if (item.statType == 1) {
                            this.statTimes.push(item.statTime);
                            this.createCount.push([item.statTime, item.statValue]);
                        }
                        if (item.statType == 2) {
                            this.targetCreateCount.push([item.statTime, item.statValue]);
                        }
                        if (item.statType == 3) {
                            this.studentTargetCreateCount.push([item.statTime, item.statValue]);
                        }
                        if (item.statType == 4) {
                            this.productTargetCreateCount.push([item.statTime, item.statValue]);
                        }
                        if (item.statType == 5) {
                            this.removeCount.push([item.statTime, item.statValue]);
                        }
                    });
            },
            clearItem: function() {
                this.statTimes = [];
                this.createCount = [];
                this.targetCreateCount = [];
                this.studentTargetCreateCount = [];
                this.productTargetCreateCount = [];
                this.removeCount = [];
            },
            getItems: function () {
				console.log('getItems called: ', this)
                let para = {"statTypes": this.statTypes};
                console.log("statTyps:" + this.statTypes);
				let testObj = queryApi.getStockStatCount(para).then(res => {
					console.log('%c CreateAndCancelStockCount.vue getItems:', 'color:#0f0;', res);
                    let items = res.data.data;
                    console.log('%c CreateAndCancelStockCount.vue this.items:', 'color:#0f0;', items);
                    if (items != null) {
                       this.getItem(items);
                    } else {
                        console.log("items is null, url is error")
                    }
                    this.drawLineChart();
                    this.clearItem();
                });
            },
            getCustomizeItems: function(dateValue) {
                console.log("date:" + dateValue);
                if (dateValue == null) { this.getItems(); return;}
                let fromTime = dateValue.toString().substr(0, 19);
                let toTime = dateValue.toString().substr(20, 38);
                console.log("fromTime:" + fromTime + Date.parse(fromTime) +'\n'+ "toTime:" + toTime + Date.parse(toTime));
                let para= {"statTypes": this.statTypes, "fromTime": Date.parse(fromTime), "toTime": Date.parse(toTime)};
                let testObj = queryApi.getStockStatCount(para).then(res => {
                    console.log('%c CreateAndCancelStockCountInPeriod.vue getItems:', 'color:#0f0;', res);
                    let items = res.data.data;
                    console.log('%c CreateAndCancelStockCountInPeriod.vue this.items:', 'color:#0f0;', items);
                    if (items != null) {
                       this.getItem(items);
                    } else {
                        console.log("items is null, no data")
                    }
                    this.drawLineChart();
                    this.clearItem();
                });
            },
            drawLineChart() {
                this.chartLine = echarts.init(document.getElementById('chartLine'));
                this.chartLine.setOption({
                    backgroundColor: '#fbfbfb',
                    title: {
                        text: '放课统计',
                        subtext: '(点击图例隐藏或显示折线,以便观察对比)',
                        subtextStyle:{
                            color:'darkslategrey',
                            fontStyle:'normal',
                            fontWeight:"lighter",
                            fontFamily:"san-serif",
                            fontSize:12
                        }
                    },
                    tooltip: {
                        trigger: 'axis'
                    },
                    legend: {
                        selectedMode: 'multiple',
                        data: ['放课总量', '定向放课量', '学生定向放课量', '产品定向放课量', '取消课量']
                    },
                    grid: {
                        top: '20%',
                        left: '3%',
                        right: '5%',
                        bottom: '3%',
                        containLabel: true
                    },
                    toolbox: {
                        feature: {
                            saveAsImage: {}
                        }
                    },
                    xAxis: {
                        name: '时间',
                        type: 'category',
                        boundaryGap: false,
                        axisLabel: {
                            rotate: 40,
                            interval: 0
                        },
                        data: this.statTimes
                    },
                    yAxis: {
                        name: '数量',
                        type: 'value'
                    },
                    series: [
                        {
                            name: '放课总量',
                            type: 'line',
                            stack: '总量1',
                            symbol: 'none',
                            smooth: 0.3,
                            itemStyle: {
                                normal: {
                                    label: {
                                        show: true
                                    }
                                }
                            },
                            data: this.createCount
                        },
                        {
                            name: '定向放课量',
                            type: 'line',
                            stack: '总量2',
                            symbol: 'none',
                            smooth: 0.3,
                            itemStyle: {
                                normal: {
                                    label: {
                                        show: true
                                    }
                                }
                            },
                            data: this.targetCreateCount
                        },
                        {
                            name: '学生定向放课量',
                            type: 'line',
                            stack: '总量3',
                            symbol: 'none',
                            smooth: 0.3,
                            itemStyle: {
                                normal: {
                                    label: {
                                        show: true
                                    }
                                }
                            },
                            data: this.studentTargetCreateCount
                        },
                        {
                            name: '产品定向放课量',
                            type: 'line',
                            stack: '总量4',
                            symbol: 'none',
                            smooth: 0.3,
                            itemStyle: {
                                normal: {
                                    label: {
                                        show: true
                                    }
                                }
                            },
                            data: this.productTargetCreateCount
                        },
                        {
                            name: '取消课量',
                            type: 'line',
                            stack: '总量5',
                            symbol: 'none',
                            smooth: 0.3,
                            itemStyle: {
                                normal: {
                                    label: {
                                        show: true
                                    }
                                }
                            },
                            data: this.removeCount
                        }
                    ]
                }, true);
            }
        },
        mounted: function () {
            this.getItems();
        }
    }
</script>

<style scoped>
    .chart-container {
        width: 100%;
        float: left;
    }
    .el-col {
        padding: 30px 20px;
    }
</style>

```