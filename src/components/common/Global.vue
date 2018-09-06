<script>

    const hostUrl = 'http://localhost:8081/blog/';

    // 菜单管理接口
    const menu_add_url = hostUrl + 'sys/menu/add';
    const menu_del_url = hostUrl + 'sys/menu/del';
    const menu_update_url = hostUrl + 'sys/menu/update';
    const menu_listByPid_url = hostUrl + 'sys/menu/listByPid';
    const menu_listByRole_url = hostUrl + 'sys/menu/listByRole';



    export default
    {
        hostUrl,
        menu_add_url,
        menu_del_url,
        menu_update_url,
        menu_listByPid_url,
        menu_listByRole_url
    }
</script>
