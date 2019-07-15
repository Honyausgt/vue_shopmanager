<template>
  <div>
    <h3>权限列表</h3>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card></el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 获取商品分类数据的请求参数
      queryInfo: {
        type: [1],
        pagenum: 1,
        pagesize: 2
      },
      //   商品分类信息
      cateInfo: []
    }
  },

  created() {
    this.getGoodsCateList()
  },

  methods: {
    //   获取商品分类列表数据
    async getGoodsCateList() {
      const { data: res } = await this.$http.get('categories', { paramas: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('拉取数据败')
      this.cateInfo = res.data
      console.log(this.cateInfo)
    }
  }
}
</script>

<style <style lang="less" scoped>
</style>
