<template>
	<view class="app">
		<!-- 分类导航 -->
		<scroll-view class="category-scroll" scroll-x="true" :show-scrollbar="false">
			<text
				v-for="cat in categories"
				:key="cat.key"
				class="chip"
				:class="{ active: currentCategory === cat.key }"
				@click="switchCategory(cat.key)"
			>{{ cat.label }}</text>
		</scroll-view>

		<!-- 内容区 -->
		<scroll-view class="feed" scroll-y="true" @scroll="onFeedScroll" :scroll-top="scrollTopValue" :scroll-with-animation="true">
			<!-- 轮播图（仅推荐页） -->
			<view v-if="currentCategory === 'recommend' && banners.length > 0" class="banner-wrapper">
				<view class="banner-carousel" @touchstart="onTouchStart" @touchend="onTouchEnd">
					<view class="banner-track" :style="{ transform: 'translateX(-' + bannerIndex * 100 + '%)' }">
						<view
							v-for="(banner, idx) in banners"
							:key="idx"
							class="banner-slide"
							@click="openBannerVideo(banner)"
						>
							<image :src="getCoverSrc({ url: banner.url, cover: banner.img, cat: 'recommend' })" mode="aspectFill" class="banner-image"></image>
							<view class="banner-overlay">
								<text class="banner-title">{{ banner.title }}</text>
							</view>
						</view>
					</view>
					<view class="banner-dots">
						<view
							v-for="(banner, idx) in banners"
							:key="idx"
							class="dot"
							:class="{ active: bannerIndex === idx }"
							@click.stop="goToBanner(idx)"
						></view>
					</view>
					<view class="banner-btn banner-btn-prev" @click.stop="prevBanner">
						<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
							<polyline points="15,18 9,12 15,6"/>
						</svg>
					</view>
					<view class="banner-btn banner-btn-next" @click.stop="nextBanner">
						<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
							<polyline points="9,18 15,12 9,6"/>
						</svg>
					</view>
				</view>
			</view>

			<!-- 视频列表 -->
			<view v-if="filteredVideos.length > 0" class="video-list">
				<view
					v-for="video in filteredVideos"
					:key="video.id"
					class="video-card"
					@click="openVideoDetail(video)"
				>
					<view class="thumb">
						<image :src="getCoverSrc(video)" mode="aspectFill" class="thumb-image"></image>
						<text class="duration">{{ video.duration }}</text>
						<text class="play-badge">
							<svg viewBox="0 0 24 24" width="10" height="10" fill="white">
								<polygon points="5,3 19,12 5,21 5,3"/>
							</svg>
							{{ video.views }}
						</text>
					</view>
					<view class="info">
						<text class="title">{{ video.title }}</text>
						<view class="meta">
							<text class="author">👤 {{ video.author }}</text>
							<view class="stats">
								<text class="stat-item">▶ {{ video.views }}</text>
								<text class="stat-item">💬 {{ video.danmaku }}</text>
							</view>
						</view>
					</view>
					<text class="delete-btn" @click.stop="showDeleteModal(video)">✕</text>
				</view>
			</view>
			<view v-else class="empty-state">
				<text class="empty-icon">📭</text>
				<text class="empty-title">暂无内容</text>
				<text class="empty-desc">换个分类看看其他视频吧</text>
			</view>

		</scroll-view>

		<view class="back-to-top" v-show="showBackToTop" @click="scrollToTop">
			<svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
				<polyline points="18 15 12 9 6 15"/>
			</svg>
		</view>

		<!-- 删除确认弹窗 -->
		<view class="modal-overlay" :class="{ show: modalVisible }" @click="modalVisible = false">
			<view class="modal-box" @click.stop>
				<text class="modal-icon">🤔</text>
				<text class="modal-title">确定要删除吗？</text>
				<text class="modal-desc">删除后我们会减少此类推荐</text>
				<view class="modal-actions">
					<button class="modal-cancel" @click="modalVisible = false">取消</button>
					<button class="modal-confirm" @click="confirmDelete">确定删除</button>
				</view>
			</view>
		</view>

		<!-- 删除成功提示 -->
		<view class="toast" :class="{ show: deleteToastVisible }">
			<text class="toast-icon">👌</text>
			<text>已减少此类推荐</text>
		</view>
	</view>
</template>

