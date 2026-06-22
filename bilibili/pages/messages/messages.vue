<template>
	<view class="msg-page">
		<!-- 自定义头部 -->
		<view class="page-header">
			<view class="header-title">消息</view>
			<view class="header-right">
				<view class="icon-btn" @click="clearAllUnread">
					<svg class="icon-svg" viewBox="0 0 1024 1024" width="44" height="44">
						<path
							d="M512 82c64.518 0 116.942 51.84 117.984 116.186l0.016 1.953v149.176h202c34.993 0 63.426 28.117 63.991 63.016l0.009 1.06v72.085c0 35.034-28.084 63.5-62.942 64.067l-1.058 0.008h-20.625l48.383 306.367c5.465 34.604-17.834 67.113-52.176 73.122l-1.044 0.174c-2.976 0.47-5.98 0.73-8.991 0.778l-1.004 0.008H227.457c-35.346 0-64-28.688-64-64.076 0-3.015 0.213-6.027 0.636-9.012l0.15-0.994 48.381-306.367H192c-34.993 0-63.426-28.116-63.991-63.016l-0.009-1.06v-72.084c0-35.034 28.084-63.501 62.942-64.067l1.058-0.009h202V200.139c0-64.594 51.78-117.08 116.049-118.123L512 82z m234.58 467.551H277.42l-49.963 316.373h117.15l28.621-180.913c3.106-19.631 21.538-33.028 41.17-29.922 0.004 0 0.008 0 0.012 0.002 19.64 3.114 33.039 21.556 29.932 41.197l-26.837 169.636h379.038L746.58 549.551zM394 413.39H192v72.086h640V413.39l-202-0.001v1.002H394v-1.002z m118-267.314c-29.525 0-53.516 23.723-53.993 53.17l-0.007 0.893v149.176h108V200.139c0-29.264-23.224-53.098-52.226-54.035l-0.881-0.021-0.893-0.007z"
							fill="currentColor" />
					</svg>
				</view>

				<view class="icon-btn" @click="showSheet = true">
					<svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
						<path
							d="M512 256a64 64 0 1 0-64-64 64.1 64.1 0 0 0 64 64z m0 192a64 64 0 1 0 64 64 64.1 64.1 0 0 0-64-64z m0 320a64 64 0 1 0 64 64 64.1 64.1 0 0 0-64-64z"
							fill="currentColor" />
					</svg>
				</view>
			</view>
		</view>

		<!-- 快捷入口 -->
		<view class="quick-entries">
			<view class="entry" @click="switchTab('reply')">
				<view class="entry-icon green">
					<svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
						<path
							d="M799.451429 654.872381a510.293333 510.293333 0 0 1-43.641905-73.142857 856.990476 856.990476 0 0 1-20.72381-180.175238c0-95.329524-73.142857-157.500952-157.744762-174.08a383.512381 383.512381 0 0 0-128.731428 0c-85.089524 14.384762-157.744762 78.750476-157.744762 174.08a837.241905 837.241905 0 0 1-20.72381 178.224762 363.032381 363.032381 0 0 1-43.641904 71.192381C197.485714 692.419048 182.857143 715.093333 182.857143 731.428571c2.194286 33.158095 27.062857 48.761905 68.510476 48.761905h31.207619c116.297143 1.950476 346.697143 4.144762 460.8 0h31.207619c48.761905 0 66.31619-24.380952 66.316191-48.761905 2.194286-14.140952-14.384762-37.059048-41.447619-76.55619zM512 926.47619a147.992381 147.992381 0 0 0 143.36-87.04H369.615238A150.186667 150.186667 0 0 0 512 926.47619z m70.704762-752.15238h-140.434286v-2.194286a70.704762 70.704762 0 1 1 141.165714 2.194286z"
							fill="currentColor" />
					</svg>
				</view>
				<text class="entry-text">回复与@</text>
			</view>
			<view class="entry" @click="switchTab('like')">
				<view class="entry-icon pink">
					<svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
						<path
							d="M727.771429 909.061224h-181.812245c-95.608163 0-179.2-38.661224-208.457143-53.812244h-67.918368c-25.6 0-47.020408-20.897959-47.020408-47.020409V431.020408c0-25.6 20.897959-47.020408 47.020408-47.020408h72.09796c12.538776-4.702041 57.469388-22.987755 78.367347-59.036735 7.836735-13.061224 14.628571-42.318367 21.420408-68.963265 11.493878-47.020408 22.465306-91.428571 48.065306-117.55102 23.510204-24.032653 65.306122-31.346939 104.489796-17.763266 50.677551 17.240816 85.159184 62.171429 92.473469 120.163266 6.791837 56.946939-2.612245 110.236735-6.791837 130.612244-1.044898 4.179592 1.044898 6.791837 2.089796 8.359184 1.044898 1.567347 3.657143 3.657143 7.836735 3.657143h119.640816c46.497959 0 100.310204 44.408163 100.310204 101.355102 0 26.122449-39.706122 331.232653-63.216326 369.893878-26.122449 43.363265-88.816327 54.334694-118.595918 54.334693zM134.791837 847.412245c-10.971429 0-19.853061-8.881633-19.853061-19.853061V420.571429c0-10.971429 8.881633-19.853061 19.853061-19.853062s19.853061 8.881633 19.853061 19.853062v406.987755c0 10.971429-8.881633 19.853061-19.853061 19.853061z"
							fill="currentColor" />
					</svg>
				</view>
				<text class="entry-text">收到喜欢</text>
			</view>
			<view class="entry" @click="switchTab('follow')">
				<view class="entry-icon blue">
					<svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
						<path
							d="M898.142435 603.91467a117.623892 117.623892 0 0 0-96.78766 53.602888 115.439449 115.439449 0 0 0-97.459796-53.602888 96.451592 96.451592 0 0 0-75.783394 33.606827 175.427634 175.427634 0 0 0-36.127338 104.853298c0 6.217263-19.323925 154.591401 197.776173 278.264522a24.196915 24.196915 0 0 0 11.762389 3.192649 23.524778 23.524778 0 0 0 12.266492-3.528717C945.023958 940.991139 1013.41385 844.371513 1011.229406 739.350181c0.168034-54.443059-29.742041-135.43551-113.086971-135.435511zM481.92189 0a308.6787 308.6787 0 0 0-136.947817 582.74237C165.177552 641.218247 16.299311 805.555628 12.60256 974.597965v8.065639A27.557598 27.557598 0 0 0 39.992123 1008.204792 27.893666 27.893666 0 0 0 67.717755 980.311126c0-180.468658 209.03446-364.129964 414.204135-364.129964A306.326223 306.326223 0 0 0 785.559567 308.006564 306.326223 306.326223 0 0 0 481.92189 0z m-3.360682 555.52084A241.297013 241.297013 0 0 1 238.944536 312.543485a239.616672 239.616672 0 1 1 479.401378 0 241.128979 241.128979 0 0 1-239.784706 242.977355z"
							fill="currentColor" />
					</svg>
				</view>
				<text class="entry-text">新增粉丝</text>
			</view>
		</view>

		<!-- 消息列表 -->
		<scroll-view scroll-y class="msg-list" enable-back-to-top>
			<view v-if="filteredMsgs.length === 0" class="empty-state">
				<text class="empty-icon">📭</text>
				<text class="empty-text">暂无消息</text>
			</view>

			<!-- 消息列表项 -->
			<view v-for="msg in filteredMsgs" :key="msg.id" class="msg-item" @click="openChat(msg)">
				<!-- 头像区域 - 带兜底处理 -->
				<view class="avatar-wrapper">
					<image class="avatar" :src="msg.avatar" mode="aspectFill" @error="handleImageError(msg)" />
					<!-- 图片加载失败时显示的占位符 -->
					<view v-if="msg.loadFailed" class="avatar-placeholder"
						:style="{ background: msg.placeholderColor }">
						<text>{{ msg.title.charAt(0) }}</text>
					</view>
				</view>

				<!-- 消息内容 -->
				<view class="msg-body">
					<view class="msg-header">
						<view class="msg-title">
							<text>{{ msg.title }}</text>
							<text v-if="msg.isLive" class="live-tag">直播中</text>
						</view>
						<text class="msg-time">{{ msg.time }}</text>
					</view>
					<text class="msg-preview" :class="{ 'auto-reply': msg.preview.includes('自动回复') }">
						{{ msg.preview }}
					</text>
				</view>

				<!-- 未读红点 -->
				<view v-if="msg.unreadCount > 0" class="badge">
					<text>{{ msg.unreadCount > 99 ? '99+' : msg.unreadCount }}</text>
				</view>
			</view>
		</scroll-view>

		<!-- 底部菜单遮罩 -->
		<view v-if="showSheet" class="overlay" @click="showSheet = false"></view>
		<view v-if="showSheet" class="bottom-sheet">
			<view class="handle"></view>
			<view class="menu-list">
				<view class="menu-item" @click="onMenuSelect('settings')"><text class="menu-text">消息设置</text></view>
				<view class="menu-item" @click="onMenuSelect('fanclub')"><text class="menu-text">应援团消息助手</text></view>
				<view class="menu-item" @click="onMenuSelect('contacts')"><text class="menu-text">通讯录</text></view>
				<view class="menu-item" @click="onMenuSelect('antiharass')"><text class="menu-text">防骚扰和互动人群设置</text>
				</view>
			</view>
		</view>

		<!-- Toast 提示 -->
		<view v-if="toastVisible" class="toast" :class="toastType">
			<text>{{ toastMsg }}</text>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentTab: 'reply',
			showSheet: false,
			toastVisible: false,
			toastMsg: '',
			toastType: '',
			msgData: [
				{ id: 1, type: 'system', avatar: '/static/images/bilibili.jpg', title: '哔哩哔哩智能机', preview: '登录操作通知', time: '3月25日', unreadCount: 3, isLive: false, placeholderColor: '#e8f7f3' },
				{ id: 2, type: 'system', avatar: '/static/images/system.jpg', title: '系统通知', preview: '[4条] 《哔哩哔哩隐私政策》修订通知', time: '3月16日', unreadCount: 4, isLive: false, placeholderColor: '#e3f2fd' },
				{ id: 3, type: 'reply', avatar: '/static/images/pr.jpg', title: 'pr免费版怎么下载', preview: '[自动回复] LR软件安装包 下载链接...', time: '2025年10月21日', unreadCount: 0, isLive: false, placeholderColor: '#fce4ec' },
				{ id: 4, type: 'reply', avatar: '/static/images/kk.jpg', title: '框框老师课堂', preview: '[自动回复] 感谢您的关注', time: '2024年11月7日', unreadCount: 0, isLive: false, placeholderColor: '#fff3e0' },
				{ id: 5, type: 'reply', avatar: '/static/images/ted.jpg', title: 'TED官方英语', preview: '[自动回复] 【粉丝福利：领优惠券】', time: '2024年9月13日', unreadCount: 1, isLive: true, placeholderColor: '#f3e5f5' },
				{ id: 6, type: 'reply', avatar: '/static/images/cad.jpg', title: 'cad教程零基础入门', preview: '[自动回复] 因B站上无法直接发下载链...', time: '2024年9月6日', unreadCount: 0, isLive: false, placeholderColor: '#e0f2f1' },
				{ id: 7, type: 'reply', avatar: '/static/images/stranger.jpg', title: '陌生人消息', preview: '暂无新的陌生人消息', time: '2024年8月11日', unreadCount: 0, isLive: false, placeholderColor: '#f1f2f3' },
				{ id: 8, type: 'reply', avatar: '/static/images/heima.jpg', title: '黑马程序员', preview: '[自动回复] 感谢关注黑马程序员😁', time: '2024年8月1日', unreadCount: 0, isLive: false, placeholderColor: '#e8f5e9' },
				{ id: 9, type: 'reply', avatar: '/static/images/songhao.jpg', title: '宋浩老师官方', preview: '哈喽，正在备考25考研的同学...', time: '2023年11月10日', unreadCount: 2, isLive: true, placeholderColor: '#fff8e1' }
			]
		}
	},
	computed: {
		filteredMsgs() {
			if (this.currentTab === 'reply') return this.msgData.filter(m => m.type === 'reply' || m.type === 'system')
			if (this.currentTab === 'like') return this.msgData.filter(m => m.type === 'reply').map(m => ({ ...m, preview: '点赞了你的动态' }))
			if (this.currentTab === 'follow') return this.msgData.filter(m => m.type === 'reply').map(m => ({ ...m, preview: '关注了你' }))
			return this.msgData
		}
	},
	methods: {
		handleImageError(msg) {
			this.$set(msg, 'loadFailed', true)
		},
		switchTab(tab) {
			this.currentTab = tab
			const names = { reply: '回复与@', like: '收到喜欢', follow: '新增粉丝' }
			this.showToast('切换到：' + names[tab])
		},
		clearAllUnread() {
			let count = 0
			this.msgData.forEach(m => {
				if (m.unreadCount > 0) {
					count += m.unreadCount
					m.unreadCount = 0
				}
			})
			this.showToast(count > 0 ? `已清除 ${count} 条未读` : '没有未读消息', count > 0 ? 'success' : '')
		},
		openChat(msg) {
			if (msg.unreadCount > 0) msg.unreadCount = 0;
			// 跳转到 chat 页面，传递 id 参数
			uni.navigateTo({
				url: `/pages/chat/chat?id=${msg.id}`
			});
		},
		onMenuSelect(action) {
			const map = { settings: '消息设置', fanclub: '应援团助手', contacts: '通讯录', antiharass: '防骚扰设置' }
			this.showSheet = false
			this.showToast('打开：' + (map[action] || ''))
		},
		showToast(msg, type = '') {
			this.toastMsg = msg
			this.toastType = type
			this.toastVisible = true
			setTimeout(() => this.toastVisible = false, 2000)
		}
	}
}
</script>

