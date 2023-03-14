<template>
    <div class="hello">
        <el-row>
            <el-col :span="24">
                <el-select v-model="type" placeholder="请选择学期">
                    <el-option
                            label="请选择学期"
                            value="">
                    </el-option>
                    <el-option
                            v-for="(item,index) in allType"
                            :key="index + ','"
                            :label="item"
                            :value="item">
                    </el-option>
                </el-select>
                <el-select v-model="title" placeholder="请选择文章" style="margin-left: 10px">
                    <el-option
                            label="请选择标题"
                            value="">
                    </el-option>
                    <el-option
                            v-for="(item,index) in allTitle"
                            :key="index + '.'"
                            :label="item"
                            :value="item">
                    </el-option>
                </el-select>
                <el-button type="primary" @click="submit" round style="margin-left: 10px">抽取题目</el-button>
                <el-button type="primary" @click="exportDocx" round style="margin-left: 10px">导出题库</el-button>
                <el-button type="primary" @click="see" round style="margin-bottom: 10px">查看题库</el-button>
            </el-col>
        </el-row>
        <el-row v-if="showIndex == 1">
            <el-col :span="8" v-for="(item,index) in allContent" :key="index">
                <el-card class="box-card">
                    <div slot="header" class="clearfix">
                        <span>{{item.title}}  -  {{item.author}}</span>
                    </div>
                    <div v-for="(item2,index2) in item.content" :key="index +'!' + index2" class="text item">
                        {{ item2 }}
                    </div>
                </el-card>
            </el-col>
        </el-row>
        <el-row v-if="showIndex == 2">
            <el-col :span="18" v-for="(item,index) in allContent" :key="index" :offset="3">
                <el-card class="box-card">
                    <div slot="header" class="clearfix">
                        <span>{{item.title}}  -  {{item.author}}</span>
                    </div>
                    <div v-for="(item2,index2) in item.content" :key="index +'!' + index2" class="text item">
                        {{ item2.value }}
                    </div>
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    import dataJson from "../../public/data.json"
    import {ExportBriefDataDocx} from '../utils/exportBriefDataDocx2'

    export default {
        name: 'HelloWorld',
        data() {
            return {
                regExp: /[。！；]/,
                regExp2: /[，？]/,
                msg: {},
                data: dataJson,
                allTitle: [],
                allType: [],
                type: "",
                title: '',
                allContent: [],
                tableData: [],
                docxData: {
                    tableData: [],
                    year: 0,
                    month: 0,
                    date: 0
                },
                showIndex: 1
            }
        },
        created() {
            this.getAllTitle();
            this.getAllType();
            this.initDate();
        },
        methods: {
            exportDocx() {
                if (this.docxData.tableData.length == 0) {
                    this.$message({
                        message: '请先抽取题目',
                        type: 'error'
                    });
                    return
                }
                ExportBriefDataDocx('/templateDoc.docx', this.docxData, '默写文档' + this.getDate() + '.docx') // text.docx放在了根目录下的public文件夹下
            },
            initDate() {
                let date = new Date();
                this.docxData.year = date.getFullYear();
                this.docxData.month = date.getMonth() + 1
                this.docxData.date = date.getDate();
            },
            getDate() {
                return `${this.docxData.year}年${this.docxData.month}月${this.docxData.date}日`
            },
            submit() {
                this.tableData = []
                this.data.filter(item => {
                    return item.type == this.type || item.title == this.title || (this.title == "" && this.type == "")
                }).map(item => {
                    item.content.map(item1 => {
                            let item2 = JSON.parse(JSON.stringify(item1))
                            let firstIndex = Math.floor(Math.random() * item2.length);
                            item2[firstIndex] = "___________________"
                            if (item2.length > 2) {
                                while (true) {
                                    let secondIndex = Math.floor(Math.random() * item2.length);
                                    if (secondIndex == firstIndex) {
                                        continue
                                    }
                                    item2[secondIndex] = "___________________"
                                    break
                                }
                            }
                            this.tableData.push({value: item2.join("，")})
                        }
                    )
                })
                this.shuffle(this.tableData)
                this.allContent = [{
                    title: '抽取到的题目',
                    author: this.getDate(),
                    content: this.tableData
                }]
                this.docxData.tableData = this.tableData
                this.$message({
                    message: '筛选题目完成',
                    type: 'success'
                });
                this.showIndex = 2;
            },
            shuffle(arr) {
                var length = arr.length,
                    randomIndex,
                    temp;
                while (length) {
                    randomIndex = Math.floor(Math.random() * (length--));
                    temp = arr[randomIndex];
                    arr[randomIndex] = arr[length];
                    arr[length] = temp
                }
                return arr;
            },
            see() {
                this.allContent = this.allContent = this.data.filter(item => {
                    return item.type == this.type || item.title == this.title || (this.title == "" && this.type == "")
                }).map(item => {
                    let obj = {};
                    obj.title = item.title
                    obj.author = item.author
                    obj.type = item.type
                    obj.content = item.content.map(item =>
                        item.join("，")
                    )
                    return obj
                })
                this.$message({
                    message: '查询题库完成',
                    type: 'success'
                });
                this.showIndex = 1;
            },
            analysis(title, author, type, content) {
                let obj = {};
                obj.title = title
                obj.author = author
                obj.type = type
                let contents = content.split(this.regExp).map(item => item.replace("\n", ""))
                    .map(item => item.replace("“", ""))
                    .map(item => item.split(this.regExp2))
                    .filter(item => item != "")
                obj.content = contents;
                this.msg = obj
                return obj
            },
            getAllTitle() {
                this.allTitle = this.data.map(item => item.title)
            },
            getAllType() {
                let allTypeTemp = this.data.map(item => item.type)
                allTypeTemp.forEach(item => {
                    if (this.allType.indexOf(item) < 0) {
                        this.allType.push(item)
                    }
                })
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    .text {
        font-size: 14px;
    }

    .item {
        margin-bottom: 18px;
    }

    .clearfix:before,
    .clearfix:after {
        display: table;
        content: "";
    }

    .clearfix:after {
        clear: both
    }

    .box-card {
        width: 90%;
        margin: 10px auto;
    }
</style>
