<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i>菜单管理</el-breadcrumb-item>
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
                <el-table-column prop="name" label="菜单名称"  width="150">
                </el-table-column>
                <el-table-column prop="url" label="菜单地址" width="120">
                </el-table-column>
                <el-table-column prop="pid" label="父菜单ID" width="120">
                </el-table-column>
                <el-table-column prop="icon" label="图标" width="120">
                </el-table-column>
                <el-table-column prop="sort" label="排序" sortable width="120">
                </el-table-column>
                <el-table-column label="操作" width="180">
                    <template slot-scope="scope">
                        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
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
                <el-form-item label="菜单名称">
                    <el-input v-model="addForm.name"></el-input>
                </el-form-item>
                <el-form-item label="菜单地址">
                    <el-input v-model="addForm.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单">
                    <el-select v-model="addForm.pid" clearable placeholder="请选择">
                        <el-option v-for="item in rootMenus" :label="item.name" :value="item.id" :key="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="addForm.icon"></el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input v-model="addForm.sort"></el-input>
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
                <el-form-item label="菜单名称">
                    <el-input v-model="editForm.name"></el-input>
                </el-form-item>
                <el-form-item label="菜单地址">
                    <el-input v-model="editForm.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单">
                    <el-select v-model="editForm.pid" clearable placeholder="请选择">
                        <el-option v-for="item in rootMenus" :label="item.name" :value="item.id" :key="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="editForm.icon"></el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input v-model="editForm.sort"></el-input>
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
    </div>
</template>

<script>
    export default {
        // 声明并初始化变量
        data() {
            return {
                /*搜索关键词*/
                key_word:null,
                /*选中行数*/
                selectedCount:0,
                /*选中记录集合*/
                selectedRows:[],
                /*加载中*/
                loading:true,
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

                /*新增表单实体*/
                addForm:{
                    name:null,
                    url:null,
                    pid:0,
                    sort:null,
                    icon:null
                },
                /*修改表单实体*/
                editForm:{
                    id:null,
                    name:null,
                    url:null,
                    pid:null,
                    sort:null,
                    icon:null
                },
                /*根菜单集合*/
                rootMenus:[]
            }
        },
        // 页面加载后初始化
        mounted() {
            // 加载表格数据
            this.loadTable({
                "pageInfo":this.pageInfo,
                "key_word":null
            });
            // 加载rootMenus
            this.loadRootMenus();
        },
        // 声明的方法
        methods:{
            /*加载表数据*/
            loadTable(e) {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.menu_getMenusByPage_url,
                    data: {
                        'pageNo':e.pageInfo.pageNo,
                        'pageSize':e.pageInfo.pageSize,
                        'name':e.key_word
                    }
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.tableData=response.data.data.list;
                        this.total=response.data.data.total;
                    } else {
                        this.$message.info('加载数据列表失败');
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
                this.loadTable({
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
                        url: this.GLOBAL.menu_delByBatch_url + "/" + ids.join(',')
                    }).then(function (response) {
                        if(response.data.errCode == '0000') {
                            alert(JSON.stringify(response));
                            // 重新加载表数据
                            this.loadTable({
                                "pageInfo":this.pageInfo,
                                'key_word':this.key_word
                            });
                            // 重置选中记录集合
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
                    url: null,
                    pid: null,
                    sort: null,
                    icon: null
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
                    url: item.url,
                    pid: item.pid,
                    sort: item.sort,
                    icon: item.icon
                }
                this.editVisible = true;
            },
            /*点击删除*/
            handleDel(index, row) {
                this.currentIndex = index;
                this.delVisible = true;
            },
            /*分页导航*/
            handlePageChange(val) {
                this.pageInfo.pageNo = val;
                this.loadTable({
                    "pageInfo":this.pageInfo,
                    'key_word':this.key_word
                });
            },
            /*新增*/
            addOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.menu_add_url,
                    data: this.addForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.addVisible = false;
                        this.loadTable({
                            "pageInfo":this.pageInfo,
                            'key_word':this.key_word
                        });
                        this.$message.info('新增成功');
                    } else {
                        this.$message.info('新增失败');
                    }
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*修改*/
            editOk() {
                this.axios({
                    method: 'post',
                    url: this.GLOBAL.menu_update_url,
                    data: this.editForm
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        // 直接更新当前行记录
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
                    url: this.GLOBAL.menu_delByBatch_url + "/" + this.tableData[this.currentIndex].id
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
            // 加载根菜单集合
            loadRootMenus() {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.menu_getRootMenus_url
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.rootMenus=response.data.data;
                    } else {
                        this.$message.info('加载根菜单列表失败');
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
