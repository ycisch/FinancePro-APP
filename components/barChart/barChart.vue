<template>
    <view>
        <view style="width: 100%; height:500rpx">

            <view v-if="showInfo">
                <l-echart id="myChartChange" ref="chart">
                
                </l-echart>
            </view>
            <view v-if="showInfo == false" style="height: 750rpx">
                <l-echart id="lineChartChange" ref="chart1">
                
                </l-echart>
            </view>
        </view>
    </view>
</template>

<script>
    import * as echarts from 'echarts'
    import {
        getStaticInfoList,
        getLineInfoList
    } from "@/api/system/static";


    const info = {
        "expend": {
            name: "支出"
        },
        "income": {
            name: "收入"
        },
        "shangdong": {
            name: "山东"
        },
        "xinjiang": {
            name: "新疆"
        }
    }

    export default {
        props: {
            type: {
                type: String,
                default: "expend"
            }
        },
        data() {
            return {
                chart: null,
                linChart:null,
                barChartData: {},
                lineChartData: {},
                show: false,
                showInfo: true,
                query: {
                    financeCreate: "2022-01-01",
                    financeExpendTime: "",
                },
            };
        },
        // 组件能被调用必须是组件的节点已经被渲染到页面上
        // 1、在页面mounted里调用，有时候mounted 组件也未必渲染完成
        mounted() {
            // init(echarts, theme?:string, opts?:{}, chart => {})
            // echarts 必填， 非nvue必填，nvue不用填
            // theme 可选，应用的主题，目前只支持名称，如：'dark'
            // opts = { // 可选
            //  locale?: string  // 从 `5.0.0` 开始支持
            // }
            // chart => {} ， callback 返回图表实例
            this.init();
            // this.chart = this.$echarts.init(document.getElementById('myChartChange'), 'macarons');
            // this.lineChart = this.$echarts.init(document.getElementById('lineChartChange'), 'macarons');
            this.getStaticList('expend')
        },
        // watch: {
        //     type(newValue, oldValue) {
        //         console.log(newValue + "  " + oldValue)
        //         this.getStaticList(newValue);
        //     },
        // },
        methods: {
            init() {
                const currentDate = new Date();
                this.query.financeExpendTime = currentDate.toISOString().substring(0, 10); // 格式化为 YYYY-MM-DD 格式的日期字符串  
            },
            getStaticList(type) {
                if (type == 'expend' || type == 'income') {
                    this.showInfo = true;
                    getStaticInfoList(type).then(response => {
                        this.barChartData.arrayName = response.arrayName;
                        this.barChartData.arrayMoney = response.arrayMoney;
                        this.barChartData.pieInfo = response.pieInfo;
                        if (response.arrayName.length == 0) {
                            this.show = false;
                            return;
                        }
                        this.show = true;
                        this.$refs.chart.init(echarts, chart => {
                            chart.setOption(this.setOptions(this.barChartData));
                        });
                    })
                }
                if (type == 'shangdong' || type == 'xinjiang') {
                    this.showInfo = false;
                    getLineInfoList(this.query).then(response => {
                        this.lineChartData.dateInfo = response.dateInfo;
                        this.lineChartData.expendInfo = response.expendInfo;
                        this.lineChartData.incomeInfo = response.incomeInfo;
                        if (response.dateInfo.length == 0) {
                            this.show = false;
                            return;
                        }
                        this.show = true;
                        this.$refs.chart1.init(echarts, chart => {
                            chart.setOption(this.setLineOptions(this.lineChartData));
                        });
                    })
                }
            },

            setLineOptions({dateInfo, expendInfo, incomeInfo}={}) {
                let option = {
                    tooltip: {
                        trigger: 'axis'
                    },
                    xAxis: {
                        type: 'category',
                        data: dateInfo
                    },
                    yAxis: {
                        type: 'value'
                    },
                    series: [{
                            name: "出款",
                            data: expendInfo,
                            type: 'line'
                        },
                        {
                            name: "回款",
                            data: incomeInfo,
                            type: 'line'
                        }
                    ]
                };
                return option;
            },

            setOptions({
                arrayName,
                arrayMoney
            } = {}) {
                let option = {
                    tooltip: {
                        trigger: 'axis',
                        axisPointer: {
                            type: 'shadow'
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis: [{
                        type: 'category',
                        data: arrayName,
                        axisTick: {
                            alignWithLabel: true
                        },
                        axisLabel: {
                            rotate: -30
                        }
                    }],
                    yAxis: [{
                        type: 'value'
                    }],
                    series: [{
                        name: '金额',
                        type: 'bar',
                        barWidth: '60%',
                        data: arrayMoney
                    }]
                };
                return option;
            },
        }
    }
</script>

<style>

</style>