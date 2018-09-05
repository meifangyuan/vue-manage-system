<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i>菜单管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>

        <div class="container">
            <div class="handle-box">
                菜单名称：
                <el-input v-model="select_word" placeholder="关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>
            </div>

            <div class="handle-box">
                <el-button type="primary" icon="add" class="handle-del mr10" @click="openNewModal()">新增</el-button>
                <el-button type="primary" icon="update" class="handle-del mr10" @click="openModifyModal()">修改</el-button>
                <el-button type="primary" icon="delete" class="handle-del mr10" @click="del()">删除</el-button>
            </div>

            <el-table :data="data1" border style="width: 100%" ref="multipleTable" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column prop="name" label="菜单名称" width="120"></el-table-column>
                <el-table-column prop="url" label="菜单地址" ></el-table-column>
                <el-table-column prop="parentId" label="父菜单id" ></el-table-column>
                <el-table-column prop="sort" label="排序" ></el-table-column>
                <el-table-column prop="icon" label="排序" ></el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next" :total="1000">
                </el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="菜单名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="菜单地址">
                    <el-input v-model="form.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单id">
                    <el-input v-model="form.parentId"></el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input v-model="form.sort"></el-input>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="form.icon"></el-input>
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
                <el-form-item label="菜单地址">
                    <el-input v-model="form.url"></el-input>
                </el-form-item>
                <el-form-item label="父菜单id">
                    <el-input v-model="form.parentId"></el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input v-model="form.sort"></el-input>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="form.icon"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="add">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 删除提示框 -->
        <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
            <div class="del-dialog-cnt">是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="del">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        data () {
            return {
                /*用于查找的菜单id*/
                menuId:null,
                /*选择的数量*/
                count:null,
                /*选中的组数据*/
                groupId:null,
                /*新建modal的显示参数*/
                newModal:false,
                /*修改modal的显示参数*/
                modifyModal:false,
                /*分页total属性绑定值*/
                total:0,
                /*loading*/
                loading: true,
                /*pageInfo实体*/
                pageInfo:{
                    page:0,
                    pageSize:10
                },
                /*menu实体*/
                menu:{
                    id:null,
                    name:null,
                    url:null,
                    parentId:null,
                    sort:null,
                    remark:null,
                    icon:null
                },
                /*用于添加的menu实体*/
                menuNew:{
                    id:null,
                    name:null,
                    url:null,
                    parentId:null,
                    sort:null,
                    remark:null,
                    icon:null
                },
                /*用于修改的menu实体*/
                menuModify:{
                    id:null,
                    name:null,
                    url:null,
                    parentId:null,
                    sort:null,
                    remark:null,
                    icon:null
                },
                /*新建验证*/
                ruleNew:{
                    name: [
                        { type:'string',required: true, message: '输入菜单名', trigger: 'blur' }
                    ],
                    url: [
                        { type:'string',required: true, message: '输入路径', trigger: 'blur' }
                    ],
                    parentId: [
                        { required: true, message: '输入父类ID', trigger: 'blur' },
                        {validator(rule, value, callback) {
                                if (!Number.isInteger(+value)) {
                                    callback(new Error('请输入数字'));
                                } else {
                                    callback();
                                }

                            }, trigger: 'blur'}
                    ],
                    sort: [
                        { required: true, message: '输入排序', trigger: 'blur' },
                        {validator(rule, value, callback) {
                                if (!Number.isInteger(+value)) {
                                    callback(new Error('请输入数字'));
                                } else {
                                    callback();
                                }

                            }, trigger: 'blur'}
                    ],
                    icon: [
                        { type:'string',required: true, message: '输入图标', trigger: 'blur' }
                    ]
                },
                /*修改验证*/
                ruleModify:{
                    name: [
                        { type:'string',required: true, message: '输入菜单名', trigger: 'blur' }
                    ],
                    url: [
                        { type:'string',required: true, message: '输入路径', trigger: 'blur' }
                    ],
                    parentId: [
                        { required: true, message: '输入父类ID', trigger: 'blur' },
                        {validator(rule, value, callback) {
                                if (!Number.isInteger(+value)) {
                                    callback(new Error('请输入数字'));
                                } else {
                                    callback();
                                }

                            }, trigger: 'blur'}
                    ],
                    sort: [
                        { required: true, message: '输入排序', trigger: 'blur' },
                        {validator(rule, value, callback) {
                                if (!Number.isInteger(+value)) {
                                    callback(new Error('请输入数字'));
                                } else {
                                    callback();
                                }

                            }, trigger: 'blur'}
                    ],
                    icon: [
                        { type:'string',required: true, message: '输入图标', trigger: 'blur' }
                    ]
                },
                /*菜单列表*/
                menuList:[],
                /*生产类型表显示字段*/
                columns1: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '菜单ID',
                        key: 'id'
                    },
                    {
                        title: '菜单名称',
                        key: 'name'
                    },
                    {
                        title: '地址',
                        key: 'url'
                    },
                    {
                        title: '上级菜单id',
                        key: 'parentId'
                    },
                    {
                        title: '排序',
                        key: 'sort'
                    },
                    {
                        title: '图标',
                        key: 'icon'
                    }
                ],
                /*生产类型表数据*/
                data1: []
            }
        },
        mounted(){
            /*页面初始化调用方法*/
            this.getTable({
                "pageInfo":this.pageInfo,
                'menuId':this.menuId
            });
            this.axios({
                method: 'get',
                url: '/menus/parentId',
                params: {
                    'parentId': 0
                }
            }).then(function (response) {
                var listTemp = response.data;
                for (var i = 0; i < listTemp.length; i++) {
                    this.menuList.push({
                        "value": listTemp[i].id,
                        "label": listTemp[i].name
                    });
                }
            }.bind(this)).catch(function (error) {
                alert(error);
            });
        },
        methods:{
            /*pageInfo实体初始化*/
            initPageInfo(){
                this.pageInfo.page = 0;
                this.pageInfo.pageSize = 10;
            },
            /*menu实体初始化*/
            initMenu(){
                this.menu.id = null;
                this.menu.name = null;
                this.menu.url = null;
                this.menu.parentId = null;
                this.menu.sort = null;
                this.menu.remark = null;
                this.menu.icon = null;
            },
            /*menuNew实体初始化*/
            initMenuNew(){
                this.menuNew.id = null;
                this.menuNew.name = null;
                this.menuNew.url = null;
                this.menuNew.parentId = null;
                this.menuNew.sort = null;
                this.menuNew.remark = null;
                this.menuNew.icon = null;
            },
            /*menuModify实体初始化*/
            initMenuModify(){
                this.menuModify.id = null;
                this.menuModify.name = null;
                this.menuModify.url = null;
                this.menuModify.parentId = null;
                this.menuModify.sort = null;
                this.menuModify.remark = null;
                this.menuModify.icon = null;
            },
            /*menuNew设置*/
            menuSet(e){
                this.menu.id = e.id;
                this.menu.name = e.name;
                this.menu.url = e.url;
                this.menu.parentId = e.parentId;
                this.menu.sort = e.sort;
                this.menu.remark = e.remark;
                this.menu.icon = e.icon;
            },
            /*menuNew设置*/
            menuNewSet(e){
                this.menuNew.id = e.id;
                this.menuNew.name = e.name;
                this.menuNew.url = e.url;
                this.menuNew.parentId = e.parentId;
                this.menuNew.sort = e.sort;
                this.menuNew.remark = e.remark;
                this.menuNew.icon = e.icon;
            },
            /*menuModify设置*/
            menuModifySet(e){
                this.menuModify.id = e.id;
                this.menuModify.name = e.name;
                this.menuModify.url = e.url;
                this.menuModify.parentId = e.parentId+'';
                this.menuModify.sort = e.sort+'';
                this.menuModify.remark = e.remark;
                this.menuModify.icon = e.icon;
            },
            /*得到表数据*/
            getTable(e) {
                this.axios({
                    method: 'get',
                    url: '/menus',
                    params: {
                        'page':e.pageInfo.page,
                        'pageSize':e.pageInfo.pageSize,
                        'menuId':e.menuId
                    }
                }).then(function (response) {
                    this.data1=response.data.data;
                    this.total=response.data.totalCount;
                }.bind(this)).catch(function (error) {
                    alert(error);
                });
            },
            /*搜索按钮点击事件*/
            search(){
                this.initPageInfo();
                this.getTable({
                    "pageInfo":this.pageInfo,
                    'menuId':this.menuId
                });
            },
            /*分页点击事件*/
            pageSearch(e){
                this.pageInfo.page = e-1;
                this.getTable({
                    "pageInfo":this.pageInfo,
                    'menuId':this.menuId
                });
            },
            /*新建点击触发事件*/
            openNewModal(){
                this.newModal = true;
                this.initMenuNew();
                this.data1.sort();
                this.count = 0;
                this.groupId = null;
            },
            /*新建modal的newOk点击事件*/
            newOk (menuNew) {
                this.$refs[menuNew].validate((valid) => {
                    if (valid) {
                        this.initMenu();
                        this.menuSet(this.menuNew);
                        this.axios({
                            method: 'post',
                            url: '/menus/menu',
                            data: this.menu
                        }).then(function (response) {
                            this.initMenuNew();
                            this.getTable({
                                "pageInfo":this.pageInfo,
                                'menuId':this.menuId
                            });
                            this.$Message.info('新建成功');
                        }.bind(this)).catch(function (error) {
                            alert(error);
                        });
                        this.newModal = false;
                    }else {
                        setTimeout(() => {
                            this.loading = false;
                            this.$nextTick(() => {
                                this.loading = true;
                            });
                        }, 1000);
                    }
                })
            },
            /*点击修改时判断是否只选择一个*/
            openModifyModal(){
                if(this.count > 1 || this.count < 1){
                    this.modifyModal= false;
                    this.$Message.warning('请至少选择一项(且只能选择一项)');
                }else{
                    this.modifyModal = true;
                }
            },
            /*修改modal的modifyOk点击事件*/
            modifyOk (menuModify) {
                this.$refs[menuModify].validate((valid) => {
                    if (valid) {
                        this.initMenu();
                        this.menuSet(this.menuModify);
                        this.axios({
                            method: 'put',
                            url: '/menus/'+this.menu.id,
                            data: this.menu
                        }).then(function (response) {
                            this.initMenuNew();
                            this.getTable({
                                "pageInfo":this.pageInfo,
                                'menuId':this.menuId
                            });
                            this.$Message.info('修改成功');
                        }.bind(this)).catch(function (error) {
                            alert(error);
                        });
                        this.modifyModal = false;
                    }else {
                        this.$Message.error('表单验证失败!');
                        setTimeout(() => {
                            this.loading = false;
                            this.$nextTick(() => {
                                this.loading = true;
                            });
                        }, 1000);
                    }
                })
            },
            /*modal的cancel点击事件*/
            cancel () {
                this.$Message.info('点击了取消');
            },
            /*table选择后触发事件*/
            change(e){
                if(e.length==1){
                    this.menuModifySet(e['0']);
                }
                this.setGroupId(e);
            },
            /*通过选中的行设置groupId的值*/
            setGroupId(e){
                this.groupId=[];
                this.count=e.length;
                for (var i = 0; i <= e.length - 1; i++) {
                    this.groupId.push(e[i].id);
                }
            },
            /*删除table中选中的数据*/
            del(){
                if(this.groupId!=null && this.groupId!=""){
                    this.axios({
                        method: 'delete',
                        url: '/menus',
                        data: this.groupId
                    }).then(function (response) {
                        this.getTable({
                            "pageInfo":this.pageInfo,
                            'menuId':this.menuId
                        });
                        this.groupId=null;
                        this.count=0;
                        this.$Message.info('删除成功');
                    }.bind(this)).catch(function (error) {
                        alert(error);
                    });
                }
            },
            /*表格中双击事件*/
            dblclick(e){
                this.menuModifySet(e);
                this.modifyModal = true;
                this.data1.sort();
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
