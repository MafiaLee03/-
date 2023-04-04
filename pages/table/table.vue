<template>
	<view class="box">
		<view class="uni-container">
			<uni-table ref="table" :loading="loading" border stripe type="selection" emptyText="点击下方添加任务,创建新的工单吧~" @selection-change="selectionChange">
				<uni-tr>
					<uni-th width="400" >
						<view @click="delTable()" @mouseenter="changeDelColor" @mouseleave="resetDelColor">
							<image :src="del" v-if="isSelect" style="width: 20px;height: 20px;" class="del"></image>
						</view>
						<text>标题</text>
					</uni-th>
					<uni-th width="40" align="center">严重程度</uni-th>
					<uni-th width="50" align="center">任务状态</uni-th>
					<uni-th width="50" align="center">负责人</uni-th>
					<uni-th width="70" align="center">创建日期</uni-th>
					<uni-th width="50" align="center">创建人</uni-th>
					<uni-th width="70" align="center">截止日期</uni-th>
				</uni-tr>
				<uni-tr v-for="(item, index) in tableData" :key="index" >
					<uni-td><view @click="rowclick(item)">{{item.workordername}}</view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)"><uni-tag :text="urgencytranslate[item.urgency]" circle :custom-style="urgencyTagType(item.urgency)" /></view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)">{{ statustranslate[item.status] }}</view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)">{{ item.target }}</view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)">{{ dateFormat(item.created_at,'YYYY-MM-DD') }}</view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)">{{ item.creator }}</view></uni-td>
					<uni-td align="center"><view @click="rowclick(item)"><uni-tag inverted :text="deadlineDisplay(item.deadline)" :type='deadlineColor(item.deadline)'/></view></uni-td>
					<!-- <uni-td>
						<view class="uni-group">
							<button class="uni-button" size="mini" type="primary" @click="open">修改</button>
							<button class="uni-button" size="mini" type="warn" @click="delTable(item)">删除</button>
						</view>
					</uni-td> -->
				</uni-tr>				
			</uni-table>
			<view class="new_row" @click="addNewWork" @mouseenter="changeColor" @mouseleave="resetColor">
				<image :src="addNewSrc" style="width: 32px;height: 32px;"></image>
				<text class="new_tips" :style="tipsColor">点击添加任务</text>
			</view>
			<view class="uni-pagination-box" v-if= "total>pageSize"><uni-pagination show-icon :page-size="pageSize" :current="pageCurrent" :total="total" @change="change" /></view>
			<uni-popup ref="popup" type="center" :is-mask-click="false" @maskClick="close">
					<work @closePopup="close" :candidates="candidates" :formData="formData" :workId="workId" :defaultSelectedColor="selectedColor"></work>
			</uni-popup>
		</view>
	</view>
</template>


