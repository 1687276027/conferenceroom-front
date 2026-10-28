<template>

    <div id="body">
        <div class="mask"></div>

        <div class="login-box">
            <div class="login-brand">
                <i class="iconfont iconhuiyishi"></i>
                <h1>2027lgy 会议室管理系统</h1>
                <p>Conference Room Management System</p>
            </div>

            <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" class="login-form"
                     @keyup.enter.native="submitForm('ruleForm')">
                <el-form-item prop="username">
                    <el-input placeholder="请输入账号" prefix-icon="el-icon-user" v-model="ruleForm.username"></el-input>
                </el-form-item>

                <el-form-item prop="password">
                    <el-input placeholder="请输入密码" prefix-icon="el-icon-lock" show-password
                              type="password" v-model="ruleForm.password"></el-input>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" class="login-btn" @click="submitForm('ruleForm')">登 录</el-button>
                    <el-button class="reset-btn" @click="resetForm('ruleForm')">重 置</el-button>
                </el-form-item>
            </el-form>

            <div class="login-tip">
                <p><i class="el-icon-info"></i> 部门账号为部门编号，员工账号为工号，初始密码 123456，登录后可自行修改</p>
                <p>如需帮助请联系管理员：178067066xx / 619122012@qq.com</p>
            </div>
        </div>

        <div class="login-footer">© 2027 lgy 会议室管理系统</div>
    </div>
</template>

<script>
    import Cookies from 'js-cookie'
    export default {
        name: "login",
        data() {
            return {
                ruleForm: {
                    username: '',
                    password:'',
                },
                rules: {
                    username: [
                        { required: true, message: '请输入账号', trigger: 'blur' },
                        { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入密码', trigger: 'blur' },
                        { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                    ]
                }
            };
        },
        methods: {
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                       let _this = this;
                       this.axios.post("/login",_this.ruleForm).then(res=>{
                           const jwt = res.headers['authorization'];
                           const userInfo = res.data.data;
                           _this.$store.commit('SET_TOKEN',jwt);
                           _this.$store.commit('SET_USERINFO',userInfo);

                           this.$message({
                               message: '恭喜你登录成功',
                               type: 'success'
                           });

                           _this.$router.push("/index");
                       })
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            },
            resetForm(formName) {
                this.$refs[formName].resetFields();
            }
        },
        created() {

            //如果访问网站或者通过地址访问login  有Cookies则直接到index
            //退出按钮时会清除localstorage session cookie

            if(Cookies.get("userInfo")){
                //更新store session
                this.$store.commit('SET_USERINFO',JSON.parse(Cookies.get("userInfo")));
                sessionStorage.setItem("userInfo",Cookies.get("userInfo"));
                this.$router.push("/index");
            }
        }
    }
</script>

<style scoped>
    #body {
        position: fixed;
        width: 100%;
        height: 100%;
        background-image: url('../assets/images/meetingroom.jpg');
        background-position: center center;
        background-size: cover;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    /* 背景压暗一层，保证表单可读 */
    .mask {
        position: absolute;
        inset: 0;
        background: linear-gradient(120deg, rgba(16, 30, 56, .86) 0%, rgba(16, 30, 56, .58) 45%, rgba(16, 30, 56, .38) 100%);
    }

    .login-box {
        position: relative;
        width: 400px;
        padding: 38px 36px 26px;
        border-radius: 18px;
        background: rgba(255, 255, 255, .97);
        box-shadow: 0 24px 60px rgba(8, 18, 40, .35);
    }

    .login-brand {
        text-align: center;
        margin-bottom: 26px;
    }

    .login-brand i {
        font-size: 34px;
        color: #2f6bff;
    }

    .login-brand h1 {
        margin: 10px 0 6px;
        font-size: 22px;
        font-weight: 600;
        color: #1f2d3d;
        letter-spacing: 1px;
    }

    .login-brand p {
        margin: 0;
        font-size: 12px;
        color: #9aa8bd;
        letter-spacing: .3px;
    }

    .login-form >>> .el-input__inner {
        height: 44px;
        line-height: 44px;
        border-radius: 10px;
    }

    .login-form >>> .el-form-item {
        margin-bottom: 20px;
    }

    .login-btn {
        width: 100%;
        height: 44px;
        border-radius: 10px;
        font-size: 15px;
        letter-spacing: 4px;
    }

    .reset-btn {
        width: 100%;
        height: 38px;
        margin: 12px 0 0;
        border-radius: 10px;
        color: #5a6b82;
    }

    .login-form >>> .el-form-item:last-child {
        margin-bottom: 8px;
    }

    .login-tip {
        margin-top: 6px;
        padding-top: 14px;
        border-top: 1px dashed #e6ebf3;
        font-size: 12px;
        color: #8ea0b8;
        line-height: 20px;
        text-align: center;
    }

    .login-tip p {
        margin: 2px 0;
    }

    .login-tip i {
        color: #f79009;
        margin-right: 2px;
    }

    .login-footer {
        position: absolute;
        bottom: 22px;
        width: 100%;
        text-align: center;
        font-size: 12px;
        color: rgba(255, 255, 255, .55);
    }
</style>