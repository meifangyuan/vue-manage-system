<template>
    <div class="table">
    <div class="custom-tree-container">
        <div class="block">
            <el-tree
                :data="treeData"
                show-checkbox
                node-key="id"
                default-expand-all
                :expand-on-click-node="false">
                <span class="custom-tree-node" slot-scope="{ node, data }">
                    <span>{{ node.label }}</span>
                    <span>
                      <el-button type="text" size="mini" @click="() => handleAdd(node, data)">新增</el-button>
                        <el-button type="text" size="mini" @click="() => handleEdit(node, data)">编辑</el-button>
                      <el-button type="text" size="mini" @click="() => handleDel(node, data)">删除</el-button>
                    </span>
                 </span>
            </el-tree>
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
    let id = 1000;

    export default {
        data() {
            return {
                treeData : [{
                    id: 1,
                    label: '系统管理',
                    children: [
                        {
                            id: 2,
                            label: '菜单管理'
                        },
                        {
                            id: 3,
                            label: '角色管理'
                        },
                        {
                            id: 4,
                            label: '用户管理'
                        }]
                }],

                addVisible : false,
                editVisible : false,
                delVisible : false,

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
                }
            }
        },

        methods: {
            handleAdd(node, data) {
                // const newChild = { id: id++, label: 'test', children: [] };
                // if (!data.children) {
                //     this.$set(data, 'children', []);
                // }
                // data.children.push(newChild);

                this.addVisible = true;
                this.addForm.pid = {
                    name:null,
                    url:null,
                    pid: node.id,
                    sort:null,
                    icon:null
                }
            },

            addOk() {
                // this.axios({
                //     method: 'post',
                //     url: this.GLOBAL.menu_add_url,
                //     data: this.addForm
                // }).then(function (response) {
                //     if(response.data.errCode == '0000') {
                //         this.addVisible = false;
                //         this.loadTable({
                //             "pageInfo":this.pageInfo,
                //             'key_word':this.key_word
                //         });
                //         this.$message.info('新增成功');
                //     } else {
                //         this.$message.info('新增失败');
                //     }
                // }.bind(this)).catch(function (error) {
                //     alert(error);
                // });
            },

            handleEdit(node, data) {
                const parent = node.parent;
                const children = parent.data.children || parent.data;
                const index = children.findIndex(d => d.id === data.id);
                children.splice(index, 1);
            },

            editOk() {

            },

            handleDel(node, data) {
                alert(node.id);
                const parent = node.parent;
                const children = parent.data.children || parent.data;
                const index = children.findIndex(d => d.id === data.id);
                children.splice(index, 1);
            },

            delOk() {

            }
        }
    };

</script>

<style scoped>
    .custom-tree-node {
        flex: 1;
        display: flex;
        align-items: center;
        justify-content: space-between;
        font-size: 14px;
        padding-right: 8px;
    }
</style>