<style scoped>
/* 页面基础样式 */
.msg-page {
	height: 100vh;
	background-color: #f1f2f3;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	padding-bottom: 100rpx;
	box-sizing: border-box;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

/* 头部样式 */
.page-header {
	height: 96rpx;
	background: #fff;
	display: flex;
	align-items: center;
	justify-content: center;
	position: relative;
	border-bottom: 1rpx solid #e3e5e7;
}

.header-title {
	font-size: 36rpx;
	font-weight: 600;
	color: #18191c;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

.header-right {
	position: absolute;
	right: 32rpx;
	display: flex;
	gap: 24rpx;
}

.icon-btn {
	width: 48rpx;
	height: 48rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.icon {
	font-size: 40rpx;
	color: #18191c;
}

/* 快捷入口样式 */
.quick-entries {
	display: flex;
	justify-content: space-around;
	padding: 40rpx 32rpx;
	background: #fff;
	border-bottom: 16rpx solid #f1f2f3;
}

.entry {
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 16rpx;
}

.entry-icon {
	width: 112rpx;
	height: 112rpx;
	border-radius: 24rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 56rpx;
}

.green {
	background: #e8f7f3;
}

.pink {
	background: #fce4ec;
}

.blue {
	background: #e3f2fd;
}

.entry-text {
	font-size: 26rpx;
	color: #18191c;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

/* 消息列表样式 */
.msg-list {
	flex: 1;
	min-height: 0;
	overflow-y: auto;
}

.empty-state {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 120rpx 0;
	color: #9499a0;
}

.empty-icon {
	font-size: 128rpx;
	margin-bottom: 32rpx;
	opacity: 0.5;
}

/* 消息项样式 */
.msg-item {
	display: flex;
	gap: 24rpx;
	padding: 32rpx;
	background: #fff;
	border-bottom: 1rpx solid #e3e5e7;
	align-items: flex-start;
}

/* 头像容器 */
.avatar-wrapper {
	position: relative;
	width: 96rpx;
	height: 96rpx;
	flex-shrink: 0;
}

.avatar {
	width: 96rpx;
	height: 96rpx;
	border-radius: 50%;
	position: absolute;
	top: 0;
	left: 0;
	z-index: 2;
}

/* 头像占位符 */
.avatar-placeholder {
	position: absolute;
	top: 0;
	left: 0;
	width: 96rpx;
	height: 96rpx;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 40rpx;
	font-weight: bold;
	color: #fff;
	z-index: 1;
}

.msg-body {
	flex: 1;
	min-width: 0;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
}

.msg-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.msg-title {
	font-size: 30rpx;
	font-weight: 500;
	color: #18191c;
	display: flex;
	align-items: center;
	gap: 12rpx;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

.live-tag {
	background: linear-gradient(90deg, #fb7299, #ff8a9e);
	color: #fff;
	font-size: 20rpx;
	padding: 2rpx 12rpx;
	border-radius: 8rpx;
}

.msg-time {
	font-size: 24rpx;
	color: #c0c4cc;
}

.msg-preview {
	font-size: 26rpx;
	color: #9499a0;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

.auto-reply {
	color: #c0c4cc;
}

/* 未读红点样式 */
.badge {
	background: #ff4d4f;
	color: #fff;
	font-size: 22rpx;
	min-width: 36rpx;
	height: 36rpx;
	line-height: 36rpx;
	text-align: center;
	padding: 0 12rpx;
	border-radius: 18rpx;
}

/* 底部菜单样式 */
.overlay {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background-color: rgba(0, 0, 0, 0.4);
	z-index: 100;
}

.bottom-sheet {
	position: fixed;
	bottom: 0;
	left: 0;
	right: 0;
	background-color: #fff;
	border-radius: 32rpx 32rpx 0 0;
	z-index: 101;
	padding-bottom: env(safe-area-inset-bottom);
}

.handle {
	width: 72rpx;
	height: 8rpx;
	background-color: #d0d0d0;
	border-radius: 4rpx;
	margin: 20rpx auto 30rpx;
}

.menu-item {
	padding: 32rpx 40rpx;
	border-bottom: 1rpx solid #f1f2f3;
}

.menu-text {
	font-size: 30rpx;
	color: #18191c;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

/* Toast 样式 */
.toast {
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
}

.toast.success {
	background-color: #52c41a;
}

/* ----- 头部图标 SVG ----- */
.icon-btn .icon-svg {
	width: 44rpx;
	height: 44rpx;
	color: #18191c;
	display: block;
}

/* ----- 快捷入口图标容器 ----- */
.entry-icon {
	width: 112rpx;
	height: 112rpx;
	border-radius: 24rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
	/* 确保容器不会变形 */
}

/* ----- 快捷入口内的 SVG 图标 ----- */
.entry-icon .icon-svg {
	width: 56rpx;
	/* 您可以根据实际大小微调 */
	height: 56rpx;
	display: block;
	/* 去除所有可能的外边距 */
	margin: 0;
	padding: 0;
	/* 保证图标居中显示，没有额外偏移 */
}

/* ----- 不同背景色的图标颜色 ----- */
.green .icon-svg {
	color: #EFCF7B;
	/* 黄色铃铛 */
}

.pink .icon-svg {
	color: #fff;
	/* 白色点赞 */
}

.blue .icon-svg {
	color: #f3a0c5;
	/* 粉色粉丝 */
}
</style>
