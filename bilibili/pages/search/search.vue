<template>
	<view class="phone">
		<!-- Toast -->
		<view class="toast" :class="{ show: toastVisible }">
			<text>{{ toastMessage }}</text>
		</view>

		<!-- 搜索栏 -->
		<view class="search-bar">
			<view class="search-row">
				<view class="search-box" :class="{ 'focus-in': searchFocused }">
					<text class="icon">🔍</text>
					<input
						class="search-input"
						v-model="searchKeyword"
						placeholder="搜索你感兴趣的内容"
						@focus="searchFocused = true"
						@blur="searchFocused = false"
						@confirm="doSearch(searchKeyword)"
					/>
					<view class="clear" :class="{ show: searchKeyword.length > 0 }" @click="clearSearch">
						<text>✕</text>
					</view>
				</view>
				<view class="search-cancel" @click="cancelSearch">
					<text>取消</text>
				</view>
			</view>
		</view>

		<!-- 内容区 -->
		<scroll-view class="content" scroll-y="true" :show-scrollbar="false">
			<!-- 热搜 -->
			<view class="section-header">
				<text class="section-title">bilibili热搜</text>
				<view class="section-more" @click="openTrending"><text>完整榜单 ›</text></view>
			</view>
			<view class="hot-list">
				<view
					v-for="(item, index) in hotData"
					:key="index"
					class="hot-item"
					@click="doSearch(item.kw)"
				>
					<text class="rank" :style="{ color: index < 3 ? ['#ff4d4f','#ff7a45','#ffa940'][index] : '#c0c4cc' }">{{ index + 1 }}</text>
					<text class="kw">{{ item.kw }}</text>
					<text v-if="item.badge" class="badge" :class="item.badge">{{ item.text }}</text>
				</view>
			</view>

			<!-- 搜索历史 -->
			<view class="section-header" style="margin-top:18px;">
				<text class="section-title">搜索历史</text>
			</view>
			<view class="history-group">
				<text v-if="historyData.length === 0" class="empty-tip">暂无搜索历史</text>
				<view v-for="(kw, index) in historyData" :key="'h'+index" class="history-tag">
					<text class="tag-text" @click="doSearch(kw)">{{ kw }}</text>
					<view class="tag-del" @click.stop="removeHistory(index)">
						<text>✕</text>
					</view>
				</view>
			</view>

			<!-- 搜索发现 -->
			<view class="section-header" style="margin-top:18px;">
				<text class="section-title">搜索发现</text>
			</view>
			<view class="discover-group">
				<text
					v-for="(kw, index) in discoverData"
					:key="'d'+index"
					class="discover-tag"
					@click="doSearch(kw)"
				>{{ kw }}</text>
			</view>

			<!-- 反馈 -->
			<view class="feedback" @click="showToast('感谢你的反馈 ❤️')">
				<text>反馈</text>
			</view>
		</scroll-view>

		<!-- 搜索覆盖层 -->
		<view class="page-overlay" :class="{ show: overlayVisible }">
			<view class="overlay-nav">
				<view class="back-btn" @click="closeOverlay"><text>‹</text></view>
				<text class="page-title">{{ overlayTitle }}</text>
				<view class="page-action" @click="closeOverlay"><text>完成</text></view>
			</view>
			<scroll-view class="overlay-content" scroll-y="true" :show-scrollbar="false">
				<view v-if="overlayType === 'search'" class="video-list">
					<view v-for="(video, idx) in overlayVideos" :key="'v'+idx" class="video-card" @click="playVideo(video)">
						<view class="video-cover">
							<image :src="video.cover || fallbackCover(idx)" mode="aspectFill" class="cover-img"></image>
							<view class="play-icon">
								<text style="color:#fff;font-size:18px;">▶</text>
							</view>
							<text class="duration">{{ video.duration || '10:00' }}</text>
						</view>
						<view class="video-info">
							<view class="title">
								<text v-for="(part, pi) in splitHighlight(video.title, overlayKeyword)" :key="pi" :class="{ highlight: part.hl }">{{ part.text }}</text>
							</view>
							<view class="up-row">
								<view class="avatar">
									<image v-if="video.avatar" :src="video.avatar" mode="aspectFill"></image>
								</view>
								<text class="up-name">{{ video.upName }} <text v-if="video.official" class="official">官方</text></text>
							</view>
							<view class="stats">
								<text>▶ {{ video.views || '0' }}</text>
								<text>💬 {{ video.comments || '0' }}</text>
								<text>🕐 {{ video.date || '刚刚' }}</text>
							</view>
						</view>
					</view>
				</view>

				<view v-else-if="overlayType === 'trending'" class="trending-list">
					<view v-for="(item, index) in hotData" :key="'t'+index" class="trending-item" @click="doSearch(item.kw)">
						<text class="t-rank" :style="{ color: index < 3 ? ['#ff4d4f','#ff7a45','#ffa940'][index] : '#c0c4cc' }">{{ index + 1 }}</text>
						<text class="t-kw">{{ item.kw }}</text>
						<text v-if="item.badge" class="badge" :class="item.badge">{{ item.text }}</text>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