<script>
const HARDCODED_COVERS = {
	'BV1E7wtzaEdq': 'https://i0.hdslb.com/bfs/archive/fbafd1a0092b225c0b3adbbb28f931ea9eac5cf6.jpg',
	'BV1ouDbBiEWd': 'https://i0.hdslb.com/bfs/archive/118092453e52705a99a8e74a1b498588100c90ce.jpg',
	'BV1zjJ564EZc': 'https://i1.hdslb.com/bfs/archive/05746f782aa103a42b60cde6dbafeea4db849da5.jpg',
	'BV1SxE36fETR': 'https://i1.hdslb.com/bfs/archive/b31743b9c186e8c6e90948ab23b300668c59c709.jpg',
	'BV1GM41157Hz': 'https://i1.hdslb.com/bfs/archive/520cd422f60d38924e41d377d9899d2f5fbe434d.jpg',
	'BV1JeJw64E9d': 'https://i2.hdslb.com/bfs/archive/4831e4af6c81aec7f025a491ca08e42f503a63b2.jpg',
	'BV1We4y1g7sj': 'https://i2.hdslb.com/bfs/archive/ec7f8686d50496aa62d190e20c160943f586ca67.jpg',
	'BV1bUTYz5EiW': 'https://i2.hdslb.com/bfs/archive/fc672e36278883fb50c594169e6366c014a14eae.png',
	'BV1yZ1MB1Eej': 'https://i2.hdslb.com/bfs/archive/1d16055b7a39010990688cc2eed6cd546899f279.jpg',
	'BV16p9UBgEhT': 'https://i2.hdslb.com/bfs/archive/68d6b3fe2a973cff2d5adebef1ae2f9f9218cc72.jpg',
	'BV1ww4m1S7Qv': 'https://i2.hdslb.com/bfs/archive/3deceacd63172b261bf0af21c368ebe8815cf029.png',
	'BV1HPNgzFEoq': 'https://i2.hdslb.com/bfs/archive/114d71e6d97c5e5fa7c13284afe5b0c1404ae2b6.jpg',
	'BV17b41187ME': 'https://i0.hdslb.com/bfs/archive/a4a1949e7fd977ed2a90bafe2cb08a865b99ad5f.jpg',
	'BV1eN4y1w7SS': 'https://i1.hdslb.com/bfs/archive/5e9f663bd283c79a1b055964c61b808886d6ee3e.jpg',
	'BV1pT411G72y': 'https://i0.hdslb.com/bfs/archive/0d991b2d26c9b1644e377cda5fcc3debe6ce0821.jpg',
	'BV1Ga4y1A7e2': 'https://i1.hdslb.com/bfs/archive/933ff0c324af25616b8212528c897d615f6fccc9.jpg',
	'BV1WGB4BCE9B': 'https://i0.hdslb.com/bfs/archive/3bfe7ece420c151fe2609e8708c60e1a711e3aef.jpg',
	'BV1Jq6SBkEdU': 'https://i2.hdslb.com/bfs/archive/6e1dc378d1a8feceb60d30124abd91557313a135.jpg',
	'BV1cA4112728': 'https://i0.hdslb.com/bfs/archive/ef68bf80ef022ebbeb08f87e0e5e410e5a0a009d.jpg',
	'BV1yyXVB5EY3': 'https://i1.hdslb.com/bfs/archive/08765a2b9883658fe54eb56a7c7ad820fc5290b5.jpg',
	'BV1pG4y1o73N': 'https://i2.hdslb.com/bfs/archive/b1a65fe39259d011c3eba331b45ff6967f81ce74.jpg',
	'BV1eUszzJEDo': 'https://i2.hdslb.com/bfs/archive/45cb56557aaf6a666476bf14e7883228e5bff3f6.jpg',
	'BV1yW1vBXE6T': 'https://i2.hdslb.com/bfs/archive/dbd740f8c0dc50606250b86490ea9ceea68adfd1.jpg',
	'BV1478VzVE2b': 'https://i1.hdslb.com/bfs/archive/5e6d3394a33a556601b1758d0ef060eb7cc30b28.png',
	'BV17ApmzKEEz': 'https://i2.hdslb.com/bfs/archive/6e817f216227ba839ef532abb797d984ae33e786.jpg',
	'BV17TEC6AE3Z': 'https://i1.hdslb.com/bfs/archive/eba34b3f270343f2e9d00cfe39017bc85914227f.jpg',
}

