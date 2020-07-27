<template>
	<div>
		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{path: '/home'}">首页</el-breadcrumb-item>
			<el-breadcrumb-item>权限管理</el-breadcrumb-item>
			<el-breadcrumb-item>角色列表</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图 -->
		<el-card>
			<!-- 添加角色按钮区域 -->
			<el-row>
				<el-col><el-button type="primary">添加角色</el-button></el-col>
			</el-row>

			<!-- 角色列表区域 -->
			<el-table :data="rolelist" border stripe>
				<!-- 展开列 -->
				<el-table-column type="expand">
					<template slot-scope="scope">
						<el-row v-for="(item1, i1) in scope.row.children" :key="item1.id" :class="['bdbottom', i1 == 0 ? 'bdtop' : '', 'vcenter']">
							<!-- 渲染一级权限 -->
							<el-col :span="5">
								<el-tag closable @close="removeRightById(scope.row,item1.id)">{{ item1.authName }}</el-tag>
								<i class="el-icon-caret-right"></i>
							</el-col>
							<!-- 渲染二级权限 -->
							<el-col :span="19">
								<el-row :class="[i2 == 0 ? 'bdtop' : '', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
									<el-col :span="6">
										<el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{ item2.authName }}</el-tag>
										<i class="el-icon-caret-right"></i>
									</el-col>
									<el-col :span="18">
										<el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row,item3.id)">{{ item3.authName }}</el-tag>
									</el-col>
								</el-row>
							</el-col>
						</el-row>
						<pre></pre>
					</template>
				</el-table-column>
				<!-- 索引列 -->
				<el-table-column type="index"></el-table-column>
				<el-table-column label="角色名称" prop="roleName"></el-table-column>
				<el-table-column label="角色描述" prop="roleDesc"></el-table-column>
				<el-table-column label="操作" width="300px">
					<template slot-scope="scope">
						<el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
						<el-button size="mini" type="danger" icon="el-icon-delete" @click="removeUserById(scope.row.id)">删除</el-button>
						<el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
					</template>
				</el-table-column>
			</el-table>
		</el-card>

		<!-- 分配权限的对话框 -->
		<el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
			<!-- 树形控件 -->
			<el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
			<span slot="footer" class="dialog-footer">
				<el-button @click="setRightDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="allotRights">确 定</el-button>
			</span>
		</el-dialog>

		<el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
			<el-form :model="editForm" :rules="editFormRules" ref="editFormref" label-width="70px">
				<el-form-item label="用户名" prop="roleName"><el-input v-model="editForm.roleName" disabled></el-input></el-form-item>
				<el-form-item label="角色描述" prop="roleDesc"><el-input v-model="editForm.roleDesc"></el-input></el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="editRolesInfo">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
export default {
	data() {
		return {
			// 所有角色列表数据
			rolelist: [],
			// 控制分配权限对话框的显示与隐藏
			setRightDialogVisible: false,
			// 所有权限的数据
			rightslist: [],
			// 树形控件的属性绑定对象
			treeProps: {
				label: 'authName',
				children: 'children'
			},
			// 默认选中的节点Id值数组
			defKeys: [],
			// 当前即将分配权限的角色id
			roleId: '',

			// 编辑角色
			editDialogVisible: false,
			// 查询到的用户对象
			editForm: {},
			editFormRules: {
				email: [
					{
						required: true,
						message: '请输入用户邮箱',
						trigger: 'blur'
					}
				],
				mobile: [{required: true, message: '请输入手机', trigger: 'blur'}]
			}
		}
	},
	created() {
		this.getRolesList()
	},
	methods: {
		editRolesInfo() {
			this.$refs.editFormref.validate(async valid => {
				if (!valid) return
				//发起真正的网络请求
				const {data: res} = await this.$http.put('roles/' + this.editForm.roleId, this.editForm)
				console.log(res)
				if (res.meta.status != 200) {
					this.$message.error('更新用户失败')
				}
				this.editDialogVisible = false
				this.getRolesList()
				this.$message.success('更新用户成功')
			})
		},
		async showEditDialog(id) {
			const {data: res} = await this.$http.get('roles/' + id)
			// console.log(id)
			if (res.meta.status != 200) {
				this.$message.error('查询用户失败')
			}
			// this.$message.success("用户成功")
			this.editForm = res.data
			this.editDialogVisible = true
		},
		editDialogClosed() {
			this.$refs.editFormref.resetFields()
		},
		// 获取所有角色的列表
		async getRolesList() {
			const {data: res} = await this.$http.get('roles')

			if (res.meta.status !== 200) {
				return this.$message.error('获取角色列表失败！')
			}

			this.rolelist = res.data

			console.log(this.rolelist)
		},
		async removeUserById(id) {
			// 弹框提示询问
			const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning'
			}).catch(err => {
				return err
			})
			// 如果用户确认删除,返回字符串 confirm
			// 如果用户取消删除,返回字符串 cancel
			if (confirmResult != 'confirm') {
				return this.$message.info('已取消删除')
			}
			const {data: res} = await this.$http.delete('roles/' + id)
			if (res.meta.status != 200) {
				return this.$message.error('删除用户失败')
			}
			this.$message.success('删除用户成功')
			this.getRolesList()
		},
		// 根据Id删除对应的权限
		async removeRightById(role, rightId) {
			// 弹框提示用户是否要删除
			const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning'
			}).catch(err => err)

			if (confirmResult !== 'confirm') {
				return this.$message.info('取消了删除！')
			}

			const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

			if (res.meta.status !== 200) {
				return this.$message.error('删除权限失败！')
			}

			// this.getRolesList()
			role.children = res.data
		},
		// 展示分配权限的对话框
		async showSetRightDialog(role) {
			this.roleId = role.id
			// 获取所有权限的数据
			const {data: res} = await this.$http.get('rights/tree')

			if (res.meta.status !== 200) {
				return this.$message.error('获取权限数据失败！')
			}

			// 把获取到的权限数据保存到 data 中
			this.rightslist = res.data
			console.log(this.rightslist)

			// 递归获取三级节点的Id
			this.getLeafKeys(role, this.defKeys)

			this.setRightDialogVisible = true
		},
		// 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
		getLeafKeys(node, arr) {
			// 如果当前 node 节点不包含 children 属性，则是三级节点
			if (!node.children) {
				return arr.push(node.id)
			}

			node.children.forEach(item => this.getLeafKeys(item, arr))
		},
		// 监听分配权限对话框的关闭事件
		setRightDialogClosed() {
			this.defKeys = []
		},
		// 点击为角色分配权限
		async allotRights() {
			const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]

			const idStr = keys.join(',')

			const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})

			if (res.meta.status !== 200) {
				return this.$message.error('分配权限失败！')
			}

			this.$message.success('分配权限成功！')
			this.getRolesList()
			this.setRightDialogVisible = false
		}
	}
}
</script>

<style lang="less" scoped>
.el-tag {
	margin: 7px;
}

.bdtop {
	border-top: 1px solid #eee;
}

.bdbottom {
	border-bottom: 1px solid #eee;
}

.vcenter {
	display: flex;
	align-items: center;
}
</style>