const CUSTOM_DATA = {
	'claudecode': [
		{ title: '2026胎教级 华为大佬亲授Claude Code全栈教程，从入门到精通全覆盖', cover: '', upName: '百战程序员', avatar: '', views: '170.3万', comments: '294', duration: '25:43', date: '2026-05-21', official: false },
		{ title: 'ClaudeCode教程 2.claudecode常见操作(1)', cover: '', upName: 'ClaudeCode教程', avatar: '', views: '1.9万', comments: '74', duration: '12:49', date: '2026-03-09', official: false },
		{ title: '从零实现Claude Code（第二话）——Tool Use', cover: '', upName: 'AI开源社区', avatar: '', views: '7.2万', comments: '156', duration: '18:30', date: '2026-03-24', official: false },
		{ title: 'Claude Code 唯快不破 — 2026最新Vibe Coding操作演示', cover: '', upName: 'Kocoro-lab', avatar: '', views: '5.8万', comments: '89', duration: '27:38', date: '2026-02-17', official: false }
	],
	'梅西': [
		{ title: '阿根廷3-0阿尔及利亚，梅西帽子戏法，沉浸式看梅西破门', cover: '', upName: '真实球迷汇', avatar: '', views: '99.2万', comments: '412', duration: '04:43', date: '2026-06-17', official: false },
		{ title: '什么是球王！梅西帽子戏法+2助攻，阿根廷6比0大胜玻利维亚', cover: '', upName: '90分钟的艺术', avatar: '', views: '415.9万', comments: '1200', duration: '03:32', date: '2026-06-10', official: false },
		{ title: '绝世球王之出道即巅峰 第1集 小跳蚤帽子戏法绝平皇马', cover: '', upName: '球王档案馆', avatar: '', views: '56.7万', comments: '234', duration: '08:15', date: '2026-05-27', official: false },
		{ title: '梅西正在遭遇一场有预谋的造谣抹黑', cover: '', upName: '足球真相', avatar: '', views: '32.1万', comments: '567', duration: '03:38', date: '2026-06-16', official: false }
	],
	'diy汽车': [
		{ title: '90天！纯手工复刻给儿子圆梦ICAR', cover: '', upName: '手工耿', avatar: '', views: '1829.5万', comments: '4500', duration: '16:12', date: '2026-02-13', official: false },
		{ title: '自己设计制作美式风格车辆 制作一台拥有超强四驱系统的轿车', cover: '', upName: '汽车改装大师', avatar: '', views: '45.3万', comments: '189', duration: '12:30', date: '2026-03-14', official: false },
		{ title: '300天，我用毛毡和皮革手作了一辆EVO', cover: '', upName: '手作工坊', avatar: '', views: '78.6万', comments: '321', duration: '18:45', date: '2026-02-13', official: false },
		{ title: '当科幻照进现实！亚洲手工耿花费70天给儿子肝了辆AI智能跑车', cover: '', upName: '手工达人', avatar: '', views: '234.1万', comments: '876', duration: '13:48', date: '2024-04-18', official: false }
	],
	'雅思': [
		{ title: '雅思王听力真题语料库 3遍 完整版', cover: '', upName: '雅思王', avatar: '', views: '8215', comments: '56', duration: '05:12:27', date: '2026-04-24', official: false },
		{ title: '78集 万能句型 英语口语 双语朗读 雅思对话翻译 字幕', cover: '', upName: '大海英语频道', avatar: '', views: '1209.0万', comments: '768', duration: '46:30', date: '2026-05-29', official: false },
		{ title: '200集 剑桥大佬终于把雅思做成了动画片，日常磨耳朵！2026最新版', cover: '', upName: '剑桥雅思', avatar: '', views: '89.2万', comments: '345', duration: '45:30', date: '2026-03-18', official: true },
		{ title: '雅思口语每日跟读练习 从初级到中级高级 每天10分钟', cover: '', upName: '口语教练', avatar: '', views: '56.7万', comments: '234', duration: '10:00', date: '2025-12-27', official: false }
	],
	'原神': [
		{ title: '原神5.6从零开始到深渊满星合集纯享版', cover: '', upName: '阿炜18世', avatar: '', views: '316.7万', comments: '1200', duration: '03:01:11', date: '2025-06-04', official: false },
		{ title: '原神5.6B深渊 班玛希茜/神鹤爱芙 不吃苦', cover: '', upName: '原神攻略组', avatar: '', views: '1320', comments: '45', duration: '08:04', date: '2025-06-16', official: false },
		{ title: '原神5.6全新剧情合集 新魔神任务《悖理》', cover: '', upName: '原神剧情党', avatar: '', views: '45.6万', comments: '678', duration: '04:47', date: '2025-05-18', official: false }
	],
	'鬼灭之刃': [
		{ title: '燃向！鬼灭之刃无限城终极对决揭晓，一口气看完', cover: '', upName: '动漫剪辑师', avatar: '', views: '1.3万', comments: '89', duration: '02:40:07', date: '2025-10-19', official: false },
		{ title: '鬼灭之刃：无限城篇 第一章 猗窝座再袭 预告', cover: '', upName: '哔哩哔哩放映员', avatar: '', views: '234.5万', comments: '890', duration: '02:30', date: '2026-01-02', official: true },
		{ title: '鬼灭之刃无限城篇 第二章：血月回廊 4K极速蓝光未删减', cover: '', upName: '动漫先锋', avatar: '', views: '67.8万', comments: '456', duration: '01:45:30', date: '2026-02-03', official: false }
	]
}