export default {
	data() {
		return {
			categories: [
				{ key: 'recommend', label: '推荐' },
				{ key: 'live', label: '直播' },
				{ key: 'anime', label: '动画' },
				{ key: 'game', label: '游戏' },
				{ key: 'knowledge', label: '知识' },
				{ key: 'music', label: '音乐' },
				{ key: 'movie', label: '影视' },
				{ key: 'tech', label: '科技' },
				{ key: 'life', label: '生活' },
				{ key: 'fashion', label: '时尚' },
				{ key: 'food', label: '美食' },
				{ key: 'sport', label: '运动' },
				{ key: 'car', label: '汽车' },
			],
			currentCategory: 'recommend',

			banners: [
				{ img: 'https://i0.hdslb.com/bfs/archive/fbafd1a0092b225c0b3adbbb28f931ea9eac5cf6.jpg', title: '🔥 AI 技术前沿', url: 'https://www.bilibili.com/video/BV1E7wtzaEdq', bvid: 'BV1E7wtzaEdq' },
				{ img: 'https://i1.hdslb.com/bfs/archive/520cd422f60d38924e41d377d9899d2f5fbe434d.jpg', title: '🍳 青椒炒肉丝', url: 'https://www.bilibili.com/video/BV1GM41157Hz', bvid: 'BV1GM41157Hz' },
				{ img: 'https://i2.hdslb.com/bfs/archive/1d16055b7a39010990688cc2eed6cd546899f279.jpg', title: '🎮 星穹铁道·昔涟角色PV', url: 'https://www.bilibili.com/video/BV1yZ1MB1Eej', bvid: 'BV1yZ1MB1Eej' },
				{ img: 'https://i1.hdslb.com/bfs/archive/933ff0c324af25616b8212528c897d615f6fccc9.jpg', title: '🎬 肖申克的救赎', url: 'https://www.bilibili.com/video/BV1Ga4y1A7e2', bvid: 'BV1Ga4y1A7e2' },
			],
			bannerIndex: 0,
			autoPlayTimer: null,
			isTouching: false,
			touchStartX: 0,
			scrollTopValue: 0,

			videoData: [
				{ id: 1, title: '我们做了个不一样的拐杖...', author: '老师好我叫何同学', views: '135.7万', danmaku: '1890', cat: 'recommend', url: 'https://www.bilibili.com/video/BV17TEC6AE3Z', duration: '07:00', cover: 'https://i1.hdslb.com/bfs/archive/eba34b3f270343f2e9d00cfe39017bc85914227f.jpg', desc: '何同学2026最新创意发明。', tags: ['创意', '发明', '何同学'] },
				{ id: 2, title: '从 LLM 到 Agent Skill，一期视频带你打通底层逻辑！', author: '马克的技术工作坊', views: '117.2万', danmaku: '1632', cat: 'knowledge', url: 'https://www.bilibili.com/video/BV1E7wtzaEdq/', duration: '32:31', desc: '从LLM到Agent Skill完整教程。', tags: ['AI', '教程', 'Agent'] },
				{ id: 3, title: '【原神宝箱全收集】蒙德月之六', author: '你的影月月', views: '144.0万', danmaku: '3311', cat: 'game', url: 'https://www.bilibili.com/video/BV1ouDbBiEWd/', duration: '00:36', desc: '原神蒙德宝箱全收集攻略。', tags: ['原神', '攻略', '宝箱'] },
				{ id: 4, title: 'iPhone 18 Pro Max 机模上手！全新设计细节曝光', author: '油管视频搬运-Apple', views: '1549', danmaku: '1', cat: 'tech', url: 'https://www.bilibili.com/video/BV1zjJ564EZc', duration: '10:22', desc: 'iPhone 18 Pro Max 机模上手体验。', tags: ['iPhone', '科技', '评测'] },
				{ id: 5, title: '【无 限 城 Reaction】这一战太惨烈了！', author: '仓鼠小姐de频道', views: '46.0万', danmaku: '1380', cat: 'anime', url: 'https://www.bilibili.com/video/BV1SxE36fETR', duration: '03:16', desc: '无限城Reaction，又哭又笑。', tags: ['Reaction', '动画', '无限城'] },
				{ id: 6, title: '厨师长一镜分享"青椒炒肉丝"的门门道道', author: '美食作家王刚R', views: '259.3万', danmaku: '8218', cat: 'food', url: 'https://www.bilibili.com/video/BV1GM41157Hz', duration: '14:59', desc: '王刚教你做青椒炒肉丝。', tags: ['美食', '教程', '王刚'] },
				{ id: 7, title: '【2026LPL第二赛段】6月14日 TES vs BLG', author: '哔哩哔哩英雄联盟赛事', views: '116.1万', danmaku: '1.0万', cat: 'game', url: 'https://www.bilibili.com/video/BV1JeJw64E9d', duration: '51:11', desc: 'LPL第二赛段TES vs BLG。', tags: ['LPL', '电竞', '英雄联盟'] },
				{ id: 8, title: '周杰伦JAY 《晴天》 钢琴高度还原', author: 'Czzy__', views: '100.8万', danmaku: '463', cat: 'music', url: 'https://www.bilibili.com/video/BV1We4y1g7sj', duration: '04:28', desc: '周杰伦《晴天》钢琴版。', tags: ['音乐', '钢琴', '周杰伦'] },
				{ id: 9, title: '2025最新Vue3+Typescript 从进阶到实战到原理教程', author: 'vue3入门到精通', views: '13.1万', danmaku: '190', cat: 'knowledge', url: 'https://www.bilibili.com/video/BV1bUTYz5EiW', duration: '04:06', desc: 'Vue3+TS从入门到精通。', tags: ['Vue3', '教程', '前端'] },
				{ id: 10, title: '《崩坏：星穹铁道》昔涟角色PV——「再度和你」', author: '崩坏星穹铁道', views: '1064.0万', danmaku: '33.2万', cat: 'game', url: 'https://www.bilibili.com/video/BV1yZ1MB1Eej', duration: '03:32', desc: '星穹铁道昔涟角色PV。', tags: ['星穹铁道', 'PV', '昔涟'] },
				{ id: 11, title: '《崩坏：星穹铁道》「2026星铁奇妙夜」', author: '崩坏星穹铁道', views: '425.1万', danmaku: '19.6万', cat: 'live', url: 'https://www.bilibili.com/video/BV16p9UBgEhT', duration: '51:30', desc: '星铁奇妙夜线上音乐演出。', tags: ['星穹铁道', '音乐', '演出'] },
				{ id: 12, title: '（克利-咕咕兰）看原神4.7前瞻直播', author: '莉亚', views: '2.8万', danmaku: '60', cat: 'live', url: 'https://www.bilibili.com/video/BV1ww4m1S7Qv', duration: '55:41', desc: '原神4.7前瞻直播。', tags: ['原神', '直播', '前瞻'] },
				{ id: 13, title: '这么多真的受不了啊！2026年4月新番导视！【泛式】', author: '泛式', views: '347.6万', danmaku: '2.8万', cat: 'anime', url: 'https://www.bilibili.com/video/BV1HPNgzFEoq', duration: '22:14', desc: '2026年4月新番导视。', tags: ['新番', '导视', '泛式'] },
				{ id: 14, title: '奥斯卡2019最佳动画短片《包宝宝》 超温情', author: '以希同学', views: '316.3万', danmaku: '5363', cat: 'anime', url: 'https://www.bilibili.com/video/BV17b41187ME', duration: '07:18', desc: '奥斯卡最佳动画短片《包宝宝》。', tags: ['动画', '短片', '奥斯卡'] },
				{ id: 15, title: '一口气看懂量子力学，量子叠加，量子纠缠', author: '科学认识论', views: '165.1万', danmaku: '5771', cat: 'knowledge', url: 'https://www.bilibili.com/video/BV1eN4y1w7SS', duration: '17:53', desc: '量子力学入门科普。', tags: ['科普', '量子力学', '物理'] },
				{ id: 16, title: '路过听到有人唱林俊杰版《起风了》？？', author: '是大大龙', views: '26.0万', danmaku: '558', cat: 'music', url: 'https://www.bilibili.com/video/BV1pT411G72y', duration: '05:10', desc: '街头听到林俊杰版《起风了》。', tags: ['音乐', '翻唱', '街头'] },
				{ id: 17, title: '评分9.7排名第一！无数人在它身上找到了救赎之道！', author: '顾我电影', views: '815.3万', danmaku: '3.5万', cat: 'movie', url: 'https://www.bilibili.com/video/BV1Ga4y1A7e2', duration: '28:44', desc: '《肖申克的救赎》深度解说。', tags: ['电影', '解说', '肖申克的救赎'] },
				{ id: 18, title: '2026年最期待的10部华语电影', author: '张头头A', views: '5.6万', danmaku: '169', cat: 'movie', url: 'https://www.bilibili.com/video/BV1WGB4BCE9B', duration: '06:27', desc: '2026最期待华语电影盘点。', tags: ['电影', '盘点', '华语'] },
				{ id: 19, title: '埃隆·马斯克2026首次访谈', author: 'YouTube访谈实录', views: '76.5万', danmaku: '3075', cat: 'tech', url: 'https://www.bilibili.com/video/BV1Jq6SBkEdU', duration: '2:52:10', desc: '马斯克2026首次访谈完整版。', tags: ['马斯克', '访谈', '科技'] },
				{ id: 20, title: '云南旅游vlog｜一个人去大理太治愈了！', author: '-小余转晴-', views: '119.2万', danmaku: '1920', cat: 'life', url: 'https://www.bilibili.com/video/BV1cA4112728', duration: '31:55', desc: '大理旅游vlog。', tags: ['旅游', 'vlog', '大理'] },
				{ id: 21, title: '【2026春夏流行趋势】8分钟教你今年春夏怎么穿', author: 'BettySays', views: '43.8万', danmaku: '606', cat: 'fashion', url: 'https://www.bilibili.com/video/BV1yyXVB5EY3', duration: '09:00', desc: '2026春夏流行趋势。', tags: ['时尚', '穿搭', '流行'] },
				{ id: 22, title: '每天卖上千个！路边小吃摊御好烧人气火爆', author: '迹录Fount', views: '46.5万', danmaku: '759', cat: 'food', url: 'https://www.bilibili.com/video/BV1pG4y1o73N', duration: '06:22', desc: '日本路边御好烧制作。', tags: ['美食', '日本', '街头'] },
				{ id: 23, title: '日本vs中国男足全场回放 2024-09-05', author: '局域网用户', views: '4.4万', danmaku: '618', cat: 'sport', url: 'https://www.bilibili.com/video/BV1eUszzJEDo', duration: '2:14:14', desc: '日本vs中国男足全场比赛。', tags: ['足球', '国足', '比赛'] },
				{ id: 24, title: '全网最详细！仰望U8L评测', author: '韩路有点意思', views: '16.1万', danmaku: '686', cat: 'car', url: 'https://www.bilibili.com/video/BV1yW1vBXE6T', duration: '41:27', desc: '仰望U8L详细评测。', tags: ['汽车', '评测', '仰望'] },
				{ id: 25, title: '2025年最爆笑猫狗合集 😼🐶 Part 2', author: '喵星cat人', views: '43.4万', danmaku: '1174', cat: 'recommend', url: 'https://www.bilibili.com/video/BV1478VzVE2b', duration: '18:17', desc: '爆笑猫狗合集。', tags: ['搞笑', '猫', '狗'] },
				{ id: 26, title: '【日漫神曲合集】"还记得那些年熬夜追过的番"', author: 'Cugino', views: '3242', danmaku: '7', cat: 'recommend', url: 'https://www.bilibili.com/video/BV17ApmzKEEz', duration: '37:49', desc: '日漫神曲合集。', tags: ['音乐', '动漫', '合集'] },
			],

			deletedIds: [],
			modalVisible: false,
			pendingDeleteVideo: null,
			deleteToastVisible: false,
			deleteToastTimer: null,
			showBackToTop: false,
		}
	},

	computed: {
		filteredVideos() {
			let list = this.currentCategory === 'recommend'
				? this.videoData
				: this.videoData.filter(v => v.cat === this.currentCategory)
			return list.filter(v => !this.deletedIds.includes(v.id))
		},
	},

	methods: {
		switchCategory(cat) {
			this.currentCategory = cat
			this.scrollTopValue = 0
			if (cat === 'recommend') {
				this.$nextTick(() => this.startAutoPlay())
			} else {
				this.stopAutoPlay()
			}
		},

		getBvid(url) {
			const m = url.match(/BV[a-zA-Z0-9]+/)
			return m ? m[0] : null
		},

		getPlaceholderCover(cat) {
			const colors = {
				recommend: ['#fb7299', '#ff5e85'], live: ['#ff6699', '#ff3377'],
				anime: ['#ff9933', '#ff6600'], game: ['#3399ff', '#0066cc'],
				knowledge: ['#00b894', '#008866'], music: ['#a55eea', '#7c3aed'],
				movie: ['#e17055', '#c0392b'], tech: ['#0984e3', '#0565b0'],
				life: ['#00cec9', '#009999'], fashion: ['#fd79a8', '#e84393'],
				food: ['#fdcb6e', '#f39c12'], sport: ['#6c5ce7', '#4834d4'],
				car: ['#636e72', '#2d3436'],
			}
			const c = colors[cat] || colors['recommend']
			const catNames = { recommend:'推荐', live:'直播', anime:'动画', game:'游戏', knowledge:'知识', music:'音乐', movie:'影视', tech:'科技', life:'生活', fashion:'时尚', food:'美食', sport:'运动', car:'汽车' }
			const name = catNames[cat] || cat
			const svg = '<svg xmlns="http://www.w3.org/2000/svg" width="280" height="176">'
				+ '<defs><linearGradient id="g" x1="0%" y1="0%" x2="100%" y2="100%">'
				+ '<stop offset="0%" stop-color="' + c[0] + '"/>'
				+ '<stop offset="100%" stop-color="' + c[1] + '"/>'
				+ '</linearGradient></defs>'
				+ '<rect fill="url(#g)" width="280" height="176" rx="6"/>'
				+ '<text fill="rgba(255,255,255,0.25)" x="140" y="78" text-anchor="middle" dominant-baseline="central" font-size="56" font-weight="700">▶</text>'
				+ '<text fill="rgba(255,255,255,0.9)" x="140" y="138" text-anchor="middle" dominant-baseline="central" font-size="14" font-weight="600">' + name + '</text>'
				+ '</svg>'
			return 'data:image/svg+xml,' + encodeURIComponent(svg)
		},

		getCoverSrc(video) {
			if (video.cover) return video.cover
			const bvid = this.getBvid(video.url)
			if (bvid && HARDCODED_COVERS[bvid]) return HARDCODED_COVERS[bvid]
			return this.getPlaceholderCover(video.cat)
		},

		// Banner
		nextBanner() { this.bannerIndex = (this.bannerIndex + 1) % this.banners.length; this.resetAutoPlay(); },
		prevBanner() { this.bannerIndex = (this.bannerIndex - 1 + this.banners.length) % this.banners.length; this.resetAutoPlay(); },
		goToBanner(index) { this.bannerIndex = index; this.resetAutoPlay(); },
		startAutoPlay() { this.stopAutoPlay(); if (this.currentCategory === 'recommend') { this.autoPlayTimer = setInterval(() => { this.nextBanner() }, 4000); } },
		stopAutoPlay() { if (this.autoPlayTimer) { clearInterval(this.autoPlayTimer); this.autoPlayTimer = null; } },
		resetAutoPlay() { this.stopAutoPlay(); this.startAutoPlay(); },
		onTouchStart(e) { this.isTouching = true; this.touchStartX = e.touches[0].clientX; this.stopAutoPlay(); },
		onTouchEnd(e) { if (!this.isTouching) return; this.isTouching = false; const diff = this.touchStartX - e.changedTouches[0].clientX; if (Math.abs(diff) > 30) { diff > 0 ? this.nextBanner() : this.prevBanner(); } this.startAutoPlay(); },
		openBannerVideo(banner) { const video = this.videoData.find(v => v.url.includes(banner.bvid)); if (video) { this.openVideoDetail(video); } else { uni.showToast({ title: '正在跳转：' + banner.title, icon: 'none' }); } },

		openVideoDetail(video) {
			const bvid = this.getBvid(video.url)
			const cover = this.getCoverSrc(video)
			uni.navigateTo({
				url: `/pages/detail/detail?bvid=${bvid}&title=${encodeURIComponent(video.title)}&author=${encodeURIComponent(video.author)}&views=${video.views}&danmaku=${video.danmaku}&duration=${video.duration}&thumb=${encodeURIComponent(cover)}&desc=${encodeURIComponent(video.desc || video.title)}&tags=${encodeURIComponent(JSON.stringify(video.tags || []))}`
			})
		},

		onFeedScroll(e) { this.showBackToTop = e.detail.scrollTop > 300; },
		scrollToTop() {
			this.scrollTopValue = this.scrollTopValue === 0 ? 1 : 0;
			this.showBackToTop = false;
		},

		showDeleteModal(video) { this.pendingDeleteVideo = video; this.modalVisible = true; },
		confirmDelete() {
			if (this.pendingDeleteVideo) {
				const id = this.pendingDeleteVideo.id
				if (!this.deletedIds.includes(id)) this.deletedIds.push(id)
				this.deleteToastVisible = true
				clearTimeout(this.deleteToastTimer)
				this.deleteToastTimer = setTimeout(() => { this.deleteToastVisible = false }, 2000)
			}
			this.modalVisible = false
			this.pendingDeleteVideo = null
		},
	},

	mounted() {
		this.startAutoPlay()
	},
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

.app {
	width: 100%;
	height: 100vh;
	background: #f4f5f7;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	padding-bottom: 100rpx;
	box-sizing: border-box;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Helvetica Neue", Arial, sans-serif;
}

.category-scroll {
	background: #fff;
	padding: 10px 8px;
	white-space: nowrap;
	flex-shrink: 0;
	border-bottom: 1px solid #e3e5e7;
	width: 100%;
}

.category-scroll .chip {
	display: inline-block;
	padding: 7px 18px;
	font-size: 14px;
	color: #9499a0;
	border-radius: 20px;
	transition: all 0.2s;
	margin: 0 2px;
	flex-shrink: 0;
}

.category-scroll .chip.active {
	color: #fb7299;
	font-weight: 600;
	background: #fce8ef;
}

.feed {
	flex: 1;
	overflow-y: auto;
}

/* 轮播图 */
.banner-wrapper { padding: 0 12px; }
.banner-carousel {
	position: relative; width: 100%; height: 150px;
	overflow: hidden; border-radius: 12px; background: #e8e8e8; margin-top: 8px;
}
@media (min-width: 769px) { .banner-carousel { height: 240px; border-radius: 14px; } }
@media (min-width: 1025px) { .banner-carousel { height: 320px; border-radius: 16px; max-width: 1000px; margin: 12px auto 0; } }
.banner-track { display: flex; width: 100%; height: 100%; transition: transform 0.5s ease; }
.banner-slide { flex: 0 0 100%; height: 100%; position: relative; }
.banner-slide .banner-image { width: 100%; height: 100%; }
.banner-slide .banner-overlay {
	position: absolute; bottom: 0; left: 0; right: 0;
	padding: 16px 20px; background: linear-gradient(to top, rgba(0,0,0,0.7), transparent);
}
.banner-slide .banner-title { color: #fff; font-size: 14px; font-weight: 600; }
.banner-dots { position: absolute; bottom: 12px; left: 50%; transform: translateX(-50%); display: flex; gap: 8px; z-index: 2; }
.banner-dots .dot { width: 6px; height: 6px; border-radius: 50%; background: rgba(255,255,255,0.4); transition: all 0.3s; }
.banner-dots .dot.active { background: #fb7299; width: 20px; border-radius: 4px; }
.banner-btn {
	position: absolute; top: 50%; transform: translateY(-50%);
	width: 32px; height: 32px; border: none; border-radius: 50%;
	background: rgba(0,0,0,0.4); color: #fff;
	display: flex; align-items: center; justify-content: center;
	z-index: 3; opacity: 0; transition: opacity 0.3s;
}
.banner-carousel:active .banner-btn { opacity: 1; }
.banner-btn-prev { left: 10px; }
.banner-btn-next { right: 10px; }

/* 视频列表 —— 两列网格，封面在上、详情在下 */
.video-list {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 16rpx;
	padding: 16rpx 16rpx;
}
.video-card {
	display: flex;
	flex-direction: column;
	background: #fff;
	border-radius: 12rpx;
	overflow: hidden;
	position: relative;
	transition: all 0.3s ease;
}
.video-card:active { background: #f9f9fa; transform: scale(0.97); }
.video-card .thumb {
	width: 100%;
	padding-top: 56.25%;
	position: relative;
	overflow: hidden;
	background: #e0e0e0;
}
.video-card .thumb .thumb-image {
	position: absolute;
	top: 0; left: 0;
	width: 100%; height: 100%;
}
.video-card .thumb .duration {
	position: absolute; bottom: 8rpx; right: 8rpx;
	background: rgba(0,0,0,0.7); color: #fff; font-size: 11px; padding: 1px 6px; border-radius: 3px;
}
.video-card .thumb .play-badge {
	position: absolute; bottom: 8rpx; left: 8rpx;
	background: rgba(0,0,0,0.65); color: #fff; font-size: 10px;
	padding: 2px 8px; border-radius: 3px; display: flex; align-items: center; gap: 4px;
}
.video-card .info {
	flex: 1;
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	min-width: 0;
	padding: 16rpx 20rpx 20rpx;
}
.video-card .info .title {
	font-size: 13px; font-weight: 500; color: #18191c; line-height: 1.4;
	display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;
	margin-bottom: 8rpx;
}
.video-card .info .meta {
	font-size: 11px; color: #9499a0;
	display: flex; flex-direction: column; gap: 4rpx;
}
.video-card .info .meta .stats { display: flex; gap: 12rpx; }
.video-card .delete-btn {
	position: absolute; top: 8rpx; right: 8rpx; width: 36rpx; height: 36rpx;
	border: none; background: rgba(0,0,0,0.35); color: #fff;
	font-size: 20rpx; border-radius: 50%; display: flex;
	align-items: center; justify-content: center; z-index: 2;
}
.video-card .delete-btn:active { background: #ff4d4f; color: #fff; }

/* 平板 —— 间隙略大 */
@media (min-width: 769px) {
	.video-list { gap: 20rpx; padding: 20rpx 32rpx; }
	.video-card .info { padding: 20rpx 24rpx 24rpx; }
	.video-card .info .title { font-size: 14px; }
}
/* 桌面 —— 三列 */
@media (min-width: 1100px) {
	.video-list { grid-template-columns: repeat(3, 1fr); padding: 24rpx 40rpx; }
}

/* 返回顶部 */
.back-to-top { position: fixed;
  right: 24px; bottom: 96px; width: 40px; height: 40px; border-radius: 50%;
  background: #fff; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; z-index: 200;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); border: none;
}
.back-to-top:hover { transform: translateY(-2px); box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15); }
.back-to-top:active { transform: translateY(0) scale(0.95); box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); }

/* 空状态 */
.empty-state { text-align: center; padding: 60px 20px; color: #9499a0; grid-column: 1 / -1; }
.empty-state .empty-icon { font-size: 56px; display: block; margin-bottom: 12px; }
.empty-state .empty-title { font-size: 16px; font-weight: 500; color: #18191c; display: block; }
.empty-state .empty-desc { font-size: 13px; color: #b0b5bc; display: block; margin-top: 4px; }

/* 弹窗 */
.modal-overlay {
	position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 999;
	display: flex; align-items: center; justify-content: center;
	opacity: 0; pointer-events: none; transition: opacity 0.3s ease;
}
.modal-overlay.show { opacity: 1; pointer-events: auto; }
.modal-box {
	background: #fff; border-radius: 16px; padding: 28px 24px 20px;
	max-width: 320px; width: 90%; transform: scale(0.9);
	transition: transform 0.3s ease; text-align: center;
}
.modal-overlay.show .modal-box { transform: scale(1); }
.modal-box .modal-icon { font-size: 28px; display: block; margin-bottom: 8px; }
.modal-box .modal-title { font-size: 17px; font-weight: 600; color: #18191c; margin-bottom: 6px; }
.modal-box .modal-desc { font-size: 14px; color: #9499a0; margin-bottom: 20px; display: block; }
.modal-box .modal-actions { display: flex; gap: 12px; }
.modal-box .modal-actions button { flex: 1; padding: 10px 0; border: none; border-radius: 10px; font-size: 15px; font-weight: 500; }
.modal-box .modal-cancel { background: #f1f2f5; color: #61666d; }
.modal-box .modal-confirm { background: #fb7299; color: #fff; }

.toast {
	position: fixed; top: 50%; left: 50%;
	transform: translate(-50%, -50%) scale(0.9);
	background: rgba(0,0,0,0.78); color: #fff;
	padding: 20px 32px; border-radius: 12px; font-size: 15px;
	z-index: 1000; opacity: 0; pointer-events: none;
	transition: all 0.3s ease; text-align: center;
}
.toast.show { opacity: 1; transform: translate(-50%, -50%) scale(1); }
.toast .toast-icon { font-size: 32px; display: block; margin-bottom: 8px; }
</style>
