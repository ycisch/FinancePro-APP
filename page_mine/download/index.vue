<template>
    <view class="content">
        <u-button v-for="(item, index) in types" :customStyle="buttonStyle" type="primary" :text="item.text"
            @click="tableToExcel(item)"></u-button>
        <view class="tip">导出文件为:{{ successTip }}</view>
    </view>
</template>

<script>
    import {
        getTypeList
    } from "@/api/system/redis";

    import {
        getExcelInfo
    } from "@/api/system/static";
    export default {

        data() {
            return {
                title: '导出excel',
                successTip: '',
                types: [],
                fileName: "",
                buttonStyle:{
                    marginTop:'30rpx',
                    marginLeft: '30rpx',
                    marginRight: '30rpx',
                    fontSize: '12px',
                }
            };
        },
        mounted() {
            this.initData()
        },
        methods: {
            initData() {
                getTypeList().then(response => {
                    // 使用 map 函数进行转换
                    this.types = response.rows.map((row, index) => {
                        return {
                            value: row.typeId,
                            text: row.typeName
                        };
                    });
                });
            },
            tableToExcel(e) {
                this.fileName = e.text
                let str = ``
                if (e.value == 21) {
                    str = `
                    <tr>
                      <td style="text-align:center;font-weight:bold;">日期</td>
                      <td style="text-align:center;font-weight:bold;">收入</td>
                      <td style="text-align:center;font-weight:bold;">支出</td>
                      <td style="text-align:center;font-weight:bold;">备注信息</td>
                    </tr>`;
                } else {
                    str = `
                    <tr>
                      <td style="text-align:center;font-weight:bold;">日期</td>
                      <td style="text-align:center;font-weight:bold;">收入</td>
                      <td style="text-align:center;font-weight:bold;">支出</td>
                      <td style="text-align:center;font-weight:bold;">备注</td>
                      <td style="text-align:center;font-weight:bold;">结账记录</td>
                      <td style="text-align:center;font-weight:bold;"></td>
                    </tr>`;
                }
                // 要导出的json数据
                let jsonData = [];
                let query = {}
                query.typeId = e.value
                getExcelInfo(query).then(res => {
                    if (e.value == 21) {
                        jsonData = res.map(row => ({
                            staticDate: row.staticDate,
                            incomeMoney: row.incomeMoney == null ? "" : row.incomeMoney,
                            expendMoney: row.expendMoney == null ? "" : row.expendMoney,
                            staticDesc: row.staticDesc == null ? "" : row.staticDesc,
                        }));
                    } else {
                        jsonData = res.map(row => ({
                            staticDate: row.staticDate,
                            incomeMoney: row.incomeMoney == null ? "" : row.incomeMoney,
                            expendMoney: row.expendMoney == null ? "" : row.expendMoney,
                            staticDesc: row.staticDesc == null ? "" : row.staticDesc,
                            recordInfo: row.recordInfo == null ? "" : row.recordInfo,
                            staticInfo: row.staticInfo == null ? "" : row.staticInfo,
                        }));
                    }

                    for (let i = 0; i < jsonData.length; i++) {
                        str += '<tr>';
                        for (let item in jsonData[i]) {
                            // 增加\t为了不让表格显示科学计数法或者其他格式
                            str += `<td style="text-align:center;">${jsonData[i][item] + '\t'}</td>`;
                        }
                        str += '</tr>';
                    }

                    let template = `<html xmlns="http://www.w3.org/1999/xhtml">
                    <head>
                        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
                        <style>
                            table {
                                font-family: Arial, sans-serif;
                                border-collapse: collapse;
                            }
                            td, th {
                                border: 1px solid #dddddd;
                                text-align: left;
                                padding: 8px;
                                font-size: 14px;
                            }
                        </style>
                    </head>
                    <body><table>${str}</table></body></html>`;

                    // 下载模板
                    this.exportFile(template);

                })

            },
            exportFile(fileData, documentName = '项目Excel文件') {
                // 创建根目录前先检查目录是否存在
                uni.getFileSystemManager().access({
                    path: `${uni.env.USER_DATA_PATH}/${documentName}`,
                    success: () => {
                        // 根目录已存在，继续创建年月目录和文件
                        this.createFile(fileData, documentName);
                    },
                    fail: () => {
                        // 根目录不存在，创建根目录
                        uni.getFileSystemManager().mkdir({
                            dirPath: `${uni.env.USER_DATA_PATH}/${documentName}`,
                            recursive: true,
                            success: () => {
                                // 根目录创建成功，继续创建年月目录和文件
                                this.createFile(fileData, documentName);
                            },
                            fail: (mkdirErr) => {
                                console.error('创建根目录失败：', mkdirErr);
                            }
                        });
                    }
                });
            },
            createFile(fileData, documentName) {
                // 文件路径
                const filePath = `${uni.env.USER_DATA_PATH}/${documentName}/${this.getFileName()}.xls`;

                // 写入文件
                uni.getFileSystemManager().writeFile({
                    filePath,
                    data: fileData,
                    encoding: 'utf8',
                    success: () => {
                        // 打开文档
                        uni.openDocument({
                            filePath,
                            showMenu: true,
                            fileType: 'xls',
                            success: () => {
                                uni.showToast({
                                    title: '成功导出',
                                    icon: 'success'
                                });
                                this.successTip = `文件位置：${filePath}`;
                            },
                            fail: (openErr) => {
                                console.error('打开文件失败：', openErr);
                            }
                        });
                    },
                    fail: (writeErr) => {
                        console.error('写入文件失败：', writeErr);
                    }
                });
            },
            getYearMonth() {
                let date = new Date();
                let year = date.getFullYear();
                let month = date.getMonth() + 1;
                return `${year}年${month}月`;
            },
            getFileName() {
                return this.fileName;
            }
        }
    };
</script>
<style scoped>
    .content {
        display: flex;
        flex-wrap: wrap;
    }

    .button-info {
        margin-top: 30rpx;
        font-size: 12px;
    }
</style>