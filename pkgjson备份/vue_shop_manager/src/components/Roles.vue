<template>
  <div>
    <h3>角色列表</h3>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ desc: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 编辑角色对话框 -->
      <el-dialog title="编辑角色" :visible.sync="editRoleFormVisible">
        <el-form :model="roleEditList" :rules="rules" ref="ruleForm">
          <el-form-item label="角色名称" :label-width="formLabelWidth" prop="roleName">
            <el-input v-model="roleEditList.roleName" autocomplete="off"></el-input>
          </el-form-item>

          <el-form-item label="角色描述" :label-width="formLabelWidth" prop="roleDesc">
            <el-input v-model="roleEditList.roleDesc" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="resetRoleForm">取 消</el-button>
          <el-button type="primary" @click="editRoleForm">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 表格数据 -->
      <el-table row-key="id" :data="rolesList" border stripe>
        <el-table-column type="expand">
          <!-- 点击展开,显示各级权限 -->
          <template slot-scope="scope">
            <!-- 一级权限 -->
            <el-row
              :class="[ index1==scope.row.children.length-1?'':'tag-bot-border', index1 == 0?'':'tag-top-border', 'tag-vertical-center']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[ i2==item1.children.length-1?'':'tag-bot-border', i2 == 0?'':'tag-top-border', 'tag-vertical-center']"
                >
                  <!-- 二级权限 -->
                  <el-col :span="10">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <!-- 三级权限 -->
                  <el-col :span="24">
                    <el-row
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      :class="[ i3==item2.children.length-1?'':'tag-bot-border', i3 == 0?'':'tag-top-border', 'tag-vertical-center']"
                    >
                      <el-col>
                        <el-tag
                          type="warning"
                          closable
                          @close="removeRightById(scope.row, item3.id)"
                        >{{item3.authName}}</el-tag>
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>

        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column labe1="操作">
          <template slot-scope="scope">
            <el-button
              icon="el-icon-edit"
              type="primary"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              icon="el-icon-delete"
              type="danger"
              size="mini"
              @click="delRole(scope.row.id)"
            >删除</el-button>
            <el-button
              icon="el-icon-setting"
              type="warning"
              size="mini"
              @click="showSetRightsDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分配权限对话框 -->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
        <!-- 弹窗主体 -->
        <el-tree
          :data="setRightsList"
          :props="treeProps"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defKeys"
        ></el-tree>

        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible=false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 保存角色列表的数据
      rolesList: [],
      //保存编辑角色的单项数据
      roleEditList: {},

      // 编辑角色数据
      editRoleFormVisible: false,
      formLabelWidth: '120px',
      rules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' }, { min: 1, max: 8, message: '长度在 1 到 8 个字符', trigger: 'blur' }],
        roleDesc: [{ required: true, message: '请输入角色描述', trigger: 'blur' }]
      },

      //控制分配权限对话框
      setRightDialogVisible: false,
      // 保存分配权限的树形数据
      setRightsList: [],
      // 树状数据展示
      treeProps: {
        label: 'authName', //要展示到页面的属性名
        children: 'children' //子级的属性名
      },
      //树状结构中默认选中的数据
      defKeys: [],
      // 保存角色id
      roleAllotList: {}
    }
  },
  created() {
    this.getRighsList()
  },

  methods: {
    async getRighsList() {
      // 请求角色列表的数据
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.rolesList = res.data
      this.$message.success('获取数据成功')
    },

    // 编辑角色显示对话框
    async showEditDialog(eid) {
      this.editRoleFormVisible = true

      // 根据id查询用户数据
      const { data: res } = await this.$http.get('roles/' + eid)
      if (res.meta.status !== 200) return this.$message.error('拉取数据失败')
      res.data.editId = eid
      this.roleEditList = res.data
    },
    // 编辑角色提交验证
    editRoleForm() {
      this.$refs.ruleForm.validate(async valid => {
        if (!valid) return this.$message.error('请按规则填写')
        this.editRoleFormVisible = false

        // 发送请求
        const { data: res } = await this.$http.put('roles/' + this.roleEditList.editId, this.roleEditList)
        if (res.meta.status !== 200) return this.$message.error('编辑提交失败')
        this.$message.success('编辑成功')
        this.getRighsList()
      })
    },

    // 重置编辑角色表单
    resetRoleForm() {
      this.editRoleFormVisible = false
      this.$refs.ruleForm.resetFields()
    },

    //删除角色
    async delRole(rid) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result == 'confirm') {
        // 请求删除角色接口
        const { data: res } = await this.$http.delete('roles/' + rid)
        if (res.meta.status !== 200) return this.$message.error('很遗憾,删除失败,请重试！')
        this.$message.success('恭喜您,删除成功！')
        this.getRighsList()
      }
    },

    //删除三级权限
    async removeRightById(row, rid) {
      const result = await this.$confirm('此操作将永久删除该权限, 是否继续?', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(row, rid)

      if ((result = 'cancel')) return this.$message.info('您取消了操作!')
      if (result == 'confirm') {
        // 请求删除权限接口
        const { data: res } = await this.$http.delete(`roles/${row.id}/rights/${rid}`)
        if (res.meta.status !== 200) return this.$message.error('很遗憾,删除失败,请重试！')
        this.$message.success('恭喜您,删除成功！')
        this.getRighsList()
      }
    },

    //==========分配角色权限
    async showSetRightsDialog(node) {
      this.roleAllotList.roleId = node.id

      // 先重置defKeys的勾选项
      this.defKeys = []
      //调用已有权限默认勾选函数
      this.getLeafKeys(node, this.defKeys)
      this.setRightDialogVisible = true

      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限树数据失败')
      this.setRightsList = res.data
    },
    // 已有权限默认勾选函数
    getLeafKeys(node, arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => this.getLeafKeys(item, arr))
    }, 
    // 点击确认，分配权限
    allotRights() {
      console.log(this.roleAllotList.roleId);
    }

  }
}
</script>

<style lang="less" scoped>
.tag-top-border {
  padding-top: 8px;
  // border-top: 1px solid #EBEEF5;
}
.tag-bot-border {
  padding-bottom: 8px;
  border-bottom: 1px solid #ebeef5;
}
// 标签垂直居中
.tag-vertical-center {
  display: flex;
  align-items: center;
}
</style>