export default {
	data() {
		return {
			searchKeyword: '',
			searchFocused: false,
			toastMessage: '',
			toastVisible: false,
			toastTimer: null,
			hotData: [
				{ kw: '徐静雨谈梅西世界杯射手王', badge: 'new', text: '新' },
				{ kw: '梅西足球之神', badge: 'new', text: '新' },
				{ kw: '梅西帽子戏法', badge: 'hot', text: '热' },
				{ kw: 'LCK MSI之路全位置数据', badge: 'new', text: '新' },
				{ kw: '姆巴佩世界波', badge: 'hot', text: '热' },
				{ kw: 'donk说相信自己能追上ZywOo', badge: '', text: '' },
				{ kw: '法国3-1塞内加尔', badge: '', text: '' },
				{ kw: '今晚8点京东618六大补...', badge: 'dacu', text: '大促' },
				{ kw: '哈兰德世界杯首球', badge: '', text: '' },
				{ kw: '血战高考梗', badge: '', text: '' }
			],
			historyData: [
				'diy汽车外观图纸', 'diy汽车', 'diy卡丁车', '自制汽车',
				'Bob the Canadian', '雅思王陆语料库', '王陆语料库'
			],
			discoverData: [
				'claude code', '王陆语料库', 'simon', 'claude code',
				'simon雅思写作', '雅思王陆语料库', 'claudecode接入deepseek',
				'天狗吃月亮', '自制卡丁车', '雅思听力',
				'claude接入deepseek', '英语听力磨耳朵'
			],
			overlayVisible: false,
			overlayTitle: '',
			overlayType: '',
			overlayKeyword: '',
			overlayVideos: []
		}
	},
	methods: {
		showToast(msg) {
			clearTimeout(this.toastTimer)
			this.toastMessage = msg
			this.toastVisible = true
			this.toastTimer = setTimeout(() => {
				this.toastVisible = false
			}, 1800)
		},
		clearSearch() {
			this.searchKeyword = ''
		},
		cancelSearch() {
			this.searchKeyword = ''
			this.showToast('已取消')
		},
		doSearch(kw) {
			if (!kw || !kw.trim()) return
			kw = kw.trim()
			this.searchKeyword = kw
			const idx = this.historyData.indexOf(kw)
			if (idx === -1) {
				this.historyData.unshift(kw)
				if (this.historyData.length > 20) this.historyData.pop()
			}
			this.showToast('🔍 搜索：' + kw)
			const videos = CUSTOM_DATA[kw] || this.generateFallbackVideos(kw)
			this.overlayKeyword = kw
			this.overlayVideos = videos.slice(0, 8)
			this.overlayType = 'search'
			this.overlayTitle = '搜索结果：' + kw
			this.overlayVisible = true
		},
		generateFallbackVideos(kw) {
			var list = []
			for (var i = 0; i < 4; i++) {
				list.push({
					title: kw + ' 相关视频 ' + (i + 1),
					cover: '',
					upName: 'UP主' + (i + 1),
					avatar: '',
					views: (Math.floor(Math.random() * 90) + 10) + '.' + Math.floor(Math.random() * 9) + '万',
					comments: Math.floor(Math.random() * 500) + 50,
					duration: (Math.floor(Math.random() * 20) + 2) + ':' + (Math.floor(Math.random() * 60) + '').padStart(2, '0'),
					date: ['1小时前', '昨天', '2天前', '3天前'][i % 4],
					official: false
				})
			}
			return list
		},
		fallbackCover(idx) {
			return 'https://picsum.photos/seed/search' + idx + '/400/225'
		},
		splitHighlight(title, keyword) {
			if (!keyword) return [{ text: title, hl: false }]
			var parts = []
			var esc = keyword.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
			var regex = new RegExp('(' + esc + ')', 'gi')
			var lastIdx = 0
			var match
			while ((match = regex.exec(title)) !== null) {
				if (match.index > lastIdx) {
					parts.push({ text: title.substring(lastIdx, match.index), hl: false })
				}
				parts.push({ text: match[0], hl: true })
				lastIdx = match.index + match[0].length
			}
			if (lastIdx < title.length) {
				parts.push({ text: title.substring(lastIdx), hl: false })
			}
			return parts.length > 0 ? parts : [{ text: title, hl: false }]
		},
		removeHistory(index) {
			if (index < 0 || index >= this.historyData.length) return
			var removed = this.historyData[index]
			this.historyData.splice(index, 1)
			this.showToast('已删除「' + removed + '」')
		},
		openTrending() {
			uni.switchTab({ url: '/pages/trending/trending' })
		},
		closeOverlay() {
			this.overlayVisible = false
		},
		playVideo(video) {
			// #ifdef H5
			if (video.url) {
				window.open(video.url, '_blank')
			} else {
				window.open('https://search.bilibili.com/all?keyword=' + encodeURIComponent(video.title), '_blank')
			}
			// #endif
			// #ifndef H5
			this.showToast('正在播放：' + video.title)
			// #endif
		}
	},
	mounted() {
		// 不再自动搜索，让用户看到搜索首页
	}
}
</script>

