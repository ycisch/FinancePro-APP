<template>
    <view class="form-container">
        <u-form :model="form" ref="uForm" :rules="rules">
            <u-form-item label="金额">
                <u--input border="bottom" placeholder="请输入金额" v-model="form.money"></u--input>
            </u-form-item>
            <u-form-item prop="type" borderBottom @click="clickType" label="所属表格">
                <u--input v-model="form.type" disabled disabledColor="#ffffff" placeholder="请选择所属表格"
                    border="none"></u--input>
                <u-icon slot="right" name="arrow-right"></u-icon>
            </u-form-item>
            <u-form-item prop="recordInfo" @click="changeRecord" borderBottom label="结账信息">
                <xfl-select ref="inputRef" placeholder="请选择结账信息" :list="recordList" :clearable="false" :showItemNum="5"
                    :focusShowList="recordList" @change="changeItem" :isCanInput="true"
                    :style_Container="'height: 30px; font-size: 16px;'">
                </xfl-select>
            </u-form-item>
            <u-form-item prop="date" borderBottom @click="clickCalendar" label="时间">
                <u--input v-model="form.date" disabled disabledColor="#ffffff" placeholder="请选择时间"
                    border="none"></u--input>
                <u-icon slot="right" name="arrow-right"></u-icon>
            </u-form-item>
            <u-form-item label="备注">
                <u--textarea v-model="form.info" placeholder="请输入内容" autoHeight></u--textarea>
            </u-form-item>
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
        addRecord
    } from "@/api/system/record";
    import {
        listFinanceAll
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
                console.log(this.types)
                return [this.types];
            }
        },
        data() {
            return {
                query: {},
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
                recordList: [],
                // u-radio-group的v-model绑定的值如果设置为某个radio的name，就会被默认选中
                value: '结清',
                showCalendar: false,
                showType: false,
                showKey: false,
                showRecord: false,
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
            changeRecord(e) {
                if (this.form.type == null || this.form.type == undefined) {
                    uni.showToast({
                        title: '请先选择表格',
                        icon: 'none'
                    });
                    return;
                }
                let query = {}
                query.financeType = this.form.typeId
                listFinanceAll(query).then(response => {

                    const savedData = response.rows.map(data => ({
                        id: data.financeId,
                        value: data.financeDec,
                        financeFlag: data.financeFlag
                    }));
                    this.recordList = savedData.filter(item => item.financeFlag === '1');
                })
                this.showRecord = true;
            },
            changeItem(e) {
                console.log(e)
                this.query.financeIds = e.id
            },
            clickKey() {
                this.showKey = true;
            },
            clickType() {
                this.showType = true;
            },
            clickCalendar() {
                this.showCalendar = true;
            },
            submit() {
                //提交数据到后台
                this.$refs.uForm.validate().then(res => {
                    uni.$u.toast('校验通过')
                    let sysRecord = {}
                    sysRecord.recordMoney = this.form.money
                    sysRecord.recordTime = this.form.date
                    sysRecord.financeIds = this.query.financeIds
                    addRecord(sysRecord).then(response => {
                        if (response.code == '500') {
                            uni.$u.toast('输入金额有误 请仔细检查')
                            return
                        } else {
                            uni.$u.toast('添加成功')
                            this.resetInfo()
                        }
                    })
                }).catch(errors => {
                    console.log(errors)
                    uni.$u.toast('校验失败')
                })
            },
            resetInfo() {
                this.query = {}
                this.form = {}
                this.recordList = []
                this.callChildMethod();
            },

            callChildMethod() {
                this.$refs.inputRef.clearInput()
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
    .form-container {
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