<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i>角色管理</el-breadcrumb-item>
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
                <el-table-column prop="name" label="角色名称"  width="150">
                </el-table-column>
                <el-table-column prop="description" label="角色描述" width="120">
                </el-table-column>
                <el-table-column label="操作" width="280">
                    <template slot-scope="scope">
                        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button size="small" @click="handleConfig(scope.$index, scope.row)">配置</el-button>
                        <el-button size="small" type="danger" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <div class="pagination">
                <el-pagination @current-change="handlePageChange" layout="prev, pager, next" :total="total">
                </el-pagination>
            </div>
        </div>

        <!-- 新增弹出框 -->
        <el-dialog title="新增" :visible.sync="addVisible" width="30%">
            <el-form ref="form" :model="addForm" label-width="50px">
                <el-form-item label="角色名称">
                    <el-input v-model="addForm.name"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="addForm.description"></el-input>
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
                <el-form-item label="角色名称">
                    <el-input v-model="editForm.name"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="editForm.description"></el-input>
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

        <!-- 角色菜单权限配置弹出框 -->
        <el-dialog title="角色菜单权限配置" :visible.sync="configVisible" width="30%">
            <el-form :model="configForm">
                <el-form-item>
                    <el-table :data="configData" border height="300">
                        <el-table-column property="menuName" label="菜单名称" ></el-table-column>
                        <el-table-column property="menuState" label="默认关闭">
                            <template slot-scope="scope">
                                <el-switch
                                    on-text ="是"
                                    off-text = "否"
                                    on-color="#5B7BFA"
                                    off-color="#dadde5"
                                    v-model="scope.row.menuState"
                                    @change=change(scope.$index,scope.row)
                                >
                                </el-switch>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-form-item>
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
                /*选择的数量*/
                selectedCount:null,
                /*选中记录数组*/
                selectedRows:[],
                /*加载中*/
                loading: true,
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

                // 增删改窗口可视标志
                addVisible: false,
                editVisible: false,
                delVisible: false,
                configVisible: false,

                /*新增表单实体*/
                addForm:{
                    name:null,
                    description:null
                },
                /*修改表单实体*/
                editForm:{
                    id:null,
                    name:null,
                    description:null
                },
                /*角色菜单配置实体*/
                configForm:{

                },
                /*角色菜单配置*/
                configData:[],
                /*角色对应的菜单集合*/
                roleMenuIds:[]
            }
        },
        // 页面加载后初始化
        mounted() {
            // 加载表格数据
            this.getTable({
                "pageInfo":this.pageInfo,
                "key_word":null
            });
        },
        // 声明的方法
        methods:{
            /*得到表数据*/
            getTable(e) {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.role_getRolesByPage_url,
                    params: {
                        'pageNo':e.pageInfo.page,
                        'pageSize':e.pageInfo.pageSize,
                        'name':e.key_word
                    }
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.tableData=response.data.data.list;
                        this.total=response.data.data.total;
                    } else {
                        this.$message.info('加载角色列表失败');
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
                //this.$message.error('删除了' + ids.join(','));

                if(ids != null && ids.length > 0){
                    this.axios({
                        method: 'get',
                        url: this.GLOBAL.role_delByBatch_url + "/" + ids.join(',')
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
                    desc: null
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
                    desc: item.desc
                }
                this.editVisible = true;
            },
            /*点击配置*/
            handleConfig(index, row) {
                this.currentIndex = index;
                const item = this.tableData[index];
                // 加载角色对应的菜单
                this.loadConfigedMenus(item.id);
                this.configVisible = true;
            },
            /*点击删除*/
            handleDel(index, row) {
                this.currentIndex = index;
                this.delVisible = true;
            },
            /*分页导航*/
            handlePageChange(val) {
                this.pageInfo.pageNo = val-1;
                this.getTable({
                    "pageInfo":this.pageInfo,
                    'key_word':this.key_word
                });
            },
            /*新增*/
            addOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.role_add_url,
                    data: this.addForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.addVisible = false;
                        this.getTable({
                            "pageInfo":this.pageInfo,
                            'name':this.key_word
                        });
                        this.$message.info('创建成功');
                    } else {
                        this.$message.info('创建失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*修改*/
            editOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.role_update_url,
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
                    url: this.GLOBAL.role_del_url + "/" + this.tableData[this.currentIndex].id
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.$set(this.tableData, this.currentIndex, this.editForm);
                        this.delVisible = false;
                        this.$message.info('删除成功');
                    } else {
                        this.$message.info('删除失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*配置角色菜单*/
            change:function(index,row){
                console.log(index,row);
                if(row.menuState == true) {
                    this.roleMenuIds.push(row.menuId);
                } else {
                    this.roleMenuIds.splice(row.menuId);
                }
            },
            /*完成配置*/
            configOk() {
                alert(JSON.stringify(this.roleMenuIds));
                this.configVisible = false;

                //
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.role_setRoleMenus_url,
                    param: {
                        "roleId": this.tableData[this.currentIndex].id,
                        "menuIds": this.roleMenuIds
                    }
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.$message.info('配置角色菜单权限成功');
                    } else {
                        this.$message.info('配置角色菜单权限失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });

                this.roleMenuIds = [];
            },
            /*加载二级菜单集合*/
            loadConfigedMenus(roleId) {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.menu_getMenusByRole_url + "/" + roleId
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        alert(JSON.stringify(response));
                        this.configData = response.data.data;
                    } else {
                        this.$message.info('加载角色列表失败');
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
