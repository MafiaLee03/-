<template>
	<view class="container">
		<uni-section titleFontSize="28px" title="编辑工单" type="line">
			<view class="example">
				<uni-forms ref="form" :modelValue="formData" :rules="rules">
					<uni-forms-item label="标题" name="workordername" required>
						<uni-easyinput v-model="formData.workordername" placeholder="请输入工单标题" />
					</uni-forms-item>
					<uni-forms-item label="负责人" name="target" required>
						<uni-combox :candidates="candidates" v-model="formData.target" placeholder="请输入负责人" />
					</uni-forms-item>
					<uni-forms-item label="严重程度" name="urgency" required>
						<uni-data-checkbox :selectedColor="selectedColor" @change="changeUrgencyColor" mode="tag" v-model="formData.urgency" :localdata="level" />
					</uni-forms-item>
					<uni-forms-item label="内容描述" name="content">
						<uni-easyinput type="textarea" autoHeight v-model="formData.content" placeholder="请输入工单内容" maxlength=-1 />
					</uni-forms-item>
					<uni-forms-item label="截止日期" name="deadline">
						<uni-datetime-picker type="date" return-type="timestamp" v-model="formData.deadline"/>
					</uni-forms-item>
					<uni-forms-item label="状态" name="status">
						<uni-data-select :localdata="statusList" v-model="formData.status"></uni-data-select>
					</uni-forms-item>
					<uni-forms-item label="添加图片" name="file">
						<uni-file-picker :imageStyles="imageStyles" @select="select" @delete="delPic" :auto-upload="false" limit="9" title="选择上传图片" :value="formData.image_url_list"></uni-file-picker>
					</uni-forms-item>
				</uni-forms>
				<button type="primary" @click="submit">提交</button>
			</view>
		</uni-section>
	</view>
</template>


