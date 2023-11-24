<template>
    <view class="charts-box">
        <view style="min-height: 500rpx;">
            <view>
                <h2 class="text-info">山东信息</h2>
            </view>
            <qiun-data-charts canvas2d type="pie" :opts="pieOpts" :chartData="pieData" />
        </view>
        <view style="min-height: 500rpx;">
            <view>
                <h2 class="text-info">山东信息</h2>
            </view>
            <qiun-data-charts type="line" :opts="opts" :chartData="chartData" />
        </view>
        
    </view>
</template>

<script>
    import {
        getLineInfoList
    } from "@/api/system/static";
    export default {
        data() {
            return {
                query: {
                    financeCreate: "2022-01-01",
                    financeExpendTime: null,
                },
                
                pieData: {},
                pieOpts: {
                    rotate: false,
                    rotateLock: false,
                    color: ["#4B98FE", "#00D05E", "#FFAC00", "#FB6A67", "#957BFE", "#00B9FE", "#FE871B", "#00C8B0",
                        "#F674D6"
                    ],
                    padding: [15, 10, 15, 10],
                    dataLabel: true,
                    enableScroll: false,
                    legend: {
                        show: true,
                        lineHeight: 32
                    },
                    title: {
                        name: "",
                        fontSize: 15,
                        color: "#666666"
                    },
                    subtitle: {
                        name: "",
                        fontSize: 25,
                        color: "#4B98FE"
                    },
                    extra: {
                        pie: {
                            activeOpacity: 0.5,
                            activeRadius: 10,
                            offsetAngle: 0,
                            labelWidth: 15,
                            border: true,
                            borderWidth: 2,
                            borderColor: "#FFFFFF",
                        }
                    }
                },
                
                chartData: {},
                //这里的 opts 是图表类型 type="line" 的全部配置参数，您可以将此配置复制到 config-ucharts.js 文件中下标为 ['line'] 的节点中来覆盖全局默认参数。实际应用过程中 opts 只需传入与全局默认参数中不一致的【某一个属性】即可实现同类型的图表显示不同的样式，达到页面简洁的需求。
                opts: {
                    timing: "easeOut",
                    duration: 1000,
                    rotate: false,
                    rotateLock: false,
                    color: ["#1890FF", "#91CB74", "#FAC858", "#EE6666", "#73C0DE", "#3CA272", "#FC8452", "#9A60B4",
                        "#ea7ccc"
                    ],
                    padding: [15, 10, 0, 15],
                    fontSize: 13,
                    fontColor: "#666666",
                    dataLabel: false,
                    dataPointShape: false,
                    dataPointShapeType: "solid",
                    touchMoveLimit: 60,
                    enableScroll: false,
                    enableMarkLine: false,
                    legend: {
                        show: true,
                        position: "bottom",
                        float: "center",
                        padding: 5,
                        margin: 5,
                        backgroundColor: "rgba(0,0,0,0)",
                        borderColor: "rgba(0,0,0,0)",
                        borderWidth: 0,
                        fontSize: 13,
                        fontColor: "#666666",
                        lineHeight: 11,
                        hiddenColor: "#CECECE",
                        itemGap: 10
                    },
                    xAxis: {
                        disableGrid: true,
                        disabled: false,
                        axisLine: true,
                        axisLineColor: "#CCCCCC",
                        calibration: false,
                        fontColor: "#666666",
                        fontSize: 13,
                        lineHeight: 20,
                        marginTop: 0,
                        rotateLabel: false,
                        rotateAngle: 45,
                        labelCount: 3,
                        itemCount: 0,
                        boundaryGap: "center",
                        splitNumber: 5,
                        gridColor: "#CCCCCC",
                        gridType: "solid",
                        dashLength: 4,
                        gridEval: 1,
                        scrollShow: false,
                        scrollAlign: "left",
                        scrollColor: "#A6A6A6",
                        scrollBackgroundColor: "#EFEBEF",
                        title: "",
                        titleFontSize: 13,
                        titleOffsetY: 0,
                        titleOffsetX: 0,
                        titleFontColor: "#666666",
                        format: ""
                    },
                    yAxis: {
                        gridType: "dash",
                        dashLength: 2,
                        disabled: false,
                        disableGrid: false,
                        splitNumber: 5,
                        gridColor: "#CCCCCC",
                        padding: 10,
                        showTitle: false,
                        data: []
                    },
                    extra: {
                        line: {
                            type: "curve",
                            width: 2,
                            activeType: "hollow",
                            linearType: "none",
                            onShadow: false,
                            animation: "vertical"
                        },
                        tooltip: {
                            showBox: true,
                            showArrow: true,
                            showCategory: false,
                            borderWidth: 0,
                            borderRadius: 0,
                            borderColor: "#000000",
                            borderOpacity: 0.7,
                            bgColor: "#000000",
                            bgOpacity: 0.7,
                            gridType: "solid",
                            dashLength: 4,
                            gridColor: "#CCCCCC",
                            boxPadding: 3,
                            fontSize: 13,
                            lineHeight: 20,
                            fontColor: "#FFFFFF",
                            legendShow: true,
                            legendShape: "auto",
                            splitLine: true,
                            horizentalLine: false,
                            xAxisLabel: false,
                            yAxisLabel: false,
                            labelBgColor: "#FFFFFF",
                            labelBgOpacity: 0.7,
                            labelFontColor: "#666666"
                        },
                        markLine: {
                            type: "solid",
                            dashLength: 4,
                            data: []
                        }
                    }
                }
            };
        },
        onReady() {
            this.initDate();
            this.getLineInfoList();
        },
        methods: {
            initDate() {
                const currentDate = new Date();
                this.query.financeExpendTime = currentDate.toISOString().substring(0, 10); // 格式化为 YYYY-MM-DD 格式的日期字符串  
            },
            getLineInfoList() {
                getLineInfoList(this.query).then(response => {
                    let res = {
                        categories: response.dateInfo,
                        series: [{
                                name: "支出",
                                data: response.expendInfo,
                            },
                            {
                                name: "收入",
                                data: response.incomeInfo,
                            }
                        ]
                    };
                    
                    let data = response.pieInfo;
                    data.forEach(item => {
                        item.value = parseFloat(item.value);
                    });
                    console.log(data)
                    let res1 = {
                        series: [{
                            data: data
                        }]
                    };
                    this.pieData = JSON.parse(JSON.stringify(res1));
                    this.chartData = JSON.parse(JSON.stringify(res));
                })
            }
        }
    };
</script>

<style scoped>
    /* 请根据实际需求修改父元素尺寸，组件自动识别宽高 */
    .charts-box {
        width: 100%;
        height: 300px;
    }
    .text-info::before{
            content: "";
            display: block;
            top: 1px;
            border-radius: 10px;
            background-color: #0ea8e4;
            width: 5px;
            height: 20px;
            position: absolute;
            top: 10px;
            left: -12px;
    }
    .text-info{
            font-size: 20px;
            color: #1f2f3d;
            margin-left: 20px;
            line-height: 40px;
            font-weight: 700;
            white-space: nowrap;
            position: relative;
    }
</style>