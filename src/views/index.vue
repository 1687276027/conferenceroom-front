<template >
    <div id="app">
        <el-container class="main">
            <el-aside :width="tabWidth+'px'" class="sidebar">
                <div class="brand" :class="{collapsed:isCollapse}">
                    <i class="iconfont iconhuiyishi logo"></i>
                    <div class="brand-text" v-if="!isCollapse">
                        <span class="brand-name">2027lgy</span>
                        <span class="brand-sub">会议室管理系统</span>
                    </div>
                </div>

                <el-menu :default-active="activeIndex"
                         class="side-menu"
                         @open="handleOpen"
                         @close="handleClose"
                         :collapse="isCollapse"
                         :collapse-transition="false"
                         background-color="transparent"
                         text-color="#c3ceE2"
                         active-text-color="#ffffff"
                         :router="true">

                    <el-submenu index="1" :disabled="isAdmin">
                        <template slot="title">
                            <i class="iconfont iconshenqing"></i>
                            <span slot="title">会议室申请</span>
                        </template>
                        <el-menu-item index="/applyConference">申请会议室</el-menu-item>
                        <el-menu-item index="/conferenceRecordByDep">查看记录</el-menu-item>
                    </el-submenu>

                    <el-menu-item index="/department" :disabled="!isAdmin">
                        <i class="iconfont iconbumen"></i>
                        <span slot="title">部门管理</span>
                    </el-menu-item>

                    <el-menu-item index="/conference_room" :disabled="!isAdmin">
                        <i class="iconfont iconhuiyishi"></i>
                        <span slot="title">会议室管理</span>
                    </el-menu-item>

                    <el-submenu index="4" :disabled="!isAdmin">
                        <template slot="title">
                            <i class="iconfont iconmenu-s"></i>
                            <span slot="title">申请管理</span>
                        </template>
                        <el-menu-item index="/conferenceRecordByAdmin">查看申请</el-menu-item>
                        <el-menu-item index="/applyConferenceByAdmin">紧急申请</el-menu-item>
                    </el-submenu>
                </el-menu>

                <div class="sidebar-bottom" v-if="!isCollapse">
                    <span>© 2027 lgy</span>
                </div>
            </el-aside>

            <el-container>
                <el-header class="main-header" height="64px">
                    <div class="header-left">
                        <i class="el-icon-s-fold toggle"
                           :class="isCollapse?'is-collapse':''"
                           @click="isClossTabFun"></i>
                        <span class="header-title">{{ currentTitle }}</span>
                    </div>

                    <div class="header-right">
                        <el-dropdown @command="dropdown" trigger="click">
                            <div class="user-box">
                                <span class="avatar">{{ username ? username.substring(0,1) : 'U' }}</span>
                                <span class="user-name">{{ username }}</span>
                                <i class="el-icon-arrow-down"></i>
                            </div>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item v-if="isDep" command="部门信息" icon="el-icon-office-building">部门信息</el-dropdown-item>
                                <el-dropdown-item v-if="isEmployee" command="员工信息" icon="el-icon-user">员工信息</el-dropdown-item>
                                <el-dropdown-item command="退出" divided icon="el-icon-switch-button">退出登录</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </div>
                </el-header>

                <el-main class="main-content">
                    <el-breadcrumb separator="/" class="crumbs">
                        <el-breadcrumb-item v-for="(item)  in levelList" :key="item.path" v-if="item.meta.title">
                            <router-link :to="item.redirect||item.path">{{item.meta.title}}</router-link>
                        </el-breadcrumb-item>
                    </el-breadcrumb>
                    <div class="page-body">
                        <router-view></router-view>
                    </div>
                </el-main>

                <el-footer class="main-footer" height="46px">
                    <p>© 2027 lgy 会议室管理系统</p>
                </el-footer>
            </el-container>
        </el-container>
    </div>
</template>

