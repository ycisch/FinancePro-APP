<template>
    <view class="form-container">
        <u-form :model="form" ref="uForm">
            <u-form-item label="金额">
                <u--input  border="bottom" placeholder="请输入金额"
                    v-model="form.money"></u--input>
            </u-form-item>
            <u-form-item prop="type" borderBottom @click="clickType" label="分类">
                <u--input v-model="form.type" disabled disabledColor="#ffffff"
                    placeholder="请选择所属表格" border="none"></u--input>
                <u-icon slot="right" name="arrow-right"></u-icon>
            </u-form-item>
            <u-form-item  prop="date" borderBottom @click="clickCalendar" label="时间">
                <u--input  v-model="form.date" placeholder="请选择时间"
                    border="none"></u--input>
                <u-icon slot="right" name="arrow-right"></u-icon>
            </u-form-item>
            <u-form-item label="状态">
                <u-radio-group  v-model="form.flag" placement="row"
                    @change="radioGroupChange">
                    <u-radio :customStyle="{marginBottom: '0rpx',marginLeft:'8rpx'}" v-for="(item, index) in radioList"
                        :key="index" :label="item.name" :name="item.id" @change="radioChange">
                    </u-radio>
                </u-radio-group>
            </u-form-item>
            <u-form-item label="备注">
                <u--textarea  v-model="form.info" placeholder="请输入内容"
                    autoHeight></u--textarea>
            </u-form-item>
            <view v-show="flag">
                <u-form-item label="运费">
                    <u--input  border="bottom" placeholder="请输入内容"
                        v-model="form.freight"></u--input>
                </u-form-item>
                <u-form-item label="下车费">
                    <u--input  border="bottom" placeholder="请输入内容"
                        v-model="form.laborCost"></u--input>
                </u-form-item>
            </view>
        </u-form>
        <u-button type="primary" text="提交" customStyle="margin-top: 50rpx" @click="submit"></u-button>

        <u-picker :defaultIndex="[0]" :show="showType" :columns="computedTypes" @cancel="typeCancel" @close="typeCancel"
            @confirm="typeConfirm" closeOnClickOverlay @change="typeChange" keyName="typeName"></u-picker>
        <u-datetime-picker :show="showCalendar" :value="nowDate" mode="date" closeOnClickOverlay @confirm="confirm"
            @cancel="close" @close="close"></u-datetime-picker>

    </view>
</template>

<script>
    import {
        listFinance,
        addFinance,
        getFinance
    } from "@/api/system/finance";
    export default {
        name: 'addRecord',
        props: {
            types: {
                type: Array,
                default: function() {
                    return [{
                            typeName: "测试1",
                            id: "1",
                        },
                        {
                            typeName: "测试2",
                            id: "2",
                        }
                    ];
                }
            },
            tabIndex: {
                type: Number,
            }
        },
        computed: {
            computedTypes() {
                return [this.types];
            }
        },
        data() {
            return {
                nowDate: Number(new Date()),
                money: "",
                form: {
                    flag: 0
                },
                radioList: [{
                        name: '结清',
                        id: 0,
                    },
                    {
                        name: '未结',
                        id: 1,
                    }
                ],
                // u-radio-group的v-model绑定的值如果设置为某个radio的name，就会被默认选中
                value: '结清',
                showCalendar: false,
                showType: false,
                showKey: false,
                flag: false,
                tabInfo: "",
                rules: {
                    date: {
                        type: 'string',
                        min: 2,
                        required: true,
                        message: '请选择时间',
                        trigger: ['change']
                    },
                    type: {
                        type: 'string',
                        min: 1,
                        required: true,
                        message: '请选择所属表格',
                        trigger: ['blur', 'change']
                    },
                },
            }
        },
        onReady() {
            // 如果需要兼容微信小程序，并且校验规则中含有方法等，只能通过setRules方法设置规则
            this.$refs.uForm.setRules(this.rules)
        },
        methods: {
            clickKey(){
              this.showKey = true;  
            },
            clickType(){
              this.showType = true;  
            },
            clickCalendar(){
              this.showCalendar = true;  
            },
            submit() {
                //提交数据到后台
                this.$refs.uForm.validate().then(res => {
                    uni.$u.toast('校验通过')
                    let sysFinance = {}
                    if (this.tabIndex == 0) {
                        sysFinance.financeIncome = this.form.money
                    } else if (this.tabIndex == 1) {
                        sysFinance.financeExpenditure = this.form.money
                    }
                    sysFinance.financeCreate = this.form.date
                    sysFinance.financeFlag = this.form.flag
                    sysFinance.financeType = this.form.typeId
                    sysFinance.financeDec = this.form.info
                    addFinance(sysFinance).then(response => {
                        console.log(response)
                    })
                }).catch(errors => {
                    console.log(errors)
                    uni.$u.toast('校验失败')
                })
            },
            // 选中某个单选框时，由radio时触发
            radioChange(e) {
                // console.log(e);
            },
            // 选中任一radio时，由radio-group触发
            radioGroupChange(e) {
                this.form.flag = e
            },
            hideKeyboard() {
                uni.hideKeyboard()
            },
            confirm(e) {
                console.log(this.formatDate(e.value))
                this.showCalendar = false
                this.form.date = uni.$u.timeFormat(e.value, 'yyyy-mm-dd')
            },
            close() {
                this.showCalendar = false
            },
            typeCancel(e) {
                console.log(e)
                this.showType = false

            },
            typeChange(e) {
                console.log(e)
            },
            typeConfirm(e) {
                if (e.value[0].typeFlag == 1) {
                    this.flag = true
                } else {
                    this.flag = false
                }
                this.form.type = e.value[0].typeName
                this.form.typeId = e.value[0].typeId
                this.showType = false
            },
            keyConfirm(e) {
                this.showKey = false
                console.log(e)
            },
            keyChange(e) {
                this.money = this.money + e;
                console.log(this.formateNum())
                this.form.money = this.formateNum()
            },
            keyBackSpace(e) {
                this.money = this.money.slice(0, -1)
                this.form.money = this.formateNum()
                console.log(e)
            },
            keyCancel(e) {
                this.showKey = false
                console.log(e)
            },
            result(time, mode) {
                const timeFormat = uni.$u.timeFormat,
                    toast = uni.$u.toast
                switch (mode) {
                    case 'datetime':
                        return timeFormat(time, 'yyyy-mm-dd hh:MM')
                    case 'date':
                        return timeFormat(time, 'yyyy-mm-dd')
                    case 'year-month':
                        return timeFormat(time, 'yyyy-mm')
                    case 'time':
                        return time
                    default:
                        return ''
                }
            },
            formatDate(timestamp) {
                const date = new Date(parseInt(timestamp));
                const year = date.getFullYear();
                const month = String(date.getMonth() + 1).padStart(2, '0'); // 将月份转换为两位数，不足两位用0填充
                const day = String(date.getDate()).padStart(2, '0'); // 将日期转换为两位数，不足两位用0填充
                return `${year}/${month}/${day}`;
            },
            formateNum() {
                const num = Number(this.money);
                if (isNaN(num)) {
                    return 'Invalid Number';
                }
                return (Math.floor(num * 100) / 100).toFixed(2);
            }
        }
    }
</script>

<style>
    .form-container{
        margin-left: 10rpx;
        margin-right: 10rpx;
    }
    .icon-wrapper {
        position: absolute;
        left: 0;
        top: 100rpx;
        /* 根据需要调整图标的垂直位置 */
    }
</style>