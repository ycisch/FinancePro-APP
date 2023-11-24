<template>
    <view>
        <view style="min-height: 500rpx;">
            <view>
                <h2 class="text-info">{{typeName}}信息</h2>
            </view>
            <qiun-data-charts canvas2d type="pie" :opts="expendPieOpts" :chartData="expendPieData" />
        </view>
        <view class="column">
            <view>
                <h2 class="text-info">{{typeName}}信息</h2>
            </view>
            <qiun-data-charts canvas2d type="column" :opts="expendColumnOpts" :chartData="expendColumnData" />
        </view>

    </view>
</template>

<script>
    import {
        getStaticInfo,
        getStaticInfoList
    } from "@/api/system/static";
    export default {
        data() {
            return {
                cWidth: 750,
                cHeight: 500,
                type: "expend",
                typeName: "支出",
                expendPieData: {},
                expendPieOpts: {
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

                expendColumnData: {},
                expendColumnOpts: {
                    color: ["#4B98FE", "#00D05E", "#FFAC00", "#FB6A67", "#957BFE", "#00B9FE", "#FE871B", "#00C8B0",
                        "#F674D6"
                    ],
                    padding: [15, 15, 0, 15],
                    dataLabel: false,
                    xAxis: {
                        disableGrid: true,
                        rotateLabel: true,
                        rotateAngle: -35,
                    },
                    yAxis: {
                        gridColor: "rgba(230,230,230,0.6)",
                        disabled: false,
                        disableGrid: false,
                        gridType: 'dash',
                        dashLength: '4',
                        data: [{
                            axisLineColor: "#FFFFFF",
                        }, ],
                    },
                    legend: {
                        show: true,
                        position: "top",
                        float: "right",
                    },
                    extra: {
                        column: {
                            type: "stack",
                            width: 16,
                            activeBgColor: "#000000",
                            activeBgOpacity: 0.04,
                        }
                    },
                },



            };
        },
        onShow() {},
        onLoad() {},
        onReady() {
            //这里的 750 对应 css .charts 的 width
            this.cWidth = uni.upx2px(750);
            //这里的 500 对应 css .charts 的 height
            this.cHeight = uni.upx2px(500)
            this.getStaticInfoList();
        },
        mounted() {},
        onLoad(e) {
            this.type = e.type;
            if (this.type == 'expend') {
                this.typeName = '支出'
            } else if (this.type == 'income') {
                this.typeName = '收入'
            }
        },
        methods: {
            getStaticInfoList() {
                getStaticInfoList(this.type).then(response => {
                    let data = response.pieInfo;
                    data.forEach(item => {
                        item.value = parseFloat(item.value);
                    });
                    console.log(data)
                    let res = {
                        series: [{
                            data: data
                        }]
                    };


                    let resColumn = {
                        categories: response.arrayName,
                        series: [{
                            name: "金额",
                            data: response.arrayMoney,
                            legendShape: "circle",
                        },{
                            name: "总金额",
                            data: response.arrayMoneyAll,
                            legendShape: "circle",
                        },{
                            name: "结账记录",
                            data: response.recordMoney,
                            legendShape: "circle",
                        }
                        ]
                    };

                    this.expendColumnData = JSON.parse(JSON.stringify(resColumn));
                    this.expendPieData = JSON.parse(JSON.stringify(res));
                })
            },

        },
    };
</script>
<style lang="scss" scoped>
    .column {
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