<script>
    import Cookies from 'js-cookie'
    export default {
        data() {
            return {
                isCollapse: false,
                tabWidth: 200,
                test1: 1,
                intelval: null,
                levelList: null,
                isAdmin:true,
                username:'王小虎',
                activeIndex:'',
                isEmployee:false,
                isDep:false,

            };
        },
        computed: {
            currentTitle() {
                let meta = this.$route && this.$route.meta;
                return (meta && meta.title) ? meta.title : '工作台';
            }
        },
        methods: {

            //data会返回下拉菜单中绑定的额值
            dropdown(data){
                if(data==='退出')
                    this.logout()
                else if(data ==='部门信息'){
                    this.$router.push("/departmentInfo")
                }else{
                    this.$router.push("/employeeInfo")
                }
            },

            //退出
            logout(){

                const _this = this;
                this.axios.get("/logout",{
                    headers:{
                        "Authorization":localStorage.getItem("token")
                    }
                }).then(res=>{
                    _this.$message.success("登出成功");
                    _this.$store.commit("REMOVE_INFO");
                    _this.username='';
                    _this.$router.push("/login")
                })
            },

            /**
             * 生成面包屑的方法
             */
            getBreadcrumb() {
                let matched = this.$route.matched.filter(item => item.name)

                const first = matched[0];

                if (first && first.name !== 'index') {
                    matched = [{path: '/index', meta: {title: '首页'}}].concat(matched)
                }
                this.levelList = matched;
                if(matched.length>1){
                    this.activeIndex = matched[1].path;
                }else
                    this.activeIndex = matched[0].path;
            },

            handleOpen(key, keyPath) {
                //console.log(key, keyPath);
            },
            handleClose(key, keyPath) {
               // console.log(key, keyPath);
            },

            isClossTabFun() {
                clearInterval(this.intelval);
                if (!this.isCollapse) {
                    this.intelval = setInterval(() => {
                        if (this.tabWidth <= 64)
                            clearInterval(this.intelval);
                        this.tabWidth -= 1;
                    }, 1);
                } else {
                    this.tabWidth = 200;
                }
                this.isCollapse = !this.isCollapse;
            }
        },
        created() {
            //获取面包屑
            this.getBreadcrumb();

           let userInfo = JSON.parse(Cookies.get("userInfo"));
            //获取角色 侧边栏权限
            if(userInfo.role === 'admin'){
                this.isAdmin = true;
            }else{
                if(userInfo.depId !== undefined){
                    this.isEmployee = true;
                    this.isAdmin = false;
                    this.isDep = false;
                }else{
                    this.isEmployee = false;
                    this.isAdmin = false;
                    this.isDep = true;
                }
            }
            if(userInfo.username){
                this.username = userInfo.username;
            }
        },
        watch: {
            $route(to, from) {
                this.getBreadcrumb();
            }
        }
    }
</script>

