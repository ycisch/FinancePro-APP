<template>
    <view>
        <view class="" style="min-height: 500rpx;">
            <qiun-data-charts canvas2d type="pie" :opts="memberOpts" :chartData="memberChartData" />
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
                memberChartData: {},
                memberOpts: {
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
                        position: "left",
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

            };
        },
        onShow() {},
        onLoad() {},
        onReady() {
            this.getStaticInfoList();
            this.drawMemberCharts(); //集团成员
        },
        mounted() {},
        methods: {

            getStaticInfoList() {
                getStaticInfoList().then(response => {
                    
                })
            },

            // 集团成员 绘制
            drawMemberCharts() {
                let res = {
                    series: [{
                        data: [{
                            "name": "生产",
                            "value": 3000
                        }, {
                            "name": "运维",
                            "value": 2600
                        }, {
                            "name": "销售",
                            "value": 2200
                        }, {
                            "name": "研发",
                            "value": 900
                        }, {
                            "name": "管理",
                            "value": 600
                        }]
                    }]
                };
                this.memberChartData = JSON.parse(JSON.stringify(res));
            },

        },
    };
</script>
<style lang="scss" scoped>

</style>