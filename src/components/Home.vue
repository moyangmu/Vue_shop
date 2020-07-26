<template>

		<el-container class="home-container">
			<!-- 头部区域 -->
			<el-header>
				<div>
					<img src="../assets/logo.png" alt="" width="100px" height="60px" style="border-radius: 50%;">
					<span>电商后台管理系统</span>
				</div>
				<el-button type="info" @click="logout">退出</el-button>

			</el-header>
			<!-- 主体区域 -->
			<el-container>
				<el-aside :width="isCollapse ? '64px':'200px'">
					<!-- 侧边栏菜单区域 -->
          <div class="toggle-button" @click="toggleCollapse">|||</div>
					<el-menu

					      background-color="#111315"
					      text-color="#fff"
					      active-text-color="#0055ff"
                unique-opened
                :collapse="isCollapse"
                router
                :default-active="activePath"
          >
						  <!-- 一级菜单 -->

					      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.index">
							  <!-- 一级菜单模板区域 -->
					        <template slot="title">
								<!-- 图标 -->
					          <i :class="iconsObj[item.id]"></i>
							  <!-- 文本 -->
					          <span>{{item.authName}}</span>
					        </template>
							<!-- 二级菜单 -->
					        <el-menu-item :index="'/'+subitem.path" v-for="subitem in item.children"
							:key="subitem.id" @click="saveNavState('/'+subitem.path)"
							>
								<template slot="title">
									<!-- 图标 -->
								  <i class="el-icon-menu"></i>
								  <!-- 文本 -->
								  <span>{{subitem.authName}}</span>
								</template>
							</el-menu-item>

						  </el-submenu>



					    </el-menu>
				</el-aside>
				<el-main>

          <router-view></router-view>

        </el-main>
			</el-container>
		</el-container>

</template>

<script>
export default {
	data() {
    return {
      menulist: [],
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      // 是否折叠，false不折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath:''
    };
	},
	created(){
		this.getMenuList()
    this.activePath = window.sessionStorage.getItem("activePath");
	},
	methods: {
		logout() {
			window.sessionStorage.clear()
			this.$router.push('/login')
		},
		async getMenuList(){
			const res= await this.$http.get("menus");
			const data = res.data;
			// return res;
			console.log(data);

			if(data.meta.status != 200){
				return this.$message.error(res.meta.message);

			}
			this.menulist = res.data.data;
			// console.log(this.menulist);
		},
    // 点击按钮，切换菜单折叠和展开
    toggleCollapse(){
        this.isCollapse = !this.isCollapse;
    },
    saveNavState(activePath){
		    window.sessionStorage.setItem("activePath",activePath);
		    this.activePath = activePath;
    }
	}
}
</script>

<style lang="less" scoped>
	.el-header{
		background-color: #373d41;
		display: flex;
		justify-content: space-between;
		padding-left: 5px;
		color: #fff;
		font-size: 20px;
		div{
			display: flex;
			align-items: center;
			span{
				margin-left: 15px;
			}
		}
	}
	.el-aside{
		background-color: #333744;
    .el-menu{
      border-right: none;
    }
	}
	.el-main{
		background-color: #eaedf1;
	}
	.home-container{
		height: 100%;
	}
  .iconfont{
    margin-right: 10px;
  }
  .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
</style>
