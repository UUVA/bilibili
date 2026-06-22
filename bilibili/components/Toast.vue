<template>
	<view v-if="visible" class="toast-container" :class="type">
		<text>{{ message }}</text>
	</view>
</template>

<script>
export default {
	props: {
		visible: Boolean,
		message: String,
		type: { type: String, default: '' } // '', 'success', 'error'
	}
}
</script>

<style scoped>
.toast-container {
	position: fixed;
	top: 150rpx;
	left: 50%;
	transform: translateX(-50%);
	background-color: rgba(24, 25, 28, 0.9);
	color: #fff;
	padding: 20rpx 40rpx;
	border-radius: 40rpx;
	font-size: 28rpx;
	z-index: 9999;
	white-space: nowrap;
	animation: fadeIn 0.3s ease-out;
}
.toast-container.success { background-color: #52c41a; }
.toast-container.error { background-color: #ff4d4f; }

@keyframes fadeIn {
	from { opacity: 0; transform: translateX(-50%) translateY(-20rpx); }
	to { opacity: 1; transform: translateX(-50%) translateY(0); }
}
</style>