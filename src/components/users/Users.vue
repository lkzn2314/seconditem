<template>
	<div id="users">
		<!-- 面包屑导航栏 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>用户管理</el-breadcrumb-item>
			<el-breadcrumb-item>用户列表</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图 -->
		<el-card class="box-card">
			<el-row :gutter="20">
				
				<el-col :span="8">
					<el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
						<el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
					</el-input>
				</el-col>
				
				<el-col :span="6">
					<el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
				</el-col>
			</el-row>
			
			<!-- 添加用户对话框 -->
			<el-dialog title="添加用户" :visible.sync="dialogVisible"
			@close="dialogClose" width="50%">
			  <span>
					<el-form :model="addForm" :rules="addFormRules" 
					ref="addFormRef" label-width="70px">
					  <el-form-item label="用户名" prop="username">
					    <el-input v-model="addForm.username"></el-input>
					  </el-form-item>
						<el-form-item label="密码" prop="password">
						  <el-input v-model="addForm.password"></el-input>
						</el-form-item>
						<el-form-item label="邮箱" prop="email">
						  <el-input v-model="addForm.email"></el-input>
						</el-form-item>
						<el-form-item label="手机" prop="mobile">
						  <el-input v-model="addForm.mobile"></el-input>
						</el-form-item>
					</el-form>	
				</span>
			  <span slot="footer" class="dialog-footer">
			    <el-button @click="dialogVisible = false">取 消</el-button>
			    <el-button type="primary" @click="addUser">确 定</el-button>
			  </span>
			</el-dialog>
			
			<!-- 修改用户对话框 -->
			<el-dialog title="修改用户" :visible.sync="editDialogVisible"
			width="50%" @close="editDialogClosed">
			  <span>
					<el-form :model="editForm" :rules="editFormRules" 
					ref="editFormRef" label-width="70px">
					  <el-form-item label="用户名">
					    <el-input v-model="editForm.username" disabled></el-input>
					  </el-form-item>
						<el-form-item label="邮箱" prop="email">
						  <el-input v-model="editForm.email"></el-input>
						</el-form-item>
						<el-form-item label="手机" prop="mobile">
						  <el-input v-model="editForm.mobile"></el-input>
						</el-form-item>
					</el-form>	
				</span>
			  <span slot="footer" class="dialog-footer">
			    <el-button @click="editDialogVisible = false">取 消</el-button>
			    <el-button type="primary" @click="editUserInfo()">确 定</el-button>
			  </span>
			</el-dialog>
			
			<!-- 分配角色对话框 -->
			<el-dialog title="分配角色" :visible.sync="DisRoleDialogVisible"
			width="50%" @close="setRoleDialogClosed">
			  <div>
					<p>当前的用户: {{userInfo.username}}</p>
					<p>当前的角色: {{userInfo.role_name}}</p>
					<p>分配新角色:
						<el-select v-model="selectedRoleId" placeholder="请选择">
							<el-option v-for="item in rolesList" :key="item.id"
							:label="item.roleName" 
							:value="item.id">
							</el-option>
						</el-select>
					</p>
				</div>
			  <span slot="footer" class="dialog-footer">
			    <el-button @click="DisRoleDialogVisible = false">取 消</el-button>
			    <el-button type="primary" @click="saveRole">确 定</el-button>
			  </span>
			</el-dialog>
			
			<!-- 用户列表区 -->
			<el-table :data="userList" border stripe>
				<el-table-column type="index"></el-table-column>

				<el-table-column label="姓名" prop="username"></el-table-column>
				<el-table-column label="邮箱" prop="email"></el-table-column>
				<el-table-column label="电话" prop="mobile"></el-table-column>
				<el-table-column label="角色" prop="role_name"></el-table-column>

				<el-table-column label="状态">
					<!-- 作用域插槽 -->
					<template slot-scope="scope"> <!-- {{scope.row}} 获取的是对应行的数据 -->
						<el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch> 
					</template>
				</el-table-column>

				<el-table-column label="操作" width="180">
					<template slot-scope="scope">
						<!-- 数据修改按钮 -->
						<el-button type="primary" icon="el-icon-edit" 
						size="mini" @click="editClick(scope.row.id)">
						</el-button>
						<!-- 数据删除按钮 -->
						<el-button type="danger" icon="el-icon-delete" 
						size="mini" @click="removeUserInfo(scope.row.id)"></el-button>
						<!-- 分配角色按钮 -->
						<el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
							<el-button type="warning" icon="el-icon-setting" size="mini" @click="disRoles(scope.row)"></el-button>
						</el-tooltip>
					</template>
				</el-table-column>
			</el-table>

			<!-- 分页区域 -->
			<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" 
			:current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" 
			:page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
			</el-pagination>
		</el-card>
	</div>
</template>

