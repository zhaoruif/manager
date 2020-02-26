<template>
    <div>
        <div class="container">
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                >批量删除</el-button>
                <el-upload
                    class="upload"
                    action
                    :multiple="false"
                    :show-file-list="false"
                    :drag="false"
                    :http-request="httpRequest"
                >
                    <el-button slot="trigger" size="small" type="primary">导入</el-button>
                </el-upload>
                <el-button type="primary" icon="el-icon-search" @click="handleDownload">导出表格</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="name" label="用户名"></el-table-column>
                <el-table-column label="账户余额">
                    <template slot-scope="scope">￥{{scope.row.money}}</template>
                </el-table-column>
                <el-table-column label="头像(查看大图)" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.thumb"
                            :preview-src-list="[scope.row.thumb]"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="address" label="地址"></el-table-column>
                <el-table-column label="状态" align="center">
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.state==='成功'?'success':(scope.row.state==='失败'?'danger':'')"
                        >{{scope.row.state}}</el-tag>
                    </template>
                </el-table-column>

                <el-table-column prop="date" label="注册时间"></el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="地址">
                    <el-input v-model="form.address"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
import ExportExcel from './Export2Excel';
import XLSX from 'xlsx';
export default {
    name: 'basetable',
    components: { ExportExcel },
    data() {
        return {
            query: {
                address: '',
                name: '',
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            pageTotal: 0,
            form: {},
            idx: -1,
            id: -1
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取 easy-mock 的模拟数据
        getData() {
            fetchData(this.query).then(res => {
                console.log(res);
                this.tableData = res.list;
                this.pageTotal = res.pageTotal || 50;
            });
        },
        // 触发搜索按钮
        handleSearch() {
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    this.$message.success('删除成功');
                    this.tableData.splice(index, 1);
                })
                .catch(() => {});
        },
        /**
         * handleDownload{导出Excel方法}
         * param{this.tValue}数据
         */
        handleDownload() {
            var that = this;
            this.downloadLoading = true;
            // this.loading = true;
            for (var i in this.tableData) {
                this.tableData[i].ids = parseInt(i) + 1;
            }
            import('./Export2Excel.js').then(excel => {
                const data = this.formatJson(['ids', 'name', 'money', 'address', 'thumb', 'state', 'date'], this.tableData);
                excel.export_json_to_excel({
                    header: ['序号', '姓名', '余额', '地址', '头像', '状态', '时间'],
                    data,
                    filename: '导出Excel',
                    autoWidth: this.autoWidth,
                    bookType: this.bookType
                });
                this.downloadLoading = false;
                this.loading = false;
            });
        },
        /**格式化 */
        formatJson(filterVal, jsonData) {
            return jsonData.map(v =>
                filterVal.map(j => {
                    return v[j];
                })
            );
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].name + ' ';
            }
            this.$message.error(`删除了${str}`);
            this.multipleSelection = [];
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.editVisible = false;
            this.$message.success(`修改第 ${this.idx + 1} 行成功`);
            this.$set(this.tableData, this.idx, this.form);
        },
        // 分页导航
        handlePageChange(val) {
            this.$set(this.query, 'pageIndex', val);
            this.getData();
        },
        /**导入 */
        httpRequest(e) {
            let file = e.file; // 文件信息
            console.log('e: ', e);
            console.log('file: ', e.file);
            var realname = '';
            var planners = '';
            var deptId = localStorage.getItem('deptId');
            if (deptId != null && deptId == 3) {
                realname = localStorage.getItem('realname');
                planners = localStorage.getItem('saleIds');
            } else {
                realname = '';
            }
            console.log('realname', realname, 'planners', planners);
            if (!file) {
                // 没有文件
                return false;
            } else if (!/\.(xls|xlsx)$/.test(file.name.toLowerCase())) {
                // 格式根据自己需求定义
                this.$message.error('上传格式不正确，请上传xls或者xlsx格式');
                return false;
            }

            const fileReader = new FileReader();
            fileReader.onload = ev => {
                try {
                    const data = ev.target.result;
                    const workbook = XLSX.read(data, {
                        type: 'binary' // 以字符编码的方式解析
                    });
                    const exlname = workbook.SheetNames[0]; // 取第一张表
                    console.log('exlname', workbook.Sheets[exlname]);
                    const exl = XLSX.utils.sheet_to_json(workbook.Sheets[exlname]); // 生成json表格内容
                    // 将 JSON 数据挂到 data 里
                    console.log('exl', exl[0]);
                    if (
                        workbook.Sheets[exlname].A1.h != '姓名' ||
                        workbook.Sheets[exlname].B1.h != '余额' ||
                        workbook.Sheets[exlname].C1.h != '地址' ||
                        workbook.Sheets[exlname].D1.h != '头像' ||
                        workbook.Sheets[exlname].E1.h != '状态' ||
                        workbook.Sheets[exlname].F1.h != '时间'
                    ) {
                        this.$message.error('该表格格式不正确');
                        return;
                    }
                    var formdate = {};
                    for (var i in exl) {
                        (exl[i].name = exl[i].姓名),
                            (exl[i].money = exl[i].余额),
                            (exl[i].address = exl[i].地址),
                            (exl[i].thumb = exl[i].头像),
                            (exl[i].state = exl[i].状态),
                            (exl[i].date = exl[i].时间);
                    }
                    for (var j in exl) {
                        this.tableData.push(exl[j]);
                    }
                    return;

                    this.tableData = exl;
                    // document.getElementsByName('file')[0].value = '' // 根据自己需求，可重置上传value为空，允许重复上传同一文件
                } catch (e) {
                    console.log('出错了：：');
                    return false;
                }
            };
            fileReader.readAsBinaryString(file);
        }
    }
};
</script>

<style>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
.upload {
    display: inline-block !important;
    position: relative;
    top: 13px;
    margin-left: 1px;
    margin-right: 1px;
}
.el-upload--text {
    /* background-color: #fff; */
    border: 0px !important;
    border-radius: 0px !important;
    display: inline-block !important;
    /* -webkit-box-sizing: border-box; */
    width: 80px !important;
    height: 30px !important;
    text-align: center;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}
</style>