<style scoped>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	-webkit-tap-highlight-color: transparent;
}

:root {
	--bili-pink: #fb7299;
	--bili-pink-light: #fff0f4;
	--bili-bg: #f4f5f7;
	--bili-white: #ffffff;
	--bili-text: #18191c;
	--bili-text-secondary: #9499a0;
	--bili-text-muted: #c0c4cc;
	--bili-border: #e3e5e7;
	--bili-card-radius: 12px;
}

.phone {
	width: 100%;
	height: 100vh;
	background: var(--bili-bg);
	overflow: hidden;
	display: flex;
	flex-direction: column;
	position: relative;
	margin: 0 auto;
	padding-bottom: 100rpx;
	box-sizing: border-box;
}

/* Toast */
.toast {
	position: fixed;
	top: 80px;
	left: 50%;
	transform: translateX(-50%) translateY(-100px);
	background: rgba(0, 0, 0, 0.72);
	color: #fff;
	padding: 8px 20px;
	border-radius: 18px;
	font-size: 13px;
	font-weight: 500;
	z-index: 999;
	pointer-events: none;
	white-space: nowrap;
	transition: transform 0.3s ease, opacity 0.3s ease;
	opacity: 0;
}
.toast.show {
	transform: translateX(-50%) translateY(0);
	opacity: 1;
}

