<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img alt="" src="../assets/logo.png">
        <span>电商管理后台</span>
      </div>
      <el-button @click="logout" type="info">退出</el-button>
    </el-header>
    <!--  页面主体  -->
    <el-container>
      <!--   侧边栏   -->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <div @click="toggleCollapse" class="toggle-button">|||</div>
        <!--  侧边栏菜单区域  -->
        <el-menu
          :collapse="isCollapse"
          :collapse-transition="false"
          active-text-color="#ffd04b"
          background-color="#333744"
          :default-active="$route.path"
          router
          text-color="#fff"
          unique-opened>
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" :key="item.id" v-for="item in menuList">
            <template slot="title">
              <!--  图标  -->
              <i class="el-icon-location"/>
              <!--  文字  -->
              <span>{{item['authName']}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' + subItem.path" :key="subItem.id" @click="saveNavState('/' + subItem.path)"
                          v-for="subItem in item.children">
              <template slot="title">
                <!--  图标  -->
                <i class="el-icon-menu"/>
                <!--  文字  -->
                <span>{{subItem['authName']}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!--   右侧主体内容   -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data () {
      return {
        menuList: [],
        // 是否折叠
        isCollapse: false,
        activePath: ''
      }
    },
    created () {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
      logout () {
        window.sessionStorage.clear()
        this.$message.success('退出成功！')
        this.$router.push('/login')
      },
      // 获取所有的菜单
      async getMenuList () {
        const { data: res } = await this.$http.get('menus')
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menuList = res.data
      },
      // 菜单折叠和展开
      toggleCollapse () {
        this.isCollapse = !this.isCollapse
      },
      saveNavState (activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath
      }
    }
  }
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 5;
    align-items: center;
    color: #fff;
    font-size: 25px;

    > div {
      display: flex;
      align-items: center;

      > span {
        margin-left: 15px;
      }
    }

    img {
      height: 40px;
      width: 40px;
    }
  }

  .el-aside {
    background-color: #333744;

    .el-menu {
      border: none;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }

  .toggle-button {
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
</style>
