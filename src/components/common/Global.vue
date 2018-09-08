<script>

    const hostUrl = 'http://localhost:8080/blog/';

    // 菜单管理接口
    const menu_add_url = hostUrl + 'sys/menu/add';
    const menu_delByBatch_url = hostUrl + 'sys/menu/delByBatch';
    const menu_update_url = hostUrl + 'sys/menu/update';
    const menu_getRootMenus_url = hostUrl + 'sys/menu/getRootMenus';
    const menu_getSubMenusByPage_url = hostUrl + 'sys/menu/getSubMenusByPage';
    const menu_getMenusByRole_url = hostUrl + 'sys/menu/getMenusByRole';

    // 角色管理接口
    const role_add_url = hostUrl + 'sys/role/add';
    const role_delByBatch_url = hostUrl + 'sys/role/delByBatch';
    const role_update_url = hostUrl + 'sys/role/update';
    const role_setRoleMenus_url = hostUrl + 'sys/role/setRoleMenus';
    const role_getAllRoles_url = hostUrl + 'sys/role/getAllRoles';

    // 用户管理接口
    const user_add_url = hostUrl + 'sys/user/add';
    const user_delByBatch_url = hostUrl + 'sys/user/delByBatch';
    const user_update_url = hostUrl + 'sys/user/update';
    const user_setUserRole_url = hostUrl + 'sys/user/setUserRole';
    const user_getAllUsers_url = hostUrl + 'sys/user/getAllUsers';

    export default
    {
        hostUrl,

        menu_add_url,
        menu_delByBatch_url,
        menu_update_url,
        menu_getRootMenus_url,
        menu_getSubMenusByPage_url,
        menu_getMenusByRole_url,

        role_add_url,
        role_delByBatch_url,
        role_update_url,
        role_setRoleMenus_url,
        role_getAllRoles_url,

        user_add_url,
        user_delByBatch_url,
        user_update_url,
        user_setUserRole_url,
        user_getAllUsers_url
    }
</script>
