<template>
    <view>
        <view class="" style="min-height: 500rpx;">
            <qiun-data-charts canvas2d type="pie" :opts="expendPieOpts" :chartData="expendPieData" />
        </view>
        <view>
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
                
                // 活跃用户
                expendColumnData: {},
                expendColumnOpts: {
                    color: ["#4B98FE", "#00D05E", "#FFAC00", "#FB6A67", "#957BFE", "#00B9FE", "#FE871B", "#00C8B0",
                        "#F674D6"
                    ],
                    padding: [15, 15, 0, 15],
                    dataLabel: false,
                    xAxis: {
                        disableGrid: true,
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
            this.getStaticInfoList();
        },
        mounted() {
           
        },
        methods: {
            getStaticInfoList(){
              getStaticInfoList('expend').then(response=>{
                  let data = response.pieInfo;
                  data.forEach(item => {
                      item.value = parseFloat(item.value);
                  });
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

</style>