<script>
	export default {
		name: 'Users',
		data() {
			//验证邮箱的规则
			var checkEmail = (rule, value, callback) => {
				 const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
				 if(regEmail.test(value)) {
					 return callback()
				 }
				 callback(new Error('请输入正确的邮箱'))
			};
			//验证手机规则
			var checkMobile = (rule, value, callback) => {
				 const regEmail = /^1[3|4|5|7|8]\d{9}$/
				 if(regEmail.test(value)) {
					 return callback()
				 }
				 callback(new Error('请输入正确的手机号'))
			};
			
			return {
				//获取用户列表的参数对象
				queryInfo: {
					query: '',
					pagenum: 1, //当前的页数
					pagesize: 2 //当前每页显示多少条数据
				},
				userList: [],
				total: 0,
				dialogVisible: false,
				editDialogVisible: false,
				DisRoleDialogVisible: false,
				editForm: {},
				userInfo: {},
				rolesList: [],
				//已选中的角色id
				selectedRoleId: '',
				addForm: {
					username: '',
					password: '',
					email: '',
					mobile: ''
				},
				//添加用户验证规则
				addFormRules: {
					username: [
						 {required: true, message: '请输入用户名', trigger: 'blur'}
					],
					password: [
						{required: true, message: '请输入密码', trigger: 'blur'},
						{min: 5, max: 16, message: '长度在 5 到 16 个字符', trigger: 'blur'}
					],
					email: [
						{required: true, message: '请输入邮箱', trigger: 'blur'},
						{validator: checkEmail, trigger: 'blur'}
					],
					mobile: [						
						{required: true, message: '请输入手机号', trigger: 'blur'},
						{validator: checkMobile, trigger: 'blur'}
					]
				},
				//修改用户验证规则
				editFormRules: {
					email: [
						{required: true, message: '请输入邮箱', trigger: 'blur'},
						{validator: checkEmail, trigger: 'blur'}
					],
					mobile: [
						{required: true, message: '请输入手机号', trigger: 'blur'},
						{validator: checkMobile, trigger: 'blur'}
					]
				}
			}
		},
		created() {
			this.getUserList()
		},
		methods: {
			async getUserList() {
				const {
					data: res
				} = await this.$http.get('users', {
					params: this.queryInfo
				})
				// console.log(res)
				if (res.meta.status !== 200) return res.meta.msg
				this.userList = res.data.users
				this.total = res.data.total
				// console.log(this.userList)
			},
			//监听pageSize改变的事件
			handleSizeChange(newSize) {
				this.queryInfo.pagesize = newSize
				this.getUserList()
			},
			//监听当前页码的改变
			handleCurrentChange(newPage) {
				this.queryInfo.pagenum = newPage
				this.getUserList()
			},
			//监听switch开关状态的改变
			async userStateChanged(userInfo) {
				const {data:res} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
				if(res.meta.status !== 200) {
					userInfo.ma_state = !userInfo.ma_state
					return this.$message.error('修改状态失败!')
				}
				this.$message.success('修改状态成功!')
			},
			//监听dialog对话框关闭事件
			dialogClose() {
				this.$refs.addFormRef.resetFields()
			},
			//点击确认添加用户事件
			addUser() {
				this.$refs.addFormRef.validate(async valid => {  //valid返回的是boolean值
					if(!valid) return
					//发起添加用户的网络请求
					const {data:res} = await this.$http.post('users', this.addForm)
					console.log(res)
					if(res.meta.status === 400) return this.$message.error('用户已存在!')
					if(res.meta.status !== 201) return this.$message.error('添加用户失败!')
					this.$message.success('添加用户成功!')
					//隐藏对话框
					this.dialogVisible = false
					//重新获取用户列表数据
					this.getUserList()
				})
			},
			//修改用户事件
			async editClick(id) {
				const {data:res} = await this.$http.get('users/' + id)
				if(res.meta.status !== 200) return this.$message.error('查询用户失败!')
				this.editForm = res.data
				
				this.editDialogVisible = true 
			},
			//修改用户对话框关闭事件
			editDialogClosed() {
				this.$refs.editFormRef.resetFields()
			},
			//提交用户修改信息 
			editUserInfo() {
				this.$refs.editFormRef.validate(async valid => {
					if(!valid) return
					const {data:res} = await this.$http.put('users/' + this.editForm.id, {
						email: this.editForm.email,
						mobile: this.editForm.mobile
					})
					if(res.meta.status !== 200) return this.$message.error('修改用户失败!')
					this.$message.success('修改用户成功!')
					this.editDialogVisible = false
					this.getUserList()
				})
			},
			//删除用户操作
			async removeUserInfo(id) {
				const confirmRe = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
				   confirmButtonText: '确定',
				   cancelButtonText: '取消',
				   type: 'warning'
				}).catch(err => err)
				// console.log(confirmRe)
				//用户确认删除返回‘confirm’
				if(confirmRe !== 'confirm') return this.$message.info('已取消删除')
				const {data:res} = await this.$http.delete('users/' + id)
				if(res.meta.status !== 200) return this.$message.error('删除失败')
				this.$message.success('删除成功')
				this.getUserList()
			},
			//点击分配角色对话框事件
			async disRoles(userInfo) {
				this.userInfo = userInfo
				//在展示对话框前，获取所有的角色列表
				const {data:res} = await this.$http.get('roles')
				if(res.meta.status !== 200) return this.$message.error('获取权限列表失败')
				
				this.rolesList = res.data
				this.DisRoleDialogVisible = true
			},
			//点击按钮，分配角色
			async saveRole() {
				if(!this.selectedRoleId) return this.$message.error('请选择要分配的角色')
				
				const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`, {
					rid: this.selectedRoleId
				})
				if(res.meta.status !== 200) return this.$message.error('更新角色失败')
				this.$message.success('更新角色成功')
				this.getUserList()
				this.DisRoleDialogVisible = false
			},
			//监听分配角色对话框的关闭事件
			setRoleDialogClosed() {
				this.selectedRoleId = '',
				this.userInfo = {}
			}
		}
	}
</script>

<style>
</style>