<script>
	export default {
		name:"work",
		props:{
			candidates:{     //所有可指派人
				type:Array,
				default:[]
			},
			// 基础表单数据
			formData: {
				type:Object,
				default(){
					return{
						workordername: '',
						target: '',
						content: '',
						urgency: 2,
						status:'',
						deadline: Date.now(),
						image_url_list:[],
						image_del_list:[]
					}
				}
			},
			//工单id
			workId:{
				type:Number,
				default:null
			},
			defaultSelectedColor:{
				type:String,
				default:"#999999"
			},
		},
		data() {
			return {
				token:'',
				files:[],
				rules:{
					workordername:{
						rules:[
							{
								required:true,
								errorMessage:"工单标题不能为空"
							}
						]
					},
					target:{
						rules:[
							{
								required:true,
								errorMessage:"负责人不能为空"
							}
						]
					}
				},
				imageStyles: {
					width: 100,
					height: 100,
				},
				// 单选数据源
				level: [{
					text: 'P3',
					value: 3
				}, {
					text: 'P2',
					value: 2
				}, {
					text: 'P1',
					value: 1
				},{
					text: 'P0',
					value: 0
				}],
				selectedColor:this.defaultSelectedColor,
				statusList:[
					{ value: 0, text: "未开始" },
					{ value: 1, text: "进行中" },
					{ value: 2, text: "已完成" },
					{ value: 3, text: "已关闭" }
				]
			}
		},
		computed: {
		},
		onLoad() {
		},
		onReady() {
		},
		methods: {
			changeUrgencyColor(e){
				console.log(e.detail.value)
				this.changeColor(e.detail.value)
			},
			changeColor(e){
				if (e == 0){
					this.selectedColor = "#E43D33"
				}
				if (e == 1){
					this.selectedColor = "#FB7433"
				}
				if (e == 3){
					this.selectedColor = "#51B52F"
				}
				if(e == 2){
					this.selectedColor = "#999999"
				}
			},
			//删除图片 删除this.files列表里当前点击X的元素
			delPic(res){
				console.log(res)
				//删除显示
				this.formData.image_url_list.forEach((item,index,arr)=>{
					if(item.uuid==res.tempFile.uuid){
						this.formData.image_url_list.splice(index,1)
						if (res.tempFile.isOld){
							this.formData.image_del_list.push(res.tempFile.uuid)
						}
					}
				})
				//删除上传列表
				if(!res.tempFile.isOld){
					this.files.forEach((item,index,arr)=>{
						if(item.uuid==res.tempFile.uuid){
							this.files.splice(index,1)
						}
					})
					}
				console.log(111111,this.files)
				console.log(222222,this.formData.image_url_list)
			},
			//选择图片this.files列表增加
			select(res){
				console.log(res)
				for (let i in res.tempFiles){
					this.files.push(res.tempFiles[i])
					let current_pic_obj = {
						extname:res.tempFiles[i].extname,
						isOld:false,
						name:res.tempFiles[i].name,
						url:res.tempFiles[i].url,
						path:res.tempFiles[i].path,
						uuid:res.tempFiles[i].uuid
					}
					this.formData.image_url_list.push(current_pic_obj)
				}
				console.log(this.files)
				console.log(this.picList)
			},
			//提交表单
			submit() {
				try {
					const value = uni.getStorageSync('token');
					if (value) {
						console.log(value);
						this.token=value
					}
				} catch (e) {
					// error
				}
				this.formData.deadline = parseInt(this.formData.deadline/1000)
				this.$refs.form.validate().then(res=>{
						console.log('表单数据信息：', this.formData)
						if(this.workId){
							//编辑工单请求
							uni.uploadFile({
								url:`http://10.89.136.99:5050/workorder/editworkorder/${this.workId}`,
								method:'POST',
								formData:this.formData,
								files:this.files,
								withCredentials: true,
								header:{
									"Authorization":`Bearer ${this.token}`,
								},
								success:res=>{
									let resJson = eval("(" + res.data + ")")
									console.log(resJson)
									if (resJson.code != 200){
										if (resJson.msg){
											uni.showToast({
												title:resJson.msg,
												icon:'none'
											})
										}
										else{
											uni.showToast({
												title:'服务器异常:' + res.data.code,
												icon:'none'
											})
										}
									}
									else{
										uni.showToast({
											title:"更新成功",
											icon:'none'
										})
										this.$emit('closePopup')
										
									}
								}
							});
						}
						else{
							//新建工单请求
							uni.uploadFile({
								url:`http://10.89.136.99:5050/workorder/foundwk`,
								method:'POST',
								formData:this.formData,
								files:this.files,
								withCredentials: true,
								header:{
									"Authorization":`Bearer ${this.token}`,
								},
								success:res=>{
									let resJson = eval("(" + res.data + ")")
									console.log('这个是返回的消息',resJson)
									if (resJson.code != 200){
										if (resJson.msg){
											uni.showToast({
												title:resJson.msg,
												icon:'none'
											})
										}
										else{
											uni.showToast({
												title:'服务器异常',
												icon:'none'
											})
										}
									}
									else{
										uni.showToast({
											title:"创建成功",
											icon:'none'
										})
										this.$emit('closePopup')
										
									}
								},
								fail:(err)=>{
									console.log('请求失败:',err)
								}
							});
							console.log(this.files)
						}
					}).catch(err =>{
						console.log('表单错误信息：', err);
					})

			}

		}
	}
</script>


<style lang="scss">

	.example {
		padding: 15px;
		background-color: #fff;
	}

	.segmented-control {
		margin-bottom: 15px;
	}

	.button-group {
		margin-top: 15px;
		display: flex;
		justify-content: space-around;
	}

	.form-item {
		display: flex;
		align-items: center;
	}

	.button {
		display: flex;
		align-items: center;
		height: 35px;
		margin-left: 10px;
	}
	// .container{
	// 	padding: 200rpx 800rpx;
	// }
</style>


