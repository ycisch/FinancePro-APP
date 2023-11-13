<template>
	<view>
		<uni-ec-canvas style="width: 500px; height: 500px;" class="uni-ec-canvas" id="uni-ec-canvas" ref="canvas" canvas-id="uni-ec-canvas" :ec="ec">
		</uni-ec-canvas>
	</view>
</template>
 
<script>
	import uniEcCanvas from '@/components/uni-ec-canvas/uni-ec-canvas.vue'
	import * as echarts from '@/components/uni-ec-canvas/echarts'
	let chart = null
	export default {
        name:"testChart",
		components: {
			uniEcCanvas
		},
		data() {
			return {
				ec: {
					lazyLoad: true
				},
				option: {
					tooltip: {
						trigger: 'axis',
						axisPointer: { // 坐标轴指示器，坐标轴触发有效
							type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
						}
					},
					grid: {
						left: '40',
						right: '40',
						bottom: '3%',
						containLabel: true
					},
					xAxis: {
						type: 'category',
						axisTick: {
							show: false,
						},
						nameTextStyle: {
							color: '#666666'
						},
						axisLabel: {
							show: true,
							textStyle: {
								color: '#666',
								fontSize: '12',
								fontWeight:'bold'
							}
						},
						axisLine: {
							lineStyle: {
								color: '#666',
								width: 1
							}
						},
						data: ["寿险", "重疾", "意外", "医疗", "年金"],
					},
					yAxis: {
						type: 'value',
						axisLine: {
							show: false, //y轴线消失
 
						},
						axisLabel: {
							show: true,
							textStyle: {
								color: '#666',
								fontSize: '11'
							}
						},
 
						axisTick: {
							show: false,
						},
					},
					series: [{
						barWidth: 20,
						type: 'bar',
						data: [20, 50, 40, 10, 20],
						itemStyle: {
							normal: {
								//每根柱子颜色设置
								color: function(params) {
									const colorList = ["#FFC600", "#21A5FF", "#FF6000", "#00D69C",
										"#998BFF"
									];
									return colorList[params.dataIndex];
								}
							}
						},
						label: {
							show: true,
							position: 'top',
							formatter: '{c}万',
							color: '#666666',
							fontStyle: 'PingFang SC',
							fontWeight: 'bold',
							fontSize:'8'
						}
					}]
				},
			}
		},
		methods: {
			initChart(canvas, width, height, canvasDpr) {
				console.log(canvas, width, height, canvasDpr)
				chart = echarts.init(canvas, null, {
					width: width,
					height: height,
					devicePixelRatio: canvasDpr
				})
				canvas.setChart(chart)
				chart.setOption(this.option)
				return chart
			},
		},
		created() {
			setTimeout(() => {
				console.log(this.$refs)
                this.$refs.canvas.init(this.initChart)
			}, 2000)
			
		},
	}
</script>
<style>
	.uni-ec-canvas {
		width: 100%;
		height: 500rpx;
		display: block;
		margin-top: 30rpx;
	}
</style>