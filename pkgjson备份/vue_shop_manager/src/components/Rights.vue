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
    <el-card>
      <!-- 添加权限对话框 -->
      <el-button type="primary" @click="addRightsFormVisible = true">新增权限</el-button>

      <el-dialog title="新增权限" :visible.sync="addRightsFormVisible">
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm">
          <el-form-item label="权限名称" :label-width="formLabelWidth" required prop="name">
            <el-input v-model="ruleForm.name" autocomplete="off"></el-input>
          </el-form-item>

          <el-form-item label="路径" :label-width="formLabelWidth" required prop="path">
            <el-input v-model="ruleForm.path" autocomplete="off"></el-input>
          </el-form-item>

          <el-form-item label="权限等级" :label-width="formLabelWidth" required prop="level">
            <el-select v-model="ruleForm.level" placeholder="请设置该权限的等级">
              <el-option label="一级" value="1"></el-option>
              <el-option label="二级" value="2"></el-option>
              <el-option label="三级" value="3"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="resetRightsForm">取 消</el-button>
          <el-button type="primary" @click="addRightsForm">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 表格数据 -->
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="leve1">
          <template slot-scope="scope">
            <el-tag color="#E6A23C" type="info" v-if="scope.row.level==0">一级</el-tag>
            <el-tag color="#FAECD8" type="info" v-else-if="scope.row.level==1">二级</el-tag>
            <el-tag color="#FDF6EC" type="info" v-else>三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 保存权限列表的数据
      rightsList: [],

      // 添加权限数据
      addRightsFormVisible: false,
      formLabelWidth: '120px',
      ruleForm: {
        name: '',
        path: '',
        level: ''
      },
      rules: {
        name: [{ required: true, message: '请输入权限名称', trigger: 'blur' }, { min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }],
        path: [{ required: true, message: '请输入路径名称', trigger: 'blur' }],
        level: [{ required: true, message: '请设置权限等级', trigger: 'change' }]
      }
    }
  },
  created() {
    this.getRighsList()
  },

  methods: {
    async getRighsList() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.rightsList = res.data
      this.$message.success('获取数据成功')
    },

    // 添加权限提交验证
    addRightsForm() {
      this.$refs.ruleForm.validate(valid => {
        if (!valid) return this.$message.error('请填写完整')
        this.addRightsFormVisible = false
        
        // 请求后台数据
      })
    },
    resetRightsForm() {
      this.addRightsFormVisible = false
      this.$refs.ruleForm.resetFields()
    },

  }
}
</script>

<style lang="less" scoped>
</style>


