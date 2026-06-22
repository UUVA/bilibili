<template>
  <view class="page-container">
    <!-- 顶部导航 -->
    <view class="detail-header">
      <view class="back-btn" @click="goBack">‹</view>
      <view class="detail-title">{{ video.title || '视频详情' }}</view>
    </view>

    <view class="player-area">
      <!-- #ifdef H5 -->
      <iframe
        :src="playerUrl"
        scrolling="no"
        border="0"
        frameborder="no"
        framespacing="0"
        allowfullscreen="true"
        class="bili-player"
      ></iframe>
      <!-- #endif -->
      
      <!-- #ifndef H5 -->
      <web-view :src="playerUrl"></web-view>
      <!-- #endif -->
    </view>

    <!-- 数据 -->
    <view class="detail-stats">
      <text>▶ {{ video.views || '0' }} 播放</text>
      <text>💬 {{ video.danmaku || '0' }} 弹幕</text>
      <text>{{ video.duration || '00:00' }}</text>
    </view>

    <!-- UP主 -->
    <view class="up-section">
      <view class="up-left">
        <view class="up-avatar">
          <image :src="video.thumb" mode="aspectFill" />
        </view>
        <text class="up-name">{{ video.author || 'UP主' }}</text>
      </view>
      <view 
        class="follow-btn" 
        :class="{ followed: isFollowed }"
        @click="toggleFollow"
      >
        {{ isFollowed ? '已关注' : '+ 关注' }}
      </view>
    </view>

    <!-- 互动按钮 -->
    <view class="action-row">
      <view class="action-item" @click="handleLike">
        <svg class="action-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3H14z"/>
          <path d="M7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"/>
        </svg>
        <text>点赞</text>
      </view>
      
      <view class="action-item" @click="handleCoin">
        <svg class="action-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="9"/>
          <path d="M12 8v8"/>
          <path d="M9.5 10.5c0-.8.5-1.5 1.5-1.5s1.5.7 1.5 1.5-.5 1.5-1.5 1.5c-.8 0-1.5.5-1.5 1.5s.5 1.5 1.5 1.5 1.5-.7 1.5-1.5"/>
        </svg>
        <text>投币</text>
      </view>
      
      <view class="action-item" @click="handleCollect">
        <svg class="action-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/>
        </svg>
        <text>收藏</text>
      </view>
      
      <view class="action-item" @click="showShare = true">
        <svg class="action-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="18" cy="5" r="3"/>
          <circle cx="6" cy="12" r="3"/>
          <circle cx="18" cy="19" r="3"/>
          <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"/>
          <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"/>
        </svg>
        <text>分享</text>
      </view>
    </view>

    <!-- 标签 -->
    <view class="tags" v-if="video.tags && video.tags.length">
      <text class="tag" v-for="tag in video.tags" :key="tag">#{{ tag }}</text>
    </view>

    <!-- 简介 -->
    <view class="desc-text">{{ video.desc }}</view>

    <!-- 相关推荐 -->
    <view class="related-section">
      <view class="related-title">相关推荐</view>
      <view class="related-list">
        <view 
          class="related-item" 
          v-for="(item, index) in relatedVideos" 
          :key="index"
          @click="goDetail(item)"
        >
          <view class="related-cover">
            <image :src="item.thumb" mode="aspectFill" />
            <view class="related-duration">{{ item.duration }}</view>
          </view>
          <view class="related-info">
            <view class="related-video-title">{{ item.title }}</view>
            <view class="related-author">👤 {{ item.author }}</view>
            <view class="related-stats">▶ {{ item.views }}  💬 {{ item.danmaku }}</view>
          </view>
        </view>
      </view>
    </view>

    <!-- 分享弹窗 -->
    <view class="share-mask" v-if="showShare" @click="showShare = false">
      <view class="share-panel" @click.stop>
        <view class="share-title">分享到</view>
        <view class="share-options">
          <view class="share-item" @click="shareTo('微信')">
            <view class="share-icon wechat">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M20.25 8.511c.884.284 1.5 1.128 1.5 2.097v4.286c0 1.136-.847 2.1-1.98 2.193-.34.027-.68.052-1.02.072v3.091l-3-3c-1.354 0-2.694-.055-4.02-.163a2.115 2.115 0 0 1-.825-.242m9.345-8.334a2.126 2.126 0 0 0-.476-.095 48.64 48.64 0 0 0-8.048 0c-1.131.094-1.976 1.057-1.976 2.192v4.286c0 .837.46 1.58 1.155 1.951m9.345-8.334V6.637c0-1.621-1.152-3.026-2.76-3.235A48.455 48.455 0 0 0 11.25 3c-2.115 0-4.198.137-6.24.402-1.608.209-2.76 1.614-2.76 3.235v6.226c0 1.621 1.152 3.026 2.76 3.235.577.075 1.157.14 1.74.194V21l4.155-4.155" />
              </svg>
            </view>
            <text>微信</text>
          </view>
          <view class="share-item" @click="shareTo('微博')">
            <view class="share-icon weibo">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 15.75V18m-7.5-6.75h.008v.008H8.25v-.008Zm0 2.25h.008v.008H8.25V13.5Zm0 2.25h.008v.008H8.25v-.008Zm0 2.25h.008v.008H8.25V18Zm2.498-6.75h.007v.008h-.007v-.008Zm0 2.25h.007v.008h-.007V13.5Zm0 2.25h.007v.008h-.007v-.008Zm0 2.25h.007v.008h-.007V18Zm2.504-6.75h.008v.008h-.008v-.008Zm0 2.25h.008v.008h-.008V13.5Zm0 2.25h.008v.008h-.008v-.008Zm0 2.25h.008v.008h-.008V18Zm2.498-6.75h.008v.008h-.008v-.008Zm0 2.25h.008v.008h-.008V13.5ZM8.25 6h7.5v2.25h-7.5V6ZM12 2.25c-1.892 0-3.758.11-5.593.322C5.307 2.7 4.5 3.65 4.5 4.757V19.5a2.25 2.25 0 0 0 2.25 2.25h10.5a2.25 2.25 0 0 0 2.25-2.25V4.757c0-1.108-.806-2.057-1.907-2.185A48.507 48.507 0 0 0 12 2.25Z" />
              </svg>
            </view>
            <text>微博</text>
          </view>
          <view class="share-item" @click="shareTo('QQ')">
            <view class="share-icon qq">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M18 18.72a9.094 9.094 0 0 0 3.741-.479 3 3 0 0 0-4.682-2.72m.94 3.198.001.031c0 .225-.012.447-.037.666A11.944 11.944 0 0 1 12 21c-2.17 0-4.207-.576-5.963-1.584A6.062 6.062 0 0 1 6 18.719m12 0a5.971 5.971 0 0 0-.941-3.197m0 0A5.995 5.995 0 0 0 12 12.75a5.995 5.995 0 0 0-5.058 2.772m0 0a3 3 0 0 0-4.681 2.72 8.986 8.986 0 0 0 3.74.477m.94-3.197a5.971 5.971 0 0 0-.94 3.197M15 6.75a3 3 0 1 1-6 0 3 3 0 0 1 6 0Zm6 3a2.25 2.25 0 1 1-4.5 0 2.25 2.25 0 0 1 4.5 0Zm-13.5 0a2.25 2.25 0 1 1-4.5 0 2.25 2.25 0 0 1 4.5 0Z" />
              </svg>
            </view>
            <text>QQ</text>
          </view>
          <view class="share-item" @click="shareTo('复制链接')">
            <view class="share-icon link">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="m18.375 12.739-7.693 7.693a4.5 4.5 0 0 1-6.364-6.364l10.94-10.94A3 3 0 1 1 19.5 7.372L8.552 18.32m.009-.01-.01.01m5.699-9.941-7.81 7.81a1.5 1.5 0 0 0 2.112 2.13" />
              </svg>
            </view>
            <text>复制链接</text>
          </view>
        </view>
        <view class="share-cancel" @click="showShare = false">取消</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      video: {},
      isFollowed: false,
      showShare: false,
      relatedVideos: [
        {title:'Blender原创动画 | 工程展示 | 第二弹',author:'Lan_er2',duration:'5:12',views:'156.3万',danmaku:'423',thumb:'https://images.weserv.nl/?url=i1.hdslb.com/bfs/archive/e5fa56ef7c04180134210420e575b9913adab385.jpg',bvid:'BV1wEEg62EDP',desc:'Blender原创3D动画第二弹，更多精彩。',tags:['Blender','3D','动画']},
        {title:'【Blender】超写实角色制作全流程',author:'CG_Panda',duration:'23:40',views:'89.7万',danmaku:'1.2千',thumb:'https://images.weserv.nl/?url=i2.hdslb.com/bfs/archive/d47343df294c1ba36c3329bbe3d1dd954aff0de3.jpg',bvid:'BV1PnV46DEP4',desc:'从零开始制作超写实角色。',tags:['Blender','角色','教程']},
        {title:'Blender 4.0 新功能全面解析',author:'技术美术小课堂',duration:'15:20',views:'45.2万',danmaku:'568',thumb:'https://images.weserv.nl/?url=i1.hdslb.com/bfs/archive/ba06201e7786575aaf3b77304c7f74e9c958eb05.jpg',bvid:'BV1Wzjg65EM6',desc:'Blender 4.0所有新功能详解。',tags:['Blender','教程','4.0']},
        {title:'用Blender做出《流浪地球》级特效',author:'VFX_Studio',duration:'12:08',views:'234.1万',danmaku:'3.2千',thumb:'https://images.weserv.nl/?url=i2.hdslb.com/bfs/archive/0dd7257472d03c50150cd38ff7efb425c8cfd40a.jpg',bvid:'BV1QNJu6bEts',desc:'Blender特效制作全流程揭秘。',tags:['Blender','特效','VFX']}
      ]
    };
  },
  computed: {
    playerUrl() {
      if (!this.video.bvid) return '';
      return `https://player.bilibili.com/player.html?bvid=${this.video.bvid}&high_quality=1&danmaku=1&autoplay=0`;
    }
  },
  onLoad(options) {
    this.video = {
      bvid: options.bvid || '',
      title: decodeURIComponent(options.title || ''),
      author: decodeURIComponent(options.author || ''),
      views: options.views || '0',
      danmaku: options.danmaku || '0',
      duration: options.duration || '00:00',
      thumb: decodeURIComponent(options.thumb || ''),
      desc: decodeURIComponent(options.desc || ''),
      tags: options.tags ? JSON.parse(decodeURIComponent(options.tags)) : []
    };
    const key = `follow_${this.video.bvid}`;
    this.isFollowed = uni.getStorageSync(key) || false;
  },
  methods: {
    goBack() { uni.navigateBack(); },
    goDetail(video) {
      uni.navigateTo({
        url: `/pages/detail/detail?bvid=${video.bvid}&title=${encodeURIComponent(video.title)}&author=${encodeURIComponent(video.author)}&views=${video.views}&danmaku=${video.danmaku}&duration=${video.duration}&thumb=${encodeURIComponent(video.thumb)}&desc=${encodeURIComponent(video.desc || '')}&tags=${encodeURIComponent(JSON.stringify(video.tags || []))}`
      });
    },
    toggleFollow() {
      this.isFollowed = !this.isFollowed;
      const key = `follow_${this.video.bvid}`;
      uni.setStorageSync(key, this.isFollowed);
      uni.showToast({ 
        title: this.isFollowed ? '已关注' : '已取消关注', 
        icon: 'none', 
        duration: 800 
      });
    },
    handleLike() { uni.showToast({ title: '已点赞', icon: 'none', duration: 800 }); },
    handleCoin() { uni.showToast({ title: '已投币', icon: 'none', duration: 800 }); },
    handleCollect() { uni.showToast({ title: '已收藏', icon: 'none', duration: 800 }); },
    shareTo(platform) {
      this.showShare = false;
      if (platform === '复制链接') {
        const url = `https://www.bilibili.com/video/${this.video.bvid}`;
        uni.setClipboardData({
          data: url,
          success: () => {
            uni.showToast({ title: '已复制视频链接', icon: 'none', duration: 1000 });
          }
        });
      } else {
        uni.showToast({ title: `分享到${platform}`, icon: 'none', duration: 800 });
      }
    }
  }
};
</script>

