<template>
	<div>
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{path: '/home'}">首页</el-breadcrumb-item>
			<el-breadcrumb-item>用户管理</el-breadcrumb-item>
			<el-breadcrumb-item>用户列表</el-breadcrumb-item>
		</el-breadcrumb>

		<!--  卡片视图区域-->
		<el-card class="box-card">
			<el-row :gutter="20">
				<el-col :span="8">
					<el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
						<el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
					</el-input>
				</el-col>
				<el-col :span="4"><el-button type="primary" @click="addDialogVisible = true">添加用户</el-button></el-col>
			</el-row>

			<el-table :data="userlist" border stripe="">
				<el-table-column type="index"></el-table-column>
				<el-table-column label="姓名" prop="username"></el-table-column>
				<el-table-column label="邮箱" prop="email"></el-table-column>
				<el-table-column label="电话" prop="mobile"></el-table-column>
				<el-table-column label="角色" prop="role_name"></el-table-column>
				<el-table-column label="状态" prop="mg_state">
					<template slot-scope="scope">
						<!-- {{scope.row}} -->
						<el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
					</template>
				</el-table-column>

				<el-table-column label="操作">
					<template slot-scope="scope">
						<!-- 修改按钮 -->
						<el-button type="primary" icon="el-icon-edit" circle size="mini" @click="showEditDialog(scope.row.id)"></el-button>
						<!-- 删除按钮 -->
						<el-button type="danger" icon="el-icon-delete" circle size="mini" @click="removeUserById(scope.row.id)"></el-button>
						
						<el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
							<el-button type="warning" icon="el-icon-setting" circle size="mini"></el-button>
						</el-tooltip>
					</template>
				</el-table-column>
			</el-table>
			<el-pagination
				@size-change="handleSizeChange"
				@current-change="handleCurrentChange"
				:current-page="queryInfo.pagenum"
				:page-sizes="[1, 2, 3, 10]"
				:page-size="queryInfo.pagesize"
				layout="total, sizes, prev, pager, next, jumper"
				:total="total"
			></el-pagination>
		</el-card>
		<!-- 添加用户对话框 -->
		<el-dialog title="添加用户" :visible.sync="addDialogVisible" width="30%" @close="addDiaglogClosed">
			<!-- 内容主体区域 -->
			<el-form :model="addForm" :rules="addFormRules" ref="addFormref" label-width="120px">
				<el-form-item label="用户名" prop="username"><el-input v-model="addForm.username"></el-input></el-form-item>
				<el-form-item label="密码" prop="password"><el-input v-model="addForm.password"></el-input></el-form-item>
				<el-form-item label="邮箱" prop="email"><el-input v-model="addForm.email"></el-input></el-form-item>
				<el-form-item label="手机" prop="mobile"><el-input v-model="addForm.mobile"></el-input></el-form-item>
				
				
			</el-form>
			
			<!-- 底部区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="addDialogVisible = false">取 消</el-button>

				<el-button type="primary" @click="addUser">确 定</el-button>
			</span>
		</el-dialog>
		
		<!-- 修改用户对话框 -->
		<el-dialog
		  title="修改用户"
		  :visible.sync="editDialogVisible"
		  width="50%"
		  @close="editDialogClosed"
		  >
		  <el-form :model="editForm" :rules="editFormRules" ref="editFormref" label-width="70px">
		    <el-form-item label="用户名">
		      <el-input v-model="editForm.username" disabled></el-input>
		    </el-form-item>
			<el-form-item label="邮箱" prop="email">
			  <el-input v-model="editForm.email" ></el-input>
			</el-form-item>
			<el-form-item label="手机号" prop="mobile">
			  <el-input v-model="editForm.mobile" ></el-input>
			</el-form-item>
			</el-form>
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="editDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="editUserInfo">确 定</el-button>
		  </span>
		</el-dialog>
	</div>
</template>