<style scoped>
    .main {
        height: 100vh;
        min-width: 900px;
        min-height: 620px;
        overflow: hidden;
    }

    /* ---------------- 侧边栏 ---------------- */
    .sidebar {
        height: 100%;
        background: linear-gradient(180deg, #1b2740 0%, #131c2e 100%);
        display: flex;
        flex-direction: column;
        overflow: hidden;
        transition: width .2s;
    }

    .brand {
        height: 64px;
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 0 18px;
        color: #fff;
        white-space: nowrap;
        border-bottom: 1px solid rgba(255, 255, 255, .06);
        box-sizing: border-box;
    }

    .brand-text {
        display: flex;
        flex-direction: column;
        justify-content: center;
        line-height: 1.2;
        overflow: hidden;
    }

    .brand-name {
        font-size: 16px;
        font-weight: 700;
        letter-spacing: .5px;
    }

    .brand-sub {
        font-size: 11px;
        color: rgba(255, 255, 255, .55);
        margin-top: 2px;
        letter-spacing: 2px;
    }

    .brand.collapsed {
        justify-content: center;
        padding: 0;
    }

    .brand .logo {
        font-size: 22px;
        color: #6ea0ff;
    }

    .side-menu {
        border-right: none;
        padding: 10px 10px 0;
        flex: 1;
        overflow-y: auto;
        overflow-x: hidden;
    }

    .sidebar-bottom {
        padding: 12px 20px;
        font-size: 12px;
        color: rgba(255, 255, 255, .3);
        text-align: center;
    }

    /* 菜单项：圆角 + 悬浮高亮 */
    .side-menu >>> .el-menu-item,
    .side-menu >>> .el-submenu__title {
        height: 46px;
        line-height: 46px;
        border-radius: 10px;
        margin-bottom: 4px;
        font-size: 14px;
    }

    .side-menu >>> .el-menu-item i,
    .side-menu >>> .el-submenu__title i {
        color: #8fa4c4;
        font-size: 16px;
        margin-right: 10px;
    }

    .side-menu >>> .el-menu-item:hover,
    .side-menu >>> .el-submenu__title:hover {
        background: rgba(255, 255, 255, .07) !important;
        color: #fff !important;
    }

    .side-menu >>> .el-menu-item.is-active {
        background: linear-gradient(135deg, #3d76ff, #2f6bff) !important;
        color: #fff !important;
        box-shadow: 0 6px 16px rgba(47, 107, 255, .35);
    }

    .side-menu >>> .el-menu-item.is-active i {
        color: #fff;
    }

    .side-menu >>> .el-submenu .el-menu-item {
        min-width: auto;
        padding-left: 46px !important;
        background: transparent;
    }

    .side-menu >>> .el-submenu.is-opened > .el-submenu__title {
        color: #fff !important;
    }

    .side-menu >>> .el-menu-item.is-disabled {
        opacity: .35;
    }

    /* 折叠时只显示图标，去掉内边距保证居中 */
    .side-menu.el-menu--collapse {
        padding: 10px 8px 0;
    }

    .side-menu.el-menu--collapse >>> .el-menu-item,
    .side-menu.el-menu--collapse >>> .el-submenu__title {
        padding: 0 !important;
        text-align: center;
    }

    .side-menu.el-menu--collapse >>> .el-menu-item i,
    .side-menu.el-menu--collapse >>> .el-submenu__title i {
        margin-right: 0;
    }

    /* ---------------- 顶栏 ---------------- */
    .main-header {
        background: #fff;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 24px;
        box-shadow: 0 1px 6px rgba(31, 45, 61, .06);
        z-index: 10;
    }

    .header-left {
        display: flex;
        align-items: center;
        gap: 14px;
    }

    .toggle {
        font-size: 20px;
        color: #5a6b82;
        cursor: pointer;
        transition: transform .2s;
    }

    .toggle:hover {
        color: #2f6bff;
    }

    .toggle.is-collapse {
        transform: rotate(180deg);
    }

    .header-title {
        font-size: 16px;
        font-weight: 600;
        color: #1f2d3d;
    }

    .user-box {
        display: flex;
        align-items: center;
        gap: 8px;
        cursor: pointer;
        padding: 6px 10px;
        border-radius: 10px;
        transition: background .2s;
    }

    .user-box:hover {
        background: #f4f6fa;
    }

    .avatar {
        width: 30px;
        height: 30px;
        border-radius: 50%;
        background: linear-gradient(135deg, #4d85ff, #2f6bff);
        color: #fff;
        font-size: 14px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: 600;
    }

    .user-name {
        font-size: 14px;
        color: #344563;
    }

    .user-box i {
        font-size: 12px;
        color: #9aa8bd;
    }

    /* ---------------- 内容区 ---------------- */
    .main-content {
        background: #f4f6fa;
        padding: 18px 22px 8px;
        overflow-y: auto;
    }

    .crumbs {
        margin-bottom: 14px;
        font-size: 13px;
    }

    .page-body {
        background: #fff;
        border-radius: 14px;
        padding: 22px;
        min-height: calc(100vh - 210px);
        box-sizing: border-box;
        box-shadow: 0 2px 14px rgba(31, 45, 61, .06);
    }

    .main-footer {
        background: #f4f6fa;
        text-align: center;
        font-size: 12px;
        line-height: 46px;
        color: #9aa8bd;
    }

    .main-footer p {
        margin: 0;
    }
</style>