/* 搜索栏 */
.search-bar {
	background: #fff;
	padding: 8px 16px 10px;
	flex-shrink: 0;
	z-index: 10;
}
.search-row {
	display: flex;
	align-items: center;
	gap: 10px;
}
.search-box {
	flex: 1;
	display: flex;
	align-items: center;
	background: #f1f2f5;
	border-radius: 20px;
	padding: 0 12px;
	height: 36px;
	gap: 6px;
	border: 1px solid transparent;
	transition: all 0.2s ease;
}
.search-box.focus-in {
	background: #fff;
	border-color: #fb7299;
	box-shadow: 0 0 0 3px rgba(251, 114, 153, 0.06);
}
.search-box .icon {
	font-size: 16px;
	color: #9499a0;
	flex-shrink: 0;
}
.search-input {
	flex: 1;
	border: none;
	background: none;
	outline: none;
	font-size: 14px;
	color: #18191c;
	height: 100%;
	min-width: 0;
}
.search-input::placeholder {
	color: #c0c4cc;
}
.clear {
	display: none;
	width: 18px;
	height: 18px;
	border-radius: 50%;
	background: #c0c4cc;
	color: #fff;
	font-size: 11px;
	font-weight: 600;
	flex-shrink: 0;
	line-height: 18px;
	text-align: center;
	align-items: center;
	justify-content: center;
}
.clear.show {
	display: flex;
}
.search-cancel {
	font-size: 14px;
	color: #fb7299;
	font-weight: 500;
	flex-shrink: 0;
	padding: 4px 0;
}

/* 内容区 */
.content {
	flex: 1;
	overflow-y: auto;
	padding: 0 16px 12px;
}
.section-header {
	display: flex;
	justify-content: space-between;
	align-items: baseline;
	margin: 14px 0 8px;
}
.section-title {
	font-size: 15px;
	font-weight: 600;
	color: #18191c;
}
.section-more {
	font-size: 12px;
	color: #9499a0;
	padding: 4px 0;
}
.section-more:active {
	color: #fb7299;
}

/* 热搜列表 */
.hot-list {
	background: #fff;
	border-radius: 12px;
	overflow: hidden;
}
.hot-list .hot-item {
	display: flex;
	align-items: center;
	padding: 11px 14px;
	border-bottom: 1px solid #f1f2f5;
	gap: 10px;
	font-size: 14px;
	color: #18191c;
}
.hot-list .hot-item:last-child {
	border-bottom: none;
}
.hot-list .hot-item:active {
	background: #f8f9fb;
}
.hot-list .rank {
	width: 22px;
	text-align: center;
	font-weight: 700;
	font-size: 15px;
	flex-shrink: 0;
}
.hot-list .kw {
	flex: 1;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}
