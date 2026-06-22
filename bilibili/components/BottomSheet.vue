<template>
	<view>
		<!-- 遮罩层 -->
		<view v-if="show" class="overlay" @click="$emit('close')" @touchmove.stop.prevent></view>
		
		<!-- 菜单内容 -->
		<view v-if="show" class="bottom-sheet" @touchmove.stop.prevent>
			<view class="handle"></view>
			<view class="menu-list">
				<view 
					v-for="(item, index) in menuItems" 
					:key="index" 
					class="menu-item" 
					@click="handleClick(item)"
				>
					<text class="menu-text">{{ item.text }}</text>
					<text class="arrow">›</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	props: { show: Boolean },
	data() {
		return {
			menuItems: [
				{ action: 'settings', text: '消息设置' },
				{ action: 'fanclub', text: '应援团消息助手' },
				{ action: 'contacts', text: '通讯录' },
				{ action: 'antiharass', text: '防骚扰和互动人群设置' }
			]
		}
	},
	methods: {
		handleClick(item) {
			this.$emit('select', item.action)
			this.$emit('close')
		}
	}
}
</script>

<style scoped>
.overlay {
	position: fixed; top: 0; left: 0; right: 0; bottom: 0;
	background-color: rgba(0, 0, 0, 0.4); z-index: 100;
}
.bottom-sheet {
	position: fixed; bottom: 0; left: 0; right: 0;
	background-color: #fff; border-radius: 32rpx 32rpx 0 0;
	z-index: 101; padding-bottom: env(safe-area-inset-bottom);
	animation: slideUp 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}
.handle {
	width: 72rpx; height: 8rpx; background-color: #d0d0d0;
	border-radius: 4rpx; margin: 20rpx auto 30rpx;
}
.menu-item {
	display: flex; justify-content: space-between; align-items: center;
	padding: 32rpx 40rpx; border-bottom: 1rpx solid #f1f2f3;
}
.menu-item:last-child { border-bottom: none; }
.menu-text { font-size: 30rpx; color: #18191c; font-family: 'STKaiti', serif; }
.arrow { font-size: 36rpx; color: #c0c4cc; }

@keyframes slideUp {
	from { transform: translateY(100%); }
	to { transform: translateY(0); }
}
</style>