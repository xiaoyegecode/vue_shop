<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <!-- 黑马logo -->
        <img src="../assets/heima.png" alt="">
        <!-- 顶部标题 -->
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout"> 退出 </el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 伸缩侧边栏按钮 -->
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单 -->
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409EFF" unique-opened
          :collapse="isCollapse" :collapse-transition="false" router :default-active='activePath'>
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单模板 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级子菜单 -->
            <el-menu-item :index="'/'+ subItem.path" v-for="subItem in item.children" :key="subItem.id"
              @click='saveNavState("/"+ subItem.path)'>
              <!-- 二级菜单模板 -->
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{ subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 主体结构 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList: [],
      //字体图标对象
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath:''
    }
  },
  created() {
    this.getMenuList(),
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push("/login")
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      // console.log(res);
    },
    //点击折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    //保持链接的激活状态
    saveNavState(path) {
      window.sessionStorage.setItem('activePath', path)
      this.activePath = path
    }
  }
}
</script>

<style lang="less" scoped>


    .home-container {
      height: 100%;
    }
  
    .el-header {
      background-color: #373D41;
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-left: 0;
      font-size: 20px;
      color: #fff;
      > div {
        display: flex;
        align-items: center;
        span {
          margin-left: 15px;
        }
      }
    }
  
    .el-aside {
      background-color: #333744;
      .el-menu {
        border-right: none;
      }
      .toggle-button{
        background-color: #4a5064;
        font-size: 10px;
        line-height: 24px;
        color: #fff;
        text-align: center;
        letter-spacing: 0.2em;
        cursor: pointer;
      }
    }
  
    .el-main {
      background-color: #eaedf1;
    }

    .iconfont {
      margin-right: 10px;
    }
</style>