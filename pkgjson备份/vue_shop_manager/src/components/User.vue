<template>
  <div>
    <h3>用户列表</h3>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" @clear="getUserList" clearable>
            <el-button slot="append" @click="getUserList" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 数据表格 -->
      <el-table :data="userlist" stripe border>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="手机" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userstateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!--修改按钮-->
            <el-tooltip class="item" effect="dark" content="编辑用户信息" placement="top-start">
              <el-button
                type="primary"
                icon="el-icon-edit"
                circle
                @click="showEditDialog(scope.row.id)"
              ></el-button>
            </el-tooltip>
            <!--删除按钮-->
            <el-tooltip class="item" effect="dark" content="删除用户信息" placement="top">
              <el-button
                type="danger"
                icon="el-icon-delete"
                circle
                @click="removeUserById(scope.row.id)"
              ></el-button>
            </el-tooltip>
            <!--分配角色按钮-->
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top-end">
              <el-button
                type="warning"
                icon="el-icon-setting"
                circle
                @click="showSetRoleDialog(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 4, 8, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <!-- 添加--用户的对话框 -->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%">
        <!-- 对话框内容表单 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span></span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 编辑--用户的对话框 -->
      <el-dialog title="编辑用户" :visible.sync="editDialogVisible" width="30%">
        <!-- 对话框内容表单 -->
        <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span></span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 认</el-button>
        </span>
      </el-dialog>
    </el-card>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%">
      <!-- 弹窗主体 -->
      <p>用户名：{{userInfo.username}}</p>
      <p>当前角色：{{userInfo.role_name}}</p>
      <p>
        分配新角色：
        <el-select v-model="selectRoleId" placeholder="请选择">
          <el-option
            v-for="item in rolesList"
            :key="item.id"
            :label="item.roleName"
            :value="item.id"
          ></el-option>
        </el-select>
      </p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
      

<script>
export default {
  data() {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/

      if (regEmail.test(value)) {
        // 合法的邮箱
        return cb()
      }

      cb(new Error('请输入合法的邮箱'))
    }

    // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/

      if (regMobile.test(value)) {
        return cb()
      }

      cb(new Error('请输入合法的手机号'))
    }

    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,

      // 添加用户功能
      addDialogVisible: false,
      editDialogVisible: false,
      //添加用户的表单数据
      addForm: {},
      editForm: {},
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '密码在6~15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            validator: checkEmail,
            message: '邮箱格式不正确,请重新输入',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '手机号码不正确,请重新输入',
            trigger: 'blur'
          }
        ]
      },

      //控制分配角色对话框
      setRoleDialogVisible: false,
      //需要分配角色的用户信息
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      // 分配角色下拉框的数据
      selectRoleId: ''
    }
  },

  created() {
    this.getUserList()
  },

  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })

      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      //然后将获取到的数据保存到data中
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(this.userlist)
    },

    // 切换一页显示数据条数
    handleSizeChange(newPageSize) {
      this.queryInfo.pagesize = newPageSize
      this.getUserList()
    },
    // 切换页
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },

    // 更新用户状态到后台
    async userstateChanged(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('改变状态失败')
      }
      return this.$message.success('改变状态成功')
    },

    // 添加用户功能
    async addUser() {
      this.addDialogVisible = false
      const { data: res } = await this.$http.post('users', this.addForm)
      if (res.meta.status !== 201) return this.$message.error('添加失败')
      this.getUserList()
      this.$message.success('添加成功')
    },

    // 展示编辑弹框
    async showEditDialog(uid) {
      this.editDialogVisible = true
      // 发送请求
      const { data: res } = await this.$http.get('users/' + uid)
      if (res.meta.status !== 200) return this.$message.error('请求失败')
      // 将数据保存到data的editForm
      this.editForm = res.data
      console.log(this.editForm)
    },
    // 编辑用户提交功能
    editUser() {
      this.editDialogVisible = false
      // 请求前先验证数据
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发送请求
        const { data: res } = await this.$http.put('users/' + this.editForm.id, this.editForm)
        if (res.meta.status !== 200) return this.$message.error('修改失败')
        this.$message.success('修改成功')
        this.getUserList()
      })
    },

    //删除用户功能
    async removeUserById(uid) {
      //弹框询问用户是否删除数据
      const result = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (result !== 'confirm') return this.$message.info('已取消删除')
      // 发送请求删除数据
      const { data: res } = await this.$http.delete('users/' + uid)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.getUserList()
      return this.$message.success('删除成功')
    },

    //==========分配角色
    // 显示分配角色弹框
    async showSetRoleDialog(row) {
      // row保存的就是一个用户信息
      this.userInfo = row
      this.setRoleDialogVisible = true

      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.rolesList = res.data
      this.$message.success('获取角色信息成功')
    },
    //点击确定按钮， 分配角色
    async saveRoleInfo() {
      if (!this.selectRoleId) return this.$message.error('请选择要分配的角色')
      
      const { data: res } = await this.$http.put(`users/  ${this.userInfo.id}  /role`, { rid: this.selectRoleId })
      if (res.meta.status !== 200) return this.$message.error('分配角色失败')
      this.$message.success('分配角色成功')
      this.setRoleDialogVisible = false
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