<script>
export default {
	data() {
		var checkemail = (rule,value,cb)=>{
			// 验证邮箱的正则表达式
			const regEmail =/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
			if(regEmail.test(value)){
				return cb()
			}
			cb(new Error("请输入合法的邮箱"))
		}
		var checkmobile = (rule,value,cb)=>{
			// 验证手机号的正则表达式
			const regMobile =/^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
			if(regMobile.test(value)){
				return cb()
			}
			cb(new Error("请输入合法的手机号"))
		}
		return {
			queryInfo: {
				query: '',
				pagenum: 1,
				// 当前页数
				pagesize: 2
			},
			userlist: [],
			total: 0,
			addDialogVisible: false,
			addForm: {
				username: '',
				password: '',
				email: ''
				
			},
			addFormRules: {
				username: [{required: true, message: '请输入用户名', trigger: 'blur'}, 
				{min: 3, max: 10, message: '用户名长度在3到10个字符之间', trigger: 'blur'}],
				password: [{required: true, message: '请输入密码', trigger: 'blur'}, 
				{min: 6, max: 16, message: '用户名长度在6到16个字符之间', trigger: 'blur'}],
				email:[{required: true, message: '请输入邮箱', trigger: 'blur'},
					{ validator: checkemail,trigger:'blur' }
				],
				mobile:[{required: true, message: '请输入手机', trigger: 'blur'},
				{ validator: checkmobile,trigger:'blur' }]
			},
			editDialogVisible : false,
			// 查询到的用户对象
			editForm:{},
			editFormRules:{
				email:[{
					required: true, message: '请输入用户邮箱', trigger: 'blur'
				},{ validator: checkemail,trigger:'blur' }
				],
				mobile:[{required: true, message: '请输入手机', trigger: 'blur'},
				{ validator: checkmobile,trigger:'blur' }]
				
					
				
			}
		}
	},
	created() {
		this.getUserList()
	},
	methods: {
		async getUserList() {
			const {data: res} = await this.$http.get('users', {params: this.queryInfo})
			console.log('111111')
			if (res.meta.status != 200) {
				return this.$message.error('获取用户列表失败')
			}
			this.userlist = res.data.users
			this.total = res.data.total
			console.log(res)
		},
		// 监听pageSize改变
		handleSizeChange(newSize) {
			this.queryInfo.pagesize = newSize
			this.getUserList()
		},
		// 监听页码值改变
		handleCurrentChange(newPage) {
			this.queryInfo.pagenum = newPage
			this.getUserList()
		},
		async userStateChanged(userinfo) {
			// console.log(userinfo)
			const {data: res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
			if (res.meta.status != 200) {
				userinfo.mg_state = !userinfo.mg_state
				return this.$message.error('更新用户状态失败')
			}
			this.$message.success('更新成功')
		},
		addDiaglogClosed(){
			this.$refs.addFormref.resetFields();
		},
		addUser(){
			this.$refs.addFormref.validate(async valid =>{
				if(!valid)
				return
				//发起真正的网络请求
				const {data:res} = await this.$http.post('users',this.addForm);
				if(res.meta.status != 201){
					this.$message.error("添加用户失败")
					
				}
					this.$message.success("添加用户成功")				
					this.addDialogVisible = false;
					this.getUserList()
			})
		},
		async showEditDialog(id){

			const {data: res} = await this.$http.get('users/'+id)
			// console.log(id)
			if(res.meta.status != 200){
				this.$message.error("查询用户失败")
				
			}
				// this.$message.success("用户成功")				
				this.editForm = res.data;
				this.editDialogVisible = true;
		},
		// 监听修改用户对话框的关闭事件
		editDialogClosed(){
			this.$refs.editFormref.resetFields();
			
		},
		// 修改用户信息提交
		editUserInfo(){
			this.$refs.editFormref.validate(async valid =>{
				if(!valid)
				return
				//发起真正的网络请求
				const {data:res} = await this.$http.put('users/' + this.editForm.id,this.editForm);
				if(res.meta.status != 200){
					this.$message.error("更新用户失败")
					
				}
				this.editDialogVisible = false;
					this.getUserList()
					this.$message.success("更新用户成功")
					
			})
		},
		// 根据id删除对应用户信息
		async removeUserById(id){
			// 弹框提示询问
			 const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
			          confirmButtonText: '确定',
			          cancelButtonText: '取消',
			          type: 'warning'
			        }).catch(err =>{
						return err
					})
					// 如果用户确认删除,返回字符串 confirm
					// 如果用户取消删除,返回字符串 cancel
					if(confirmResult != 'confirm'){
						return this.$message.info('已取消删除')
					}
					const {data:res} =  await this.$http.delete('users/'+id)
					if(res.meta.status != 200 ){
						return this.$message.error("删除用户失败")
					}
					this.$message.success("删除用户成功")
					this.getUserList()
		}
	}
}
</script>

<style scoped lang="less"></style>