<style scoped>
.page-container { 
  width: 100%; 
  min-height: 100vh; 
  background: #f5f5f5; 
  border-radius: 16px 16px 0 0; 
 
}

.detail-header { 
  display: flex; 
  align-items: center; 
  padding: 12px 16px; 
  background: #fff; 
  gap: 8px; 
}
.back-btn { 
  font-size: 28px; 
  line-height: 1; 
  color: #222; 
  padding-right: 6px; 
  cursor: pointer; 
}
.detail-header .detail-title { 
  font-size: 16px; 
  font-weight: 500; 
  flex: 1; 
  white-space: nowrap; 
  overflow: hidden; 
  text-overflow: ellipsis; 
}

.player-area { 
  width: 100%; 
  background: #000; 
  aspect-ratio: 16/9; 
  position: relative; 
  overflow: hidden; 
.bili-player { width: 100%; height: 100%; border: none; }
}

.detail-stats { 
  display: flex; 
  gap: 24px; 
  padding: 12px 16px; 
  font-size: 13px; 
  color: #555; 
  background: #fff;
  flex-wrap: wrap; 
}

.up-section { 
  display: flex; 
  align-items: center; 
  justify-content: space-between; 
  padding: 12px 16px; 
  background: #fff;
}
.up-left { display: flex; align-items: center; gap: 12px; }
.up-avatar { 
  width: 40px; 
  height: 40px; 
  border-radius: 50%; 
  background: #d0d0d0; 
  overflow: hidden; 
}
.up-avatar image { width: 100%; height: 100%; }
.up-name { font-weight: 500; font-size: 14px; }

