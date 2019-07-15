
<template>
  <el-container class="home-container">
    <!-- 头部 -->
    <el-header>电商后台管理</el-header>

    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse? '64px': '200px'">
        <el-menu
          background-color="#373d41"
          text-color="#fff"
          active-text-color="#ffd04b"
          router
          :default-active="activeP"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
        >
          <div class="toggle-button" @click="isCollapseF" v-text="toggleInner"></div>
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 一级菜单文本 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNavState('/' + subItem.path)"
            >{{subItem.authName}}</el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>

      <!-- 主体 -->
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
      menuList: this.getMenulist(),
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      activeP: '',
      isCollapse: false,
      toggleInner: '(*^▽^*)'
    }
  },
  methods: {
    loginOut() {
      // 清空token中的值
      window.sessionStorage.clear()
      // 编程式导航，强制跳转到首页，自动重定向到login页面
      this.$router.push('/')
    },
    // 侧边栏打开闭合
    handleOpen(key, keyPath) {
      console.log(key, keyPath)
    },
    handleClose(key, keyPath) {
      console.log(key, keyPath)
    },

    isCollapseF() {
      this.toggleInner = !this.isCollapse ? '(-▽-)' : '(*^▽^*)'
      this.isCollapse = !this.isCollapse
    },

    async getMenulist() {
      //发送请求获取菜单列表
      const { data: res } = await this.$http.get('menus')
      //判断一下获取列表是否成功
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },

    saveNavState(active) {
      // 设置当前点击的项的颜色
      this.activeP = active
      // 将当前点击的导航路径保存到本地缓存
      sessionStorage.setItem('activePath', active)
    }
  },
  created() {
    this.activeP = sessionStorage.getItem('activePath')
    console.log(this.activeP)
  }
}
</script>

<style lang="less" scoped>
// 将顶级容器设置为高度100%，占满全屏
.home-container {
  height: 100%;
}
// 设置头部区域的样式
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}

// 设置侧边栏样式
.el-aside {
  background-color: #333744;
  float: left;

  .collapse {
    width: 100%;
    color: #fff;
    font-size: 14px;
    text-align: center;
    line-height: 40px;
    cursor: pointer;
  }

  .el-menu {
    border-right: none;
  }
}

// 设置主体样式
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  text-align: center;
  color: #fff;
  font-size: 10px;
  line-height: 24px;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>

