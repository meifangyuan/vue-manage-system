<script>

    const hostUrl = 'http://localhost:8080/blog/';

    // 菜单管理接口
    const menu_add_url = hostUrl + 'sys/menu/add';
    const menu_delByBatch_url = hostUrl + 'sys/menu/delByBatch';
    const menu_update_url = hostUrl + 'sys/menu/update';
    const menu_getRootMenus_url = hostUrl + 'sys/menu/getRootMenus';
    const menu_getSubMenusByPage_url = hostUrl + 'sys/menu/getSubMenusByPage';
    const menu_getMenusByRole_url = hostUrl + 'sys/menu/getMenusByRole';

    export default
    {
        hostUrl,
        menu_add_url,
        menu_delByBatch_url,
        menu_update_url,
        menu_getRootMenus_url,
        menu_getSubMenusByPage_url,
        menu_getMenusByRole_url
    }
</script>
