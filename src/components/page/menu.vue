<template>
    <div style="margin: 20px;">
        <div>
            <Row style="margin-bottom: 25px;">
                <Col span="8">菜单名称：
                <Select v-model="menuId" filterable clearable style="width: 200px">
                    <Option v-for="item in menuList" :value="item.value" :key="item.value">{{ item.label }}</Option>
                </Select>
                </Col>
                <Col span="8"><Button type="primary" shape="circle" icon="ios-search" @click="search()">搜索</Button></Col>
            </Row>
        </div>
        <div>
            <ul>
                <li>
                    <Button type="primary" icon="plus-round" @click="openNewModal()">新建</Button>
                    <Button type="success" icon="wrench" @click="openModifyModal()">修改</Button>
                    <Button type="error" icon="trash-a" @click="del()">删除</Button>
                </li>
                <li>
                    <div style="padding: 10px 0;">
                        <Table border :columns="columns1" :data="data1" :height="400" @on-selection-change="s=>{change(s)}" @on-row-dblclick="s=>{dblclick(s)}"></Table>
                    </div>
                </li>
                <li>
                    <div style="text-align: right;">
                        <Page :total="total" :page-size="pageInfo.pageSize" show-elevator show-total @on-change="e=>{pageSearch(e)}"></Page>
                    </div>
                </li>
            </ul>
        </div>
        <!--添加modal-->
        <Modal :mask-closable="false" :visible.sync="newModal" :loading = "loading" v-model="newModal" width="600" title="新建" @on-ok="newOk('menuNew')" @on-cancel="cancel()">
            <Form ref="menuNew" :model="menuNew" :rules="ruleNew" :label-width="80" >
                <Row>
                    <Col span="12">
                    <Form-item label="菜单名称:" prop="name">
                        <Input v-model="menuNew.name" style="width: 204px"/>
                    </Form-item>
                    </Col>
                    <Col span="12">
                    <Form-item label="菜单地址:" prop="url">
                        <Input v-model="menuNew.url" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                    <Form-item label="父菜单ID:" prop="pid">
                        <Input v-model="menuNew.pid" style="width: 204px"/>
                    </Form-item>
                    </Col>
                    <Col span="12">
                    <Form-item label="排序号:" prop="sort">
                        <Input v-model="menuNew.sort" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                    <Form-item label="图标:" prop="icon">
                        <Input v-model="menuNew.icon" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
            </Form>
        </Modal>
        <!--修改modal-->
        <Modal :mask-closable="false" :visible.sync="modifyModal" :loading = "loading" v-model="modifyModal" width="600" title="修改" @on-ok="modifyOk('menuModify')" @on-cancel="cancel()">
            <Form ref="menuModify" :model="menuModify" :rules="ruleModify" :label-width="80" >
                <Row>
                    <Col span="12">
                    <Form-item label="菜单名称:" prop="name">
                        <Input v-model="menuModify.name" style="width: 204px"/>
                    </Form-item>
                    </Col>
                    <Col span="12">
                    <Form-item label="菜单地址:" prop="url">
                        <Input v-model="menuModify.url" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                    <Form-item label="父菜单ID:" prop="pid">
                        <Input v-model="menuModify.pid" style="width: 204px"/>
                    </Form-item>
                    </Col>
                    <Col span="12">
                    <Form-item label="排序号:" prop="sort">
                        <Input v-model="menuModify.sort" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                    <Form-item label="图标:" prop="icon">
                        <Input v-model="menuModify.icon" style="width: 204px"/>
                    </Form-item>
                    </Col>
                </Row>
            </Form>
        </Modal>
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
                    page:1,
                    pageSize:10
                },
                /*menu实体*/
                menu:{
                    id:null,
                    name:null,
                    url:null,
                    pid:null,
                    sort:null,
                    remark:null,
                    icon:null
                },
                /*用于添加的menu实体*/
                menuNew:{
                    id:null,
                    name:null,
                    url:null,
                    pid:null,
                    sort:null,
                    icon:null
                },
                /*用于修改的menu实体*/
                menuModify:{
                    id:null,
                    name:null,
                    url:null,
                    pid:null,
                    sort:null,
                    icon:null
                },
                /*新建验证*/
                ruleNew:{
                    name: [
                        { type:'string',required: true, message: '输入菜单名', trigger: 'blur' }
                    ],
                    url: [
                        { type:'string',required: true, message: '输入菜单地址', trigger: 'blur' }
                    ],
                    pid: [
                        { required: true, message: '输入菜单ID', trigger: 'blur' },
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
                        { type:'string',required: true, message: '输入菜单地址', trigger: 'blur' }
                    ],
                    pid: [
                        { required: true, message: '输入父菜单ID', trigger: 'blur' },
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
                        key: 'pid'
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
                'menuId':0
            });

            // 初始化搜索框中的下拉菜单
            this.axios({
                method: 'get',
                url: this.GLOBAL.menu_getRootMenus_url
            }).then(function (response) {
                if(response.data.errCode == '0000') {
                    // 初始化搜索下拉菜单
                    var listTemp = response.data.data;
                    for (var i = 0; i < listTemp.length; i++) {
                        this.menuList.push({
                            "value": listTemp[i].id,
                            "label": listTemp[i].name
                        });
                    }
                } else {
                    this.$Message.info('加载根菜单失败');
                }
            }.bind(this)).catch(function (error) {
                alert(error);
            });
        },
        methods:{
            /*pageInfo实体初始化*/
            initPageInfo(){
                this.pageInfo.page = 1;
                this.pageInfo.pageSize = 10;
            },
            /*menu实体初始化*/
            initMenu(){
                this.menu.id = null;
                this.menu.name = null;
                this.menu.url = null;
                this.menu.pid = null;
                this.menu.sort = null;
                this.menu.icon = null;
            },
            /*menuNew实体初始化*/
            initMenuNew(){
                this.menuNew.id = null;
                this.menuNew.name = null;
                this.menuNew.url = null;
                this.menuNew.pid = null;
                this.menuNew.sort = null;
                this.menuNew.icon = null;
            },
            /*menuModify实体初始化*/
            initMenuModify(){
                this.menuModify.id = null;
                this.menuModify.name = null;
                this.menuModify.url = null;
                this.menuModify.pid = null;
                this.menuModify.sort = null;
                this.menuModify.icon = null;
            },
            /*menuNew设置*/
            menuSet(e){
                this.menu.id = e.id;
                this.menu.name = e.name;
                this.menu.url = e.url;
                this.menu.pid = e.pid;
                this.menu.sort = e.sort;
                this.menu.icon = e.icon;
            },
            /*menuNew设置*/
            menuNewSet(e){
                this.menuNew.id = e.id;
                this.menuNew.name = e.name;
                this.menuNew.url = e.url;
                this.menuNew.pid = e.pid;
                this.menuNew.sort = e.sort;
                this.menuNew.icon = e.icon;
            },
            /*menuModify设置*/
            menuModifySet(e){
                this.menuModify.id = e.id;
                this.menuModify.name = e.name;
                this.menuModify.url = e.url;
                this.menuModify.pid = e.pid+'';
                this.menuModify.sort = e.sort+'';
                this.menuModify.icon = e.icon;
            },
            /*得到表数据*/
            getTable(e) {
                this.axios({
                    method: 'get',
                    url: this.GLOBAL.menu_getSubMenusByPage_url,
                    params: {
                        'pageNo':e.pageInfo.page,
                        'pageSize':e.pageInfo.pageSize,
                        'menuId':e.menuId
                    }
                }).then(function (response) {
                    if(response.data.errCode == '0000') {
                        this.data1=response.data.data.list;
                        this.total=response.data.data.total;
                    } else {
                        this.$Message.info('加载菜单列表失败');
                    }
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
                            url: this.GLOBAL.menu_add_url,
                            data: this.menu
                        }).then(function (response) {
                            if(response.data.errCode == '0000') {
                                this.initMenuNew();
                                this.getTable({
                                    "pageInfo":this.pageInfo,
                                    'menuId':this.menuId
                                });
                                this.$Message.info('新建菜单成功');
                            } else {
                                this.$Message.info('新建菜单失败');
                            }
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
                    this.$Message.warning('请选择一项');
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
                            url: this.GLOBAL.menu_update_url,
                            data: this.menu
                        }).then(function (response) {
                            if(response.data.errCode == '0000') {
                                this.initMenuNew();
                                this.getTable({
                                    "pageInfo":this.pageInfo,
                                    'menuId':this.menuId
                                });
                                this.$Message.info('修改成功');
                            } else {
                                this.$Message.info('修改失败');
                            }
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
                        method: 'get',
                        url: this.GLOBAL.menu_delByBatch_url + "/" + this.groupId.join(',')
                    }).then(function (response) {
                        if(response.data.errCode == '0000') {
                            this.getTable({
                                "pageInfo":this.pageInfo,
                                'menuId':this.menuId
                            });
                            this.groupId=null;
                            this.count=0;
                            this.$Message.info('删除成功');
                        } else {
                            this.$Message.info('删除失败');
                        }

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
