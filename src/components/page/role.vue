<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i>角色管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>

        <div class="container">
            <div class="handle-box">
                <el-button type="primary" icon="delete" class="handle-del mr10" @click="delBatch">批量删除</el-button>
                <el-button type="primary" icon="add" class="handle-del mr10" @click="toAdd">新增</el-button>
            </div>

            <el-table :data="data" border style="width: 100%" ref="multipleTable" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column prop="name" label="名称" width="120">
                </el-table-column>
                <el-table-column prop="url" label="url" >
                </el-table-column>
                <el-table-column prop="pid" label="父菜单" >
                </el-table-column>
                <el-table-column prop="order" label="序号" >
                </el-table-column>
                <el-table-column label="操作" width="180">
                    <template slot-scope="scope">
                        <el-button size="small" @click="toEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button size="small" type="danger" @click="toDel(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next" :total="1000">
                </el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="角色名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="url">
                    <el-input v-model="form.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单">
                    <el-input v-model="form.pid"></el-input>
                </el-form-item>
                <el-form-item label="序号">
                    <el-input v-model="form.order"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="edit">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 新增弹出框 -->
        <el-dialog title="新增菜单" :visible.sync="addVisible" width="30%">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="菜单名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="url">
                    <el-input v-model="form.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单">
                    <el-input v-model="form.pid"></el-input>
                </el-form-item>
                <el-form-item label="序号">
                    <el-input v-model="form.order"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="add">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 删除提示框 -->
        <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
            <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="del">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'menu',
        data() {
            // 使用return使为局部变量，非全局可见
            return {
                // 后台接口url
                addUrl: '',
                delUrl: '',
                delBatchUrl: '',
                updateUrl: '',
                listUrl: './static/menu.json',

                // 可视标识变量
                editVisible: false,
                delVisible: false,
                addVisible : false,

                tableData: [],  // 表格数据
                cur_page: 1,    // 当前页
                idx: -1,        // 选中的行
                multipleSelection: [],  // 多选记录
                del_list: [],   // 要删除的ids

                // 表单对象
                form: {
                    id: '',
                    name: '',
                    url: '',
                    order: '',
                    pid: ''
                }
            }
        },
        // 创建vue实例时触发
        created() {
            alert("created()");
            this.getData();
        },
        // 加载完成即触发，执行赋值等操作
        computed: {
            data() {
                alert("computed()");
                return this.tableData;
            }
        },
        // 方法，条件触发，如点击事件
        methods: {
            // 分页导航
            handleCurrentChange(val) {
                this.cur_page = val;
                this.getData();
            },
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            toAdd() {
                this.addVisible = true;
            },
            toDel(index, row) {
                this.delVisible = true;
                this.idx = index;
                this.row = row;
            },
            toEdit(index, row) {
                this.idx = index;
                const item = this.tableData[index];
                this.form = {
                    id: item.id,
                    name: item.name,
                    url: item.url,
                    pid: item.pid,
                    order: item.order
                }
                this.editVisible = true;
            },
            // 新增
            add() {
                this.$axios.post(this.addUrl, this.form).then((res) => {
                    alert(res);
                    this.$message.success(`新增菜单成功`);
                    this.addVisible = false;
                }).catch(function (err) {
                    console.log(err);
                    this.$message.success(`新增菜单失败`);
                });
            },
            // 删除单条记录
            del(){
                this.$axios.post(this.delUrl, {
                    id: this.tableData[this.idx].id
                }).then(function (res) {
                    alert(res);
                    this.tableData.splice(this.idx, 1);
                    this.$message.success('删除成功');
                    this.delVisible = false;
                }).catch(function (err) {
                    console.log(err);
                });
            },
            // 批量删除
            delBatch() {
                const length = this.multipleSelection.length;
                let ids = '';
                this.del_list = this.del_list.concat(this.multipleSelection);
                for (let i = 0; i < length; i++) {
                    ids += this.multipleSelection[i].name + ',';
                }
                this.$message.error('删除了' + ids);
                this.multipleSelection = [];

                this.$axios.post(this.delUrl, {
                    ids: ids
                }).then(function (res) {
                    alert(res);
                }).catch(function (err) {
                    console.log(err);
                });
            },
            // 修改
            edit() {
                this.$axios.post(this.updateUrl, this.form).then((res) => {
                    alert(res);
                    this.$set(this.tableData, this.idx, this.form);
                    this.editVisible = false;
                    this.$message.success(`修改第 ${this.idx+1} 行成功`);
                }).catch(function (err) {
                    console.log(err);
                    this.$message.success(`修改菜单失败`);
                });
            },
            getData() {
                this.$axios.get(this.listUrl, {
                    page: this.cur_page
                }).then((res) => {
                    this.tableData = res.data.list;
                }).catch(function (err) {
                    console.log(err);
                });
            }

        }
    }

</script>

<style scoped>
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
    .del-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
</style>