.follow-btn { 
  background: #fb7299; 
  color: #fff; 
  padding: 5px 14px; 
  border-radius: 20px; 
  font-size: 12px; 
  font-weight: 500; 
  cursor: pointer; 
  transition: all 0.3s ease;
}
.follow-btn.followed {
  background: #e8e8e8;
  color: #999;
}
.follow-btn:active {
  transform: scale(0.92);
}

.action-row { 
  display: flex; 
  padding: 12px 16px; 
  background: #fff;
  justify-content: space-around; 
  align-items: center; 
}
.action-item { 
  display: flex; 
  flex-direction: column; 
  align-items: center; 
  gap: 4px; 
  font-size: 11px; 
  color: #666; 
  cursor: pointer; 
  padding: 4px 10px; 
  border-radius: 8px; 
}
.action-item:active { transform: scale(0.92); background: #f5f5f5; }
.action-icon { width: 22px; height: 22px; stroke: #666; }

.tags { 
  display: flex; 
  gap: 8px; 
  padding: 10px 16px;  
  background: #fff;
  flex-wrap: wrap; 
}
.tag { 
  background: #e8f4ff; 
  color: #1c7ed6; 
  padding: 2px 12px; 
  border-radius: 20px; 
  font-size: 12px; 
}

.desc-text { 
  padding: 12px 16px; 
  font-size: 14px; 
  color: #333; 
  line-height: 1.6; 
  background: #fff;
}

/* 相关推荐 */
.related-section {
  background: #fff;
  padding: 12px 16px;;
}

.related-title {
  font-size: 15px;
  font-weight: 600;
  color: #222;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.related-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.related-item {
  display: flex;
  gap: 10px;
  cursor: pointer;
  padding: 6px;
  border-radius: 8px;
  transition: background 0.2s ease;
}
.related-item:active {
  background: #f5f5f5;
}

.related-cover {
  position: relative;
  flex-shrink: 0;
  width: 120px;
  height: 68px;
  border-radius: 6px;
  overflow: hidden;
  background: #e0e0e0;
}
.related-cover image {
  width: 100%;
  height: 100%;
}
.related-duration {
  position: absolute;
  bottom: 4px;
  right: 4px;
  background: rgba(0,0,0,0.7);
  color: #fff;
  font-size: 10px;
  padding: 0 5px;
  border-radius: 3px;
  line-height: 16px;
}

.related-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 2px 0;
}

.related-video-title {
  font-size: 13px;
  font-weight: 500;
  color: #222;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  line-height: 1.4;
}

.related-author {
  font-size: 11px;
  color: #888;
  margin-top: 2px;
}

.related-stats {
  font-size: 11px;
  color: #999;
  margin-top: 2px;
}

/* 分享弹窗 */
.share-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  animation: fadeIn 0.3s ease;
}

.share-panel {
  background: #fff;
  border-radius: 16px 16px 0 0;
  width: 100%;
  max-width: 500px;
  padding: 16px 16px 20px;
  animation: slideUp 0.3s ease;
}

.share-title {
  text-align: center;
  font-size: 14px;
  font-weight: 500;
  color: #888;
  padding-bottom: 14px;
  letter-spacing: 1px;
}

.share-options {
  display: flex;
  justify-content: space-around;
  padding-bottom: 14px;
}

.share-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  cursor: pointer;
}
.share-item:active { transform: scale(0.92); }

.share-icon {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #ddd;
  background: transparent !important;
}
.share-icon svg {
  width: 20px;
  height: 20px;
  stroke: #888;
}

.share-item:active .share-icon {
  border-color: #fb7299;
}
.share-item:active .share-icon svg {
  stroke: #fb7299;
}
.share-item:active text {
  color: #fb7299;
}

.share-item text {
  font-size: 11px;
  color: #999;
}

.share-cancel {
  text-align: center;
  padding: 10px 0 0;
  font-size: 14px;
  color: #999;
  border-top: 1px solid #f0f0f0;
  cursor: pointer;
  font-weight: 500;
}
.share-cancel:active { color: #fb7299; }

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}
</style>