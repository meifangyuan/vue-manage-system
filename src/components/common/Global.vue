<script>

    const hostUrl = 'http://localhost:8080/blog/';

    // 菜单管理接口
    const menu_add_url = hostUrl + 'sys/menu/add';
    const menu_del_url = hostUrl + 'sys/menu/del';
    const menu_delByBatch_url = hostUrl + 'sys/menu/delByBatch';
    const menu_update_url = hostUrl + 'sys/menu/update';
    const menu_getRootMenus_url = hostUrl + 'sys/menu/getRootMenus';
    const menu_getMenusByRole_url = hostUrl + 'sys/menu/getMenusByRole';
    const menu_getMenusByPage_url = hostUrl + 'sys/menu/getMenusByPage';

    // 角色管理接口
    const role_add_url = hostUrl + 'sys/role/add';
    const role_del_url = hostUrl + 'sys/role/del';
    const role_delByBatch_url = hostUrl + 'sys/role/delByBatch';
    const role_update_url = hostUrl + 'sys/role/update';
    const role_setRoleMenus_url = hostUrl + 'sys/role/setRoleMenus';
    const role_getRolesByPage_url = hostUrl + 'sys/role/getRolesByPage';
    const role_getAllRoles_url = hostUrl + 'sys/role/getAllRoles';


    // 用户管理接口
    const user_add_url = hostUrl + 'sys/user/add';
    const user_del_url = hostUrl + 'sys/user/del';
    const user_delByBatch_url = hostUrl + 'sys/user/delByBatch';
    const user_update_url = hostUrl + 'sys/user/update';
    const user_setUserRole_url = hostUrl + 'sys/user/setUserRole';
    const user_getUsersByPage_url = hostUrl + 'sys/user/getUsersByPage';
    const user_getUserRole_url = hostUrl + 'sys/user/getUserRole';

    // 登录|登出|修改密码
    const login_login_url = hostUrl + 'sys/login/login';
    const login_logout_url = hostUrl + 'sys/login/logout';
    const login_changePwd_url = hostUrl + 'sys/login/changePwd';

    export default
    {
        hostUrl,

        menu_add_url,
        menu_del_url,
        menu_delByBatch_url,
        menu_update_url,
        menu_getRootMenus_url,
        menu_getMenusByRole_url,
        menu_getMenusByPage_url,

        role_add_url,
        role_del_url,
        role_delByBatch_url,
        role_update_url,
        role_setRoleMenus_url,
        role_getRolesByPage_url,
        role_getAllRoles_url,

        user_add_url,
        user_del_url,
        user_delByBatch_url,
        user_update_url,
        user_setUserRole_url,
        user_getUsersByPage_url,
        user_getUserRole_url,

        login_login_url,
        login_logout_url,
        login_changePwd_url
    }
</script>
