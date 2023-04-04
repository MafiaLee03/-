<template>
	<view class="container">
		<view class="content">
			<view class="title">
				<h1>
					<span>质量中心平台聚合</span>
				</h1>
			</view>
			<uni-forms ref="form" :modelValue="formData" :rules="rules">
				<uni-forms-item class="item" label="姓名" name="username">
					<uni-easyinput class="input-group" type="text" v-model="formData.username" placeholder="请输入姓名(用于显示)" />
				</uni-forms-item>
				<uni-forms-item label="账号" name="account">
					<uni-easyinput class="input-group" type="text" v-model="formData.account" placeholder="请输入账号 如:libiqi" />
				</uni-forms-item>
				<uni-forms-item label="密码" name="password">
					<uni-easyinput class="input-group" type="text" v-model="formData.password" placeholder="请输入密码" />
				</uni-forms-item>
			</uni-forms>
			<button class="mini-btn" type="primary" @click="submit">注册</button>
			<router-link class="goIndex" to="/pages/signIn/signIn">返回登录</router-link>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
				formData: {
					username:"",
					account:"",
					password:""
					
				},
				rules:{
					username:{
						rules:[
							{
								required:true,
								errorMessage:"姓名不能为空"
							},
							{
								maxLength:5,
								errorMessage:"名字不能超过5个字"
							}
						]
					},
					account:{
						rules:[
							{
								required:true,
								errorMessage:"账号不能为空"
							},
							{
								validateFunction:function(rule,value,data,callback){
									let reg = /^[\da-z]+$/i
									if (!reg.test(value)){
										callback('账号只能为字母或数字')
									}
									return true
								}
							}
						]
					},
					password:{
						rules:[
							{
								required:true,
								errorMessage:"密码不能为空"
							},
							{
								validateFunction:function(rule,value,data,callback){
									let reg = /^(?!\D+$)(?![^a-zA-Z]+$)\S{6,20}$/
									if (!reg.test(value)){
										callback('数字、小写字母、大写字母、至少包含两种，6-20位')
									}
									return true
								}
							}
						]
					}
				}

			}
		},
		methods: {
			
			submit() {
				this.$refs.form.validate().then(res=>{
					console.log('表单数据信息：', res);
					uni.request({
						url:`http://10.89.136.99:5050/user/registered`,
						method:'POST',
						data:res,
						withCredentials: true,
						header:{
							"Content-Type": "application/x-www-form-urlencoded"
						},
						success:res=>{
							console.log(res)
							if (res.data.code != 200){
								uni.showToast({
									title:res.data.msg,
									icon:'none'
								})
							}
							else{
								uni.showToast({
									title:"注册成功",
									icon:'none'
								})
								setTimeout('uni.navigateTo({url:"/pages/signIn/signIn"})',500)
								// uni.navigateTo({
								// 	url:"/pages/signIn/signIn"
								// })
							}
						}
					})
				}).catch(err =>{
					console.log('表单错误信息：', err);
				})
			}

		}
	}
</script>

<style lang="scss">
	.container{
		    align-items: center;
		    display: flex;
		    height: 100vh;
		    justify-content: center;
		    overflow: auto;
		.content{
			background-clip: padding-box;
			background-color: hsla(0,0%,100%,.95);
			border-radius: 2.5rem;
			box-shadow: 0 0 3rem 1em hsl(0deg,0%,78% / 23%);
			color: #333;
			display: inline-block;
			margin: -5% auto 0;
			overflow: hidden;
			overflow-y: auto;
			padding: 2.5em 3.28em;
			position: relative;
			text-align: center;
			width: 34.71rem;
			z-index: 3;
			.title{
				margin-bottom: 20rpx
			};
			.input-group {
			    background-color: #f9f9f9;
			    border: 1px solid #cbd4e3;
			    border-radius: 0.286rem;
			    box-shadow: inset 0 1px 1px rgb(0,0,0 / 8%);
			    color: #222;
			    display: block;
			    display: flex;
			    font-size: inherit;
			    overflow: hidden;
			    width: 100%;
				margin-bottom: 1rem;
				position: relative;
			}
		}
	}

</style>
