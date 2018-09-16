<template>
    <div class="login-wrap">
        <div class="ms-title">后台管理系统</div>
        <div class="ms-login">
            <el-form :model="loginForm" :rules="rules" ref="loginForm" label-width="0px" class="demo-loginForm">
                <el-form-item prop="username">
                    <el-input v-model="loginForm.loginName" placeholder="username"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input type="password" placeholder="password" v-model="loginForm.password" @keyup.enter.native="login('loginForm')"></el-input>
                </el-form-item>
                <div class="login-btn">
                    <el-button type="primary" @click="login('loginForm')">登录</el-button>
                </div>
                <p style="font-size:12px;line-height:30px;color:#999;">Tips : 用户名和密码随便填。</p>
            </el-form>
        </div>
    </div>
</template>

<script>
    export default {
        data: function(){
            return {
                loginForm: {
                    loginName: null,
                    password: null
                },
                rules: {
                    loginName: [
                        { required: true, message: '请输入用户名', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入密码', trigger: 'blur' }
                    ]
                }
            }
        },
        methods: {
            login(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        // 后台请求登录
                        this.axios({
                            method: 'post',
                            url: this.GLOBAL.login_login_url,
                            data: this.loginForm
                        }).then(function (response) {
                            if(response.data.errCode == '0000') {
                                // 存储登录用户信息
                                localStorage.setItem('loginUser', JSON.stringify(response.data.data));
                                // 跳转到主页
                                this.$router.push('/');
                            } else {
                                this.$message.info('登录失败');
                            }
                        }.bind(this)).catch(function (error) {
                            alert(error);
                        });

                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .login-wrap{
        position: relative;
        width:100%;
        height:100%;
    }
    .ms-title{
        position: absolute;
        top:50%;
        width:100%;
        margin-top: -230px;
        text-align: center;
        font-size:30px;
        color: #fff;

    }
    .ms-login{
        position: absolute;
        left:50%;
        top:50%;
        width:300px;
        height:160px;
        margin:-150px 0 0 -190px;
        padding:40px;
        border-radius: 5px;
        background: #fff;
    }
    .login-btn{
        text-align: center;
    }
    .login-btn button{
        width:100%;
        height:36px;
    }
</style>
