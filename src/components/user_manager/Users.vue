<template>
  <!--  面包屑导航区-->
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--    卡片区域-->
    <el-card>
      <el-row :gutter="100">
        <el-col :span="10">
          <el-input placeholder="请输入内容" class="input-with-select" v-model="queryInfo.query" clearable
                    @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="7">
          <el-button type="primary" @click="showAddDialog()">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" style="width: 100%" border stripe>

        <!--        添加索引列-->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="id" label="id"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="create_time" label="日期"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mobile" label="手机号"></el-table-column>
        <el-table-column prop="email" label="email"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" round
                       @click="showEditDialog(scope.row)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUserById(scope.row.id)"
                       round></el-button>
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" round></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!--      分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!--    添加用户对话框-->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="30%"
      @close="dialogClose">
      <!--      内容主题区域-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!--      底部区域-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
    </el-dialog>
    <!--    修改用户对话框  校验规则使用 添加用户的校验规则-->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%" @close="editDialogClose">
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="用户名">
          <el-input disabled v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUser">确 定</el-button>
  </span>
    </el-dialog>
    <!--    删除用户弹窗-->

  </div>
</template>

<script>
  export default {
    name: 'Users',
    data () {
      // 检查邮箱
      const checkEmail = (rule, value, callback) => {
        // eslint-disable-next-line no-useless-escape
        const regEmail = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/
        if (regEmail.test(value)) {
          return callback()
        } else {
          callback(new Error('请输入合法邮箱'))
        }
      }
      // 检查手机号
      const checkMobile = (rule, value, callback) => {
        // eslint-disable-next-line no-useless-escape
        const regPhone = /^1[3|4|5|7|8]\d{9}$/
        if (regPhone.test(value)) {
          return callback()
        } else {
          callback(new Error('请输入合法手机号'))
        }
      }
      return {
        queryInfo: {
          query: '',
          // 当前页数
          pagenum: 1,
          // 当前每页显示多少条数据
          pagesize: 2
        },
        userList: [],
        total: 0,
        // 控制 添加用户表单隐藏和显示
        addDialogVisible: false,
        // 添加用户表单数据
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        // 添加用户校验规则
        addFormRules: {
          username: [
            {
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 9,
              message: '长度在 3 到 9 个字符',
              trigger: 'blur'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 6,
              max: 10,
              message: '长度在 6 到 10 个字符',
              trigger: 'blur'
            }
          ],
          email: [
            {
              required: true,
              message: '请输入邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [
            {
              required: true,
              message: '请输入手机号',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ]
        },
        // 控制 添加用户表单隐藏和显示
        editDialogVisible: false,
        editForm: {
          id: '',
          username: '',
          email: '',
          mobile: ''
        }
      }
    },
    created () {
      this.getUserList()
    },
    methods: {
      // 获取用户列表
      async getUserList () {
        const { data: res } = await this.$http.get('users', { params: this.queryInfo })
        if (res.meta.status !== 200) {
          return this.$message.error('获取用户列表失败')
        }
        this.userList = res.data.users
        this.total = res.data.total
      },
      // 修改每页大小
      handleSizeChange (newSize) {
        // console.log(newSize)
        this.queryInfo.pagesize = newSize
        // 重新加载数据
        this.getUserList()
      },
      // 修改页数
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        // 重新加载数据
        this.getUserList()
      },
      // 修改用户状态
      async userStateChange (userInfo) {
        const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        if (res.meta.status !== 200) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error('修改失败')
        }
        this.$message.success('修改成功')
        console.log(res)
      },
      // 打开添加用户弹窗
      showAddDialog () {
        this.addDialogVisible = true
      },
      // 监听添加用户窗口关闭
      dialogClose () {
        this.$refs.addFormRef.resetFields()
      },
      // 点击确定按钮  添加用户
      addUser () {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          // 可以发起添加用户的网络请求
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败')
          }
          this.$message.success('添加用户成功')
          // 关闭添加用户对话框
          this.addDialogVisible = false
          this.getUserList()
        })
      },
      // 控制编辑用户对话框展示
      showEditDialog (data) {
        // console.log(data)
        this.editForm.id = data.id
        this.editForm.username = data.username
        this.editForm.email = data.email
        this.editForm.mobile = data.mobile
        this.editDialogVisible = true
      },
      // 监听修改用户对话框的关闭事件
      editDialogClose () {
        this.$refs.editFormRef.resetFields()
      },
      // 点击确定按钮  修改用户信息
      editUser () {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return
          // 可以发起修改用户请求
          const { data: res } = await this.$http.put(`users/${this.editForm.id}`, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
          if (res.meta.status !== 200) {
            this.$message.error('更新失败')
          }
          this.$message.success('更新用户成功')
          this.editDialogVisible = false
          this.getUserList()
        })
      },
      // 删除用户   如果用户确认删除 则返回字符串   confirm
      //
      deleteUserById (id) {
        this.$confirm('将要删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          const { data: res } = await this.$http.delete('users/' + id)
          console.log(res)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.getUserList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }).catch(() => {
          console.log('取消删除')
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      }
    }
  }
</script>

<style scoped>

</style>