<script>
// import tableData from './tableData.js'
export default {
	data() {
		return {
			token:'',
			candidates:'',
			searchVal: '',
			tableData: [],
			// 每页数据量
			pageSize: 18,
			// 当前页
			pageCurrent: 1,
			// 数据总量
			total: 0,
			loading: false,
			tipsColor:{
				color:'#9999'
			},
			editWK:false, //true当前打开的为编辑工单 false为新建工单
			addNewSrc:"../../static/addNew.png",
			del:"../../static/del.png",
			isSelect:false,
			formData:{
				workordername: '',
				target: '',
				content: '',
				urgency: 2,
				status:'',
				deadline: Date.now(),
				image_url_list:[],
				image_del_list:[]
			},
			workId:null,
			selectedColor:'#999999',
			statustranslate:{
				"0":"未开始",
				"1":"进行中",
				"2":"已完成",
				"3":"已关闭"
			},
			urgencytranslate:{
				"0":"P0",
				"1":"P1",
				"2":"P2",
				"3":"P3"
			}
		}
	},
	onLoad() {
		this.selectedIndexs = []
		this.getToken()
		this.getData(1)
		this.getUsers()
	},
	methods: {
		dateFormat(timestamp, format) {
						  if (String(timestamp).length === 10) {
							timestamp = timestamp * 1000
						  }
						  var date = new Date(timestamp)
						  var Y = date.getFullYear()
						  var M = date.getMonth() + 1
						  var D = date.getDate()
						  var hour = date.getHours()
						  var min = date.getMinutes()
						  var sec = date.getSeconds()
						  if (format === 'YYYY') {
							return Y // 2021
						  } else if (format === 'YYYY-MM') { // 2021-07
							return Y + '-' + (M < 10 ? '0' + M : M)
						  } else if (format === 'YYYY-MM-DD') { // 2021-07-12
							return Y + '-' + (M < 10 ? '0' + M : M) + '-' + (D < 10 ? '0' + D : D)
						  } else if (format === 'HH:mm:ss') { // 10:20:35
							return (hour < 10 ? '0' + hour : hour) + ':' + (min < 10 ? '0' + min : min) + ':' + (sec < 10 ? '0' + sec : sec)
						  } else if (format === 'YYYY-MM-DD HH:mm') { // 2021-07-12 10:20
							return Y + '-' + (M < 10 ? '0' + M : M) + '-' + (D < 10 ? '0' + D : D) + ' ' + (hour < 10 ? '0' + hour : hour) + ':' + (min < 10 ? '0' + min : min)
						  } else if (format === 'YYYY-MM-DD HH:mm:ss') { // 2021-07-12 10:20:35
							return Y + '-' + (M < 10 ? '0' + M : M) + '-' + (D < 10 ? '0' + D : D) + ' ' + (hour < 10 ? '0' + hour : hour) + ':' + (min < 10 ? '0' + min : min) + ':' + (sec < 10 ? '0' + sec : sec)
						  } else {
							return '--'
						  }
					},
		//严重程度标签判断
		urgencyTagType(e){
			if (e=='0'){
				return "background-color: #E43D33; border-color: #E43D33; color: #fff;"
			}
			if (e=='1'){
				return "background-color: #FB7433; border-color: #FB7433; color: #fff;"
			}
			if (e=='2'){
				return "background-color: #999999; border-color: #999999; color: #fff;"
			}
			if (e=='3'){
				return "background-color: #51B52F; border-color: #51B52F; color: #fff;"
			}
		},
		
		// 多选处理
		selectedItems() {
			return this.selectedIndexs.map(i => this.tableData[i])
		},
		// 多选
		selectionChange(e) {
			console.log(e.detail.index)
			if(e.detail.index.length != 0){
				this.isSelect = true
			}
			else{
				this.isSelect = false
			}
			this.selectedIndexs = e.detail.index
		},
		//删除选择的工单，如果没选删除点击的
		delTable() {
			uni.showModal({
				title: '删除工单',
				content: '工单删除后无法恢复,请确认后再进行删除',
				success: res => {
					if (res.confirm) {
						if(this.selectedItems().length){
							this.deleteWork(this.selectedItems())
						}
					} else if (res.cancel) {
						console.log('用户点击取消');
					}
				}
			});
			
		},
		//截止日期判断是今天 明天 已过期 其他
		deadlineDisplay(e){
			let todayStartTimestamp = Math.floor(new Date(new Date().setHours(0, 0, 0, 0)).getTime() / 1000) //今天开始时间戳
			let todayEndTimestamp = Math.floor(new Date(new Date().setHours(23, 59, 59, 999)).getTime() / 1000) //今天结束时间戳
			let tomorrowStartTimestamp = todayStartTimestamp + 86400
			let tomorrowEndTimestamp = todayEndTimestamp + 86400
			if (e<todayEndTimestamp&&e>=todayStartTimestamp){
				return '今天'
			}
			if (e<tomorrowEndTimestamp&&e>=tomorrowStartTimestamp){
				return '明天'
			}
			if (e>=tomorrowEndTimestamp){ //截止日期大于1天
				return this.dateFormat(e,'YYYY-MM-DD')
			}
			if (e<todayStartTimestamp){ //过期
				return this.dateFormat(e,'YYYY-MM-DD')
			}
		},
		//截止日期颜色显示
		deadlineColor(e){
			let todayStartTimestamp = Math.floor(new Date(new Date().setHours(0, 0, 0, 0)).getTime() / 1000) //今天开始时间戳
			let todayEndTimestamp = Math.floor(new Date(new Date().setHours(23, 59, 59, 999)).getTime() / 1000) //今天结束时间戳
			let tomorrowStartTimestamp = todayStartTimestamp + 86400
			let tomorrowEndTimestamp = todayEndTimestamp + 86400
			if (e<todayEndTimestamp&&e>=todayStartTimestamp){
				return 'warning'
			}
			if (e<tomorrowEndTimestamp&&e>=tomorrowStartTimestamp){
				return 'success'
			}
			if (e>=tomorrowEndTimestamp){ //截止日期大于1天
				return 'success'
			}
			if (e<todayStartTimestamp){ //过期
				return 'error'
			}
		},
		//用于设置初始化严重程度tag颜色
		changeUrgencyColor(e){
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
		//点击编辑工单
		rowclick(res){
			console.log(res)
			this.resetFormData()
			this.formData.workordername=res.workordername
			this.formData.target=res.target
			this.formData.content=res.content
			this.formData.deadline=Number(res.deadline)*1000
			this.formData.status=Number(res.status)
			this.formData.urgency=Number(res.urgency)
			this.changeUrgencyColor(Number(res.urgency))
			let image_url_obj = {}
			image_url_obj = eval("(" + res.image_url_list + ")")
			let image_url = {}
			for (let key in image_url_obj){
				image_url = image_url_obj[key]
				image_url.url = 'http://10.89.136.99:5050/' + image_url.url
				image_url['uuid'] = Number(key) //给表单上已有的图片加一个uuid
				image_url['isOld'] = true //标记已有图片为旧图片
				this.formData.image_url_list.push(image_url)
			}
			console.log(this.formData.image_url_list)
			this.workId=res._id
			this.open()
		},
		//批量删除工单
		deleteWork(workId){
			for(let i of workId){
				let selet_id = i._id //获取表单id
				console.log('删除工单id',selet_id)
				uni.request({
					url:`http://10.89.136.99:5050/workorder/deletewk`,
					method:'GET',
					withCredentials: true,
					data:{
						wkid:selet_id,
					},
					header:{
						"Content-Type": "application/x-www-form-urlencoded",
						"Authorization":`Bearer ${this.token}`
					},
					success:res=> {
						if (res.data.code != 200){
							uni.showToast({
								title:res.data.msg,
								icon:'none'
							})
						}
						else{
							this.$refs.table.clearSelection()
							this.getData(this.pageCurrent)
						}
					}
				})
			}
		},
		// 分页触发
		change(e) {
			this.$refs.table.clearSelection()
			this.selectedIndexs.length = 0
			this.getData(e.current)
		},
		// 搜索
		search() {
			this.getData(1, this.searchVal)
		},
		// 获取分页工单数据
		getData(pageCurrent, value = '') {
			this.loading = true
			this.pageCurrent = pageCurrent
			// this.request({
			// 	pageSize: this.pageSize,
			// 	pageCurrent: pageCurrent,
			// 	value: value,
			// 	success: res => {
			// 		// console.log('data', res);
			// 		this.tableData = res.data
			// 		this.total = res.total
			// 		this.loading = false
			// 	}
			// })
			uni.request({
				url:`http://10.89.136.99:5050/workorder/getallworkorder`,
				method:'GET',
				withCredentials: true,
				data:{
					page:pageCurrent,
					per_page:this.pageSize
				},
				header:{
					"Content-Type": "application/x-www-form-urlencoded",
					"Authorization":`Bearer ${this.token}`
				},
				success:res=>{
					if (res.data.code != 200){
						uni.showToast({
							title:res.data.msg,
							icon:'none'
						})
					}
					else{
						this.tableData=res.data.workorders
						console.log(res.data.workorders)
						this.total=res.data.pagination.total_items
						this.loading = false
					}
				}
			})
		},
		//重置formData
		resetFormData(){
			this.formData={
				workordername: '',
				target: '',
				content: '',
				urgency: 2,
				status:'',
				deadline: Date.now(),
				image_url_list:[],
				image_del_list:[]
			}
			this.workId=null
		},
		// 打开编辑工单
		open(){
			// 通过组件定义的ref调用uni-popup方法 ,如果传入参数 ，type 属性将失效 ，仅支持 ['top','left','bottom','right','center']
			this.editWK = true
			this.$refs.popup.open()
		},
		//创建新工单
		addNewWork(){
			// this.resetFormData()
			this.formData.urgency = 2
			this.selectedColor = "#999999"
			this.formData.target = ''
			this.editWK = false
			this.$refs.popup.open()
		},
		//关闭编辑工单界面
		close() {
			this.$refs.popup.close()
			if (this.editWK){
				this.resetFormData()
				this.editWK = false
			}
			this.getData(this.pageCurrent)
		},
		//悬停改变加号及添加文本颜色
		changeColor(){
			this.tipsColor={
				color:'#606266'
			}
			this.addNewSrc="../../static/addNewS.png"
		},
		//移动重置颜色
		resetColor(){
			this.tipsColor={
				color:'#9999'
			}
			this.addNewSrc="../../static/addNew.png"
		},
		//悬停改变删除按钮颜色
		changeDelColor(){
			this.del="../../static/dels.png"
		},
		//移动重置删除按钮颜色
		resetDelColor(){
			this.del="../../static/del.png"
		},
		// 伪request请求
		request(options) {
			const { pageSize, pageCurrent, success, value } = options
			let total = tableData.length
			let data = tableData.filter((item, index) => {
				const idx = index - (pageCurrent - 1) * pageSize
				return idx < pageSize && idx >= 0
			})
			if (value) {
				data = []
				tableData.forEach(item => {
					if (item.name.indexOf(value) !== -1) {
						data.push(item)
					}
				})
				total = data.length
			}

			setTimeout(() => {
				typeof success === 'function' &&
					success({
						data: data,
						total: total
					})
			}, 500)
		},
		getToken(){
			try {
				const value = uni.getStorageSync('token');
				if (value) {
					console.log(value);
					this.token=value
				}
			} catch (e) {
				console.log(e)
			}
		},
		//获取所有用户
		getUsers(){
			uni.request({
				url:`http://10.89.136.99:5050/user/getusers`,
				method:'GET',
				withCredentials: true,
				header:{
					"Content-Type": "application/x-www-form-urlencoded",
					"Authorization":`Bearer ${this.token}`
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
						let users=[]
						for(let i of res.data.data){
							users.push(i.username)
						}
						this.candidates=users
					}
				}
			})
		}
	}
}
</script>


<style lang="scss">
/* #ifndef H5 */
/* page {
	padding-top: 85px;
} */
/* #endif */
.uni-group {
	display: flex;
	align-items: center;
};
.uni-container{
	// .uni-pagination-box{
	// 	position:fixed;
	// 	bottom: 10%;
	// 	left: 40%;
	// }
}
// .new_row{
// 	display: flex;
// 	padding: 50;
// }
.new_row{
	padding-left: 25px;
	display: flex;
	border: 2px dashed #aaaa;
	margin: 0 5px;
	align-items: center;
	.new_tips{
		color: #9999;
	}
}

.del{
	margin-right: 50rpx;
}

/deep/.uni-popup .uni-popup__wrapper {
    width: 40% !important;
	height: 80%;
    padding: 40rpx;
}
</style>