.badge {
	font-size: 10px;
	padding: 0 7px;
	border-radius: 3px;
	font-weight: 600;
	flex-shrink: 0;
	line-height: 1.8;
}
.badge.hot { background: #fff0f4; color: #fb7299; }
.badge.new { background: #fff7e6; color: #fa8c16; }
.badge.dacu { background: #fff1f0; color: #ff4d4f; }

/* 搜索历史 & 发现 */
.history-group,
.discover-group {
	display: flex;
	flex-wrap: wrap;
	gap: 8px;
	background: #fff;
	padding: 12px 14px;
	border-radius: 12px;
	min-height: 48px;
	align-items: center;
}
.empty-tip {
	width: 100%;
	text-align: center;
	color: #c0c4cc;
	font-size: 13px;
	padding: 4px 0;
}
.history-tag {
	display: inline-flex;
	align-items: center;
	background: #f1f2f5;
	border-radius: 14px;
	padding: 4px 6px 4px 14px;
	gap: 2px;
	font-size: 12px;
	color: #18191c;
	line-height: 1.6;
	height: 28px;
}
.history-tag .tag-text {
	font-weight: 400;
	white-space: nowrap;
}
.history-tag .tag-del {
	display: inline-flex;
	align-items: center;
	justify-content: center;
	width: 18px;
	height: 18px;
	border-radius: 50%;
	background: #c0c4cc;
	color: #fff;
	font-size: 10px;
	font-weight: 700;
	line-height: 1;
	flex-shrink: 0;
	margin-left: 2px;
}
.discover-tag {
	background: #f1f2f5;
	padding: 4px 14px;
	border-radius: 14px;
	font-size: 12px;
	color: #18191c;
	line-height: 1.6;
	height: 28px;
	display: inline-flex;
	align-items: center;
}
.discover-tag:active {
	background: #fb7299;
	color: #fff;
}

.feedback {
	text-align: center;
	padding: 16px 0 6px;
	font-size: 12px;
	color: #c0c4cc;
}

/* 覆盖层 */
.page-overlay {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: #f4f5f7;
	z-index: 50;
	display: flex;
	flex-direction: column;
	transform: translateY(100%);
	transition: transform 0.35s ease;
	overflow: hidden;
}
.page-overlay.show {
	transform: translateY(0);
}
.overlay-nav {
	display: flex;
	align-items: center;
	padding: 4px 8px 10px 4px;
	background: #fff;
	border-bottom: 1px solid #f0f1f3;
	flex-shrink: 0;
	gap: 4px;
	height: 46px;
}
.back-btn {
	font-size: 26px;
	color: #18191c;
	line-height: 1;
	padding: 4px 8px;
	font-weight: 300;
}
.page-title {
	font-size: 16px;
	font-weight: 600;
	color: #18191c;
	flex: 1;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}
.page-action {
	font-size: 14px;
	color: #fb7299;
	font-weight: 500;
	padding: 4px 12px;
}
.overlay-content {
	flex: 1;
	overflow-y: auto;
	padding: 12px 16px 20px;
	background: #f4f5f7;
}

/* 视频卡片 */
.video-list {
	display: flex;
	flex-direction: column;
}
.video-card {
	background: #fff;
	border-radius: 12px;
	overflow: hidden;
	margin-bottom: 12px;
}
.video-card:active {
	transform: scale(0.985);
}
.video-cover {
	position: relative;
	width: 100%;
	padding-top: 56.25%;
	background: #1a1a2e;
	overflow: hidden;
}
.cover-img {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
}
.video-cover .play-icon {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 44px;
	height: 44px;
	background: rgba(0, 0, 0, 0.5);
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
}
.video-cover .duration {
	position: absolute;
	bottom: 6px;
	right: 6px;
	background: rgba(0, 0, 0, 0.7);
	color: #fff;
	font-size: 10px;
	font-weight: 600;
	padding: 1px 6px;
	border-radius: 3px;
}
.video-info {
	padding: 10px 12px 12px;
}
.video-info .title {
	font-size: 14px;
	font-weight: 600;
	color: #18191c;
	line-height: 1.5;
}
.video-info .title .highlight {
	color: #fb7299;
}
.up-row {
	display: flex;
	align-items: center;
	gap: 6px;
	margin-top: 6px;
}
.up-row .avatar {
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background: #e8eaed;
	flex-shrink: 0;
	overflow: hidden;
}
.up-row .avatar image {
	width: 100%;
	height: 100%;
}
.up-row .up-name {
	font-size: 12px;
	color: #9499a0;
	flex: 1;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}
.official {
	display: inline-block;
	background: #fb7299;
	color: #fff;
	font-size: 9px;
	padding: 0 4px;
	border-radius: 2px;
	margin-left: 4px;
	line-height: 1.6;
}
.stats {
	display: flex;
	gap: 12px;
	margin-top: 4px;
	font-size: 11px;
	color: #c0c4cc;
}

/* 热门榜单 */
.trending-list {
	display: flex;
	flex-direction: column;
}
.trending-item {
	display: flex;
	align-items: center;
	padding: 10px 14px;
	gap: 12px;
	border-radius: 12px;
	background: #fff;
	margin-bottom: 8px;
}
.t-rank {
	font-weight: 700;
	font-size: 15px;
	width: 22px;
	text-align: center;
	flex-shrink: 0;
}
.t-kw {
	flex: 1;
	font-size: 14px;
	color: #18191c;
}
</style>
