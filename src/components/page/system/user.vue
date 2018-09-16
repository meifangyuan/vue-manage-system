<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i>用户管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button type="primary" icon="add" class="handle-add mr10" @click="handleAdd()">新增</el-button>
                <el-button type="primary" icon="delete" class="handle-del mr10" @click="batchDel">批量删除</el-button>
                <el-input v-model="key_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>
            </div>

            <el-table :data="tableData" border style="width: 100%" ref="multipleTable" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column prop="name" label="姓名" width="180">
                </el-table-column>
                <el-table-column prop="loginName" label="登录名"  width="180">
                </el-table-column>
                <el-table-column prop="email" label="邮箱" width="180">
                </el-table-column>
                <el-table-column label="操作" width="240">
                    <template slot-scope="scope">
                        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button size="small" @click="handleConfig(scope.$index, scope.row)">配置</el-button>
                        <el-button size="small" type="danger" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <div class="pagination">
                <el-pagination @current-change="handlePageChange" layout="prev, pager, next, total" :total="total">
                </el-pagination>
            </div>
        </div>

        <!-- 新增弹出框 -->
        <el-dialog title="新增" :visible.sync="addVisible" width="30%">
            <el-form ref="form" :model="addForm" label-width="50px">
                <el-form-item label="姓名">
                    <el-input v-model="addForm.name"></el-input>
                </el-form-item>
                <el-form-item label="登录名">
                    <el-input v-model="addForm.loginName"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="addOk">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="editForm" label-width="50px">
                <el-form-item label="姓名">
                    <el-input v-model="editForm.name"></el-input>
                </el-form-item>
                <el-form-item label="登录名">
                    <el-input v-model="editForm.loginName"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="editForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="editOk">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 删除提示框 -->
        <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
            <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="delOk">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 用户角色配置弹出框 -->
        <el-dialog title="用户角色配置" :visible.sync="configVisible" width="30%">
            <el-form :model="configForm">
                <el-select v-model="configForm.roleId" clearable placeholder="请选择">
                    <el-option v-for="item in roles" :label="item.name" :value="item.id" :key="item.id">
                    </el-option>
                </el-select>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="configVisible = false">取 消</el-button>
                <el-button type="primary" @click="configOk">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
    export default {
        // 声明并初始化变量
        data() {
            return {
                /*搜索关键词*/
                key_word:null,
                /*选中记录数组*/
                selectedRows:[],
                /*表格数据*/
                tableData:[],
                /*分页total属性*/
                total:0,
                /*分页实体*/
                pageInfo:{
                    pageNo:1,
                    pageSize:10
                },
                /*当前行index*/
                currentIndex:null,

                /*可视标志*/
                addVisible: false,
                editVisible: false,
                delVisible: false,
                configVisible:false,

                /*新增表单实体*/
                addForm:{
                    name:null,
                    loginName:null,
                    password:null,
                    email:null
                },
                /*修改表单实体*/
                editForm:{
                    id:null,
                    name:null,
                    loginName:null,
                    password:null,
                    email:null
                },
                /*用户角色配置实体*/
                configForm:{
                    userId:null,
                    roleId:null
                },
                /*角色集合*/
                roles:[]
            }
        },
        // 页面加载后初始化
        mounted() {
            // 加载表格数据
            this.getTable({
                "pageInfo":this.pageInfo,
                "key_word":null
            });
            // 加载角色集合
            this.loadRoles();
        },
        // 声明的方法
        methods:{
            /*得到表数据*/
            getTable(e) {
                // alert(e.pageInfo.pageNo);
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.user_getUsersByPage_url,
                    params: {
                        'pageNo':e.pageInfo.pageNo,
                        'pageSize':e.pageInfo.pageSize,
                        'name':e.key_word
                    }
                }).then(function (response) {
                    // alert(JSON.stringify(response));
                    if(response.data.errCode == '0000') {
                        this.tableData=response.data.data.list;
                        this.total=response.data.data.total;
                        // alert(this.total);
                    } else {
                        this.$message.info('加载用户列表失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*初始化分页信息*/
            initPageInfo(){
                this.pageInfo.pageNo = 1,
                this.pageInfo.pageSize = 10
            },
            /*搜索按钮点击事件*/
            search() {
                this.initPageInfo();
                this.getTable({
                    "pageInfo":this.pageInfo,
                    'key_word':this.key_word
                });
            },
            /*批量删除*/
            batchDel() {
                const length = this.selectedRows.length;
                const ids = [];
                for (let i = 0; i < length; i++) {
                    ids.push(this.selectedRows[i].id);
                }

                if(ids != null && ids.length > 0){
                    this.axios({
                        method: 'get',
                        url: this.GLOBAL.user_delByBatch_url + "/" + ids.join(',')
                    }).then(function (response) {
                        if(response.data.errCode == '0000') {
                            this.getTable({
                                "pageInfo":this.pageInfo,
                                'key_word':this.key_word
                            });
                            this.selectedRows = [];
                            this.$message.info('删除成功');
                        } else {
                            this.$message.info('删除失败');
                        }
                    }.bind(this)).catch(function (error) {
                        alert(error);
                    });
                }
            },
            /*多选改变*/
            handleSelectionChange(val) {
                this.selectedRows = val;
            },
            /*点击新增*/
            handleAdd() {
                this.addForm = {
                    name: null,
                    loginName: null,
                    password: null,
                    email: null
                }
                this.addVisible = true;
            },
            /*点击修改*/
            handleEdit(index, row) {
                this.currentIndex = index;
                const item = this.tableData[index];
                this.editForm = {
                    id: item.id,
                    name: item.name,
                    loginName: item.loginName,
                    password: item.password,
                    email: item.email
                }
                this.editVisible = true;
            },
            /*点击配置*/
            handleConfig(index, row) {
                this.currentIndex = index;
                const item = this.tableData[index];
                this.configForm.userId = item.id;
                this.configVisible = true;

                // 加载用户角色信息
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.user_getUserRole_url + "/" + item.id
                }).then(function (response) {
                    // alert(JSON.stringify(response));
                    if(response.data.errCode == '0000') {
                        this.configForm.roleId = response.data.data.id;
                    } else {
                        this.$message.info('获取用户角色失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });

            },
            /*点击删除*/
            handleDel(index, row) {
                this.currentIndex = index;
                this.delVisible = true;
            },
            /*分页导航*/
            handlePageChange(val) {
                this.pageInfo.pageNo = val;
                this.getTable({
                    "pageInfo":this.pageInfo,
                    'key_word':this.key_word
                });
            },
            /*新增*/
            addOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.user_add_url,
                    data: this.addForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.addVisible = false;
                        this.getTable({
                            "pageInfo":this.pageInfo,
                            'key_word':this.key_word
                        });
                        this.$message.info('新建成功');
                    } else {
                        this.$message.info('新建失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*修改*/
            editOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.user_update_url,
                    data: this.editForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.$set(this.tableData, this.currentIndex, this.editForm);
                        this.editVisible = false;
                        this.$message.info('修改成功');
                    } else {
                        this.$message.info('修改失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*删除*/
            delOk() {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.user_del_url + "/" + this.tableData[this.currentIndex].id
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.tableData.splice(this.currentIndex, 1);
                        this.delVisible = false;
                        this.$message.info('删除成功');
                    } else {
                        this.$message.info('删除失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*完成配置*/
            configOk() {
                this.configVisible = false;

                // 配置用户角色
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.user_setUserRole_url,
                    data: this.configForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.$message.info('配置成功');
                    } else {
                        this.$message.info('配置失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });

                // 重置配置Form
                this.configForm = {
                    userId:null,
                    roleId:null
                }

            },
            /*加载角色集合*/
            loadRoles() {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.role_getAllRoles_url
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.roles = response.data.data;
                    } else {
                        this.$message.info('加载角色集合失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
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
