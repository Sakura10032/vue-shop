<template>
  <div>
    <!--  面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--  卡片视图区域 -->
    <el-card class="box-card">
      <!--  搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input
            @clear="getUserList"
            clearable
            placeholder="请输入内容进行搜索"
            v-model="queryInfo.query">
            <el-button
              @click="getUserList"
              icon="el-icon-search"
              slot="append"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button @click="addDialogVisible = true" type="primary">添加用户</el-button>
        </el-col>
      </el-row>

      <!--   用户数据区域   -->
      <el-table
        :data="userList"
        border
        stripe
        style="width: 100%">
        <el-table-column
          label="#"
          type="index">
        </el-table-column>
        <el-table-column
          label="姓名"
          prop="username"
          width="180">
        </el-table-column>
        <el-table-column
          label="邮箱"
          prop="email"
          width="180">
        </el-table-column>
        <el-table-column
          label="电话"
          prop="mobile">
        </el-table-column>
        <el-table-column
          label="角色"
          prop="role_name">
        </el-table-column>
        <el-table-column
          label="状态"
          prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              @change="userStateChange(scope.row)"
              active-color="#13ce66"
              inactive-color="#ff4949"
              v-model="scope.row['mg_state']">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="操作">
          <template slot-scope="scope">
            <el-tooltip :enterable="false" content="编辑" effect="dark" placement="top">
              <el-button @click="showEditDialog(scope.row.id)" icon="el-icon-edit" size="mini" type="primary"
                         v-model="scope.row['id']"></el-button>
            </el-tooltip>
            <el-tooltip :enterable="false" content="删除" effect="dark"
                        placement="top">
              <el-button @click="removeUserById(scope.row.id)" icon="el-icon-delete" size="mini" type="danger"
                         v-model="scope.row['id']"></el-button>
            </el-tooltip>
            <el-tooltip :enterable="false" content="分配角色" effect="dark" placement="top">
              <el-button icon="el-icon-setting" size="mini" type="warning" v-model="scope.row['id']" @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        :page-sizes="[2,10, 20, 50]"
        :total="total"
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </el-card>

    <!--  添加用户的对话框  -->
    <el-dialog
      :visible.sync="addDialogVisible"
      @close="addDialogClosed"
      title="添加用户"
      width="50%">
      <!--   内容主体信息   -->
      <el-form :model="addForm" :rules="addFormRules" label-width="70px" ref="addFormRef">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!--   底部区域   -->
      <span class="dialog-footer" slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button @click="addUser()" type="primary">确 定</el-button>
      </span>
    </el-dialog>

    <!--  修改用户的对话框  -->
    <el-dialog
      :visible.sync="editDialogVisible"
      @close="editDialogClosed"
      title="修改用户"
      width="50%">
      <el-form :model="editForm" :rules="editFormRules" label-width="70px" ref="editFormRef">
        <el-form-item label="用户名">
          <el-input disabled v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span class="dialog-footer" slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button @click="editUser()" type="primary">确 定</el-button>
      </span>
    </el-dialog>

    <!--  分配角色的对话框  -->
    <el-dialog :visible.sync="setRoleDialogVisible" @close="setRoleDialogClosed" title="分配角色" width="50%">
      <div>
        <p>当前的用户：{{userInfo.username}}</p>
        <p>当前的角色：{{userInfo.role_name}}</p>
        <p>分配新角色：
          <el-select placeholder="请选择" v-model="selectedRoleId">
            <el-option :key="item.id" :label="item.roleName" :value="item.id" v-for="item in rolesList">
            </el-option>
          </el-select>
        </p>
      </div>
      <span class="dialog-footer" slot="footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button @click="saveRoleInfo" type="primary">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      let checkEmail = (rule, value, cb) => {
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if (regEmail.test(value)) {
          return cb()
        }
        cb(new Error('请输入合法邮箱'))
      }
      let checkMobile = (rule, value, cb) => {
        const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
        if (regMobile.test(value)) {
          return cb()
        }
        cb(new Error('请输入合法手机号'))
      }
      return {
        // 获取用户列表的参数
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 10
        },
        userList: [],
        total: 0,
        // 是否显示添加用户弹窗
        addDialogVisible: false,
        // 是否显示修改用户弹窗
        editDialogVisible: false,
        // 添加用户的数据表单
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        // 添加表单验证规则对象
        addFormRules: {
          username: [
            { required: true, message: '请输入登录名称', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入登录密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
          ],
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { validator: checkEmail, trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ]
        },
        // 修改表单的数据
        editForm: {},
        // 修改的表单的验证规则对象
        editFormRules: {
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { validator: checkEmail, trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ]
        },
        // 控制分配角色对话框的显示与隐藏
        setRoleDialogVisible: false,
        // 需要被分配角色的用户信息
        userInfo: {},
        // 所有角色的数据列表
        rolesList: [],
        // 已选中的角色Id值
        selectedRoleId: ''
      }
    },
    created () {
      this.getUserList()
    },
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.get('users', { params: this.queryInfo })
        if (res.meta.status !== 200) return this.$message.error('获取用户数据失败！')
        const { users } = res.data
        this.userList = users
        this.total = res.data.total
        console.log(this.userList)
      },
      // 监听 switch 开关状态的改变
      async userStateChange (userinfo) {
        const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        if (res.meta.status !== 200) {
          userinfo.mg_state = !userinfo.mg_state
          return this.$message.error('更新用户状态失败！')
        }
        this.$message.success('更新用户状态成功！')
      },
      addDialogClosed () {
        this.$refs.addFormRef.resetFields()
      },
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize
        this.getUserList()
      },
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        this.getUserList()
      },
      addUser () {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) return this.$message.error('添加失败！')
          this.$message.success('添加成功！')
          this.addDialogVisible = false
          this.getUserList()
        })
      },
      // 展示编辑的对话框
      async showEditDialog (id) {
        const { data: res } = await this.$http.get('users/' + id)
        if (res.meta.status !== 200) return this.$message.error('查询数据失败！')
        this.editForm = res.data
        this.editDialogVisible = true
      },
      // 编辑用户信息
      editUser () {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.put('users/' + this.editForm.id, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
          if (res.meta.status !== 200) return this.$message.error('更新用户信息失败！')
          this.$message.success('更新用户信息成功！')
          this.editDialogVisible = false
          this.getUserList()
        })
      },
      editDialogClosed () {
        this.$refs.editFormRef.resetFields()
      },
      // 根据 id 删除用户
      async removeUserById (id) {
        const confirmResult = await this.$confirm('你有一个大胆的想法, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '让我再想想',
          type: 'warning'
        }).catch(err => err)

        // 如果用户确认删除，则返回值为字符串 confirm
        // 如果用户取消了删除，则返回字符串 cancel

        if (confirmResult !== 'confirm') return this.$message.info('已放弃这个大胆的想法')

        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status !== 200) return this.$message.error('删除用户失败！')
        this.$message.success('删除用户成功')
        this.getUserList()
      },
      // 展示分配角色的对话框
      async setRole (userInfo) {
        this.userInfo = userInfo

        // 在展示对话框之前，获取所有角色的列表
        const { data: res } = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败！')
        }

        this.rolesList = res.data

        this.setRoleDialogVisible = true
      },
      // 点击按钮，分配角色
      async saveRoleInfo () {
        if (!this.selectedRoleId) {
          return this.$message.error('请选择要分配的角色！')
        }

        const { data: res } = await this.$http.put(
          `users/${this.userInfo.id}/role`,
          {
            rid: this.selectedRoleId
          }
        )

        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }

        this.$message.success(res.meta.msg)
        this.getUserList()
        this.setRoleDialogVisible = false
      },
      // 监听分配角色对话框的关闭事件
      setRoleDialogClosed () {
        this.selectedRoleId = ''
        this.userInfo = {}
      }
    }
  }
</script>

<style scoped>

</style>
