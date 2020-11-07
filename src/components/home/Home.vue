<template>
  <el-container class="home-container">
    <!--    头部区域-->
    <el-header>
      <div>
        <img src="../../assets/logo.png" height="35" width="35  " alt="">
        <span>赵鑫鑫的管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--    页面主题区域-->
    <el-container>
      <!--      侧边栏-->
      <el-aside width="200px">
        <!--        侧边栏菜单-->
        <el-menu
          background-color="#333774"
          text-color="#fff"
          active-text-color="#40 9EFF">
          <!--          一级菜单-->
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <!--            一级菜单的模板区域-->
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="subItem.id+''" v-for="subItem in item.children" :key="subItem.id">
              <template slot="title">
                <i class="el-icon-location"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
            <!--            三级菜单的使用-->
            <!--            <el-menu-item-group>-->
            <!--              <template slot="title">分组一</template>-->
            <!--              <el-menu-item index="1-1">选项1</el-menu-item>-->
            <!--              <el-menu-item index="1-2">选项2</el-menu-item>-->
            <!--            </el-menu-item-group>-->
            <!--            <el-menu-item-group title="分组2">-->
            <!--              <el-menu-item index="1-3">选项3</el-menu-item>-->
            <!--            </el-menu-item-group>-->
            <!--            <el-submenu index="1-4">-->
            <!--              <template slot="title">选项4</template>-->
            <!--              <el-menu-item index="1-4-1">选项1</el-menu-item>-->
            <!--            </el-submenu>-->
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-container>
        <!--        右侧内容-->
        <el-main>Main</el-main>
        <!--        <el-footer>Footer</el-footer>-->
      </el-container>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data () {
      return {
        menuList: [],
        iconObj: {
          125: 'iconfont icon-user',
          103: 'iconfont icon-tijikongjian'
        }
      }
    },
    created () {
      this.getMenuList()
    },
    methods: {
      logout () {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMenuList () {
        const { data: res } = await this.$http.get('menus')
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menuList = res.data
        console.log(this.menuList)
      }
    },
    name: 'Home'
  }
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #373d41;
    /*设置为弹性伸缩*/
    display: flex;
    /* 两边对齐*/
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #ffffff;
    font-size: 20px;

    > div {
      display: flex;
      align-items: center;

      > span {
        margin-left: 15px;
      }
    }
  }

  .el-aside {
    background-color: #333774;
  }

  .el-main {
    background-color: #eaedf1;
  }
</style>
