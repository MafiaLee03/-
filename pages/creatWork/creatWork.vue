<template>
	<view>
		<button @click="open">
			<image src="../../static/uni.png"></image>
		</button>
		<uni-popup ref="popup" type="center" >
				<work></work>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
	   methods:{
		  open(){
			// 通过组件定义的ref调用uni-popup方法 ,如果传入参数 ，type 属性将失效 ，仅支持 ['top','left','bottom','right','center']
			this.$refs.popup.open()
		  }
	   }
	}
</script>

<style lang="scss">

</style>
