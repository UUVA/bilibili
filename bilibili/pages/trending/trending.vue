<template>
  <view class="page-container">
    <!-- 头部 -->
    <view class="page-header">
      <view class="header-title">热门</view>
    </view>

    <!-- 标签栏 -->
    <view class="tabs">
      <view class="tab-item" :class="{ active: currentTab === 0 }" @click="switchInnerTab(0)">热门视频</view>
      <view class="tab-item" :class="{ active: currentTab === 1 }" @click="switchInnerTab(1)">每周必看</view>
      <view class="tab-item" :class="{ active: currentTab === 2 }" @click="switchInnerTab(2)">排行榜</view>
    </view>

    <scroll-view class="video-list-wrap" scroll-y="true" :scroll-top="scrollTopValue" :scroll-with-animation="true" @scroll="onVideoScroll">
      <view class="video-list" id="videoList">
        <view
          class="video-item"
          v-for="(v, index) in currentVideoList"
          :key="v.bvid"
          @click="goDetail(v)"
        >
          <view class="video-rank" :class="'rank-' + (index + 1)">{{ index + 1 }}</view>
          <view class="video-cover-wrap">
            <image :src="v.thumb" mode="aspectFill" class="cover-image" />
            <view class="video-duration">{{ v.duration }}</view>
          </view>
          <view class="video-info">
            <view class="video-title">{{ v.title }}</view>
            <view class="video-meta">
              <text class="meta-up">👤 {{ v.author }}</text>
              <text class="meta-plays">▶ {{ v.views }}</text>
              <text class="meta-danmaku">💬 {{ v.danmaku }}</text>
            </view>
          </view>
        </view>
      </view>
    </scroll-view>

    <view class="back-to-top" v-show="showBackTop" @click="scrollToTop">
      <svg class="back-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
        <polyline points="18 15 12 9 6 15"/>
      </svg>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      currentTab: 0,
      showBackTop: false,
      scrollTopValue: 0,
      hotVideos: [
        {title:'《绝区零》维琳娜角色PV | 拟剧论',author:'绝区零',duration:'3:28',views:'32.2万',danmaku:'2.1千',thumb:'https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/archive/eb6d362a0b2b68ee2ea6a04110dca848b80c5258.jpg',bvid:'BV1mrjN6qE4C',desc:'全新S级代理人维琳娜登场！',tags:['绝区零','角色PV','维琳娜']},
        {title:'日本烤羊自助！来自蒙古可汗，到底好吃吗',author:'平平出击',duration:'11:03',views:'14.9万',danmaku:'329',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/e158a84e40df88d6c378d1f13a25c6bc0d9ef12a.jpg',bvid:'BV1U4jg6pEZS',desc:'在日本吃蒙古烤羊自助',tags:['美食','探店','日本']},
        {title:'《盛世天下》女帝篇，权力感情我全都要',author:'与山0v0',duration:'3:50:39',views:'9.2万',danmaku:'932',thumb:'https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/archive/3ed66ca992ca029cc14d16524da84be109f67fc4.jpg',bvid:'BV1JvjP6XEHA',desc:'沉浸式互动影游',tags:['互动影游','盛世天下','女帝']},
        {title:'挑衅观众、忘掉初心、看到力竭',author:'话很多的小姐姐们',duration:'17:18',views:'13.4万',danmaku:'1.3千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/364ef2f044b1bc583656adf17e26137ea0374db2.jpg',bvid:'BV1Y2J36uEuS',desc:'聊聊最近那些综艺节目',tags:['综艺','吐槽','娱乐']},
        {title:'【无畏契约伦敦大师赛】6月15日 EDG vs XLG',author:'无畏契约赛事',duration:'3:16:45',views:'66.9万',danmaku:'3.1千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/8e22dcb63cceb32e798a53601b5345ae17e3536d.jpg',bvid:'BV1KwjN6NEmy',desc:'2026无畏契约伦敦大师赛',tags:['无畏契约','电竞','大师赛']},
        {title:'我们做了个不一样的拐杖...',author:'老师好我叫何同学',duration:'7:00',views:'125.7万',danmaku:'1.8千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/eba34b3f270343f2e9d00cfe39017bc85914227f.jpg',bvid:'BV17TEC6AE3Z',desc:'何同学最新创意发明',tags:['何同学','创意','发明']},
        {title:'和阎鹤祥在甘肃武威，放最野的羊，吃最夯的肉',author:'食贫道',duration:'52:07',views:'86.5万',danmaku:'4.8千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/eea97a1bfd30701c84e709b53644e243f3e399c8.jpg',bvid:'BV1GWJg6KECu',desc:'食贫道甘肃武威之旅',tags:['美食','甘肃','食贫道']},
        {title:'《安宁大厦生存守则》第十集',author:'Fel探险记',duration:'5:49',views:'77.0万',danmaku:'826',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/40f87281a0032e4846d825714689cbc0e1fc8771.jpg',bvid:'BV1GgJV6kEWd',desc:'安宁大厦的恐怖故事',tags:['恐怖','短剧','悬疑']},
        {title:'【影Sir】生死对赌！富豪壕掷1000亿',author:'我是影Sir',duration:'17:05',views:'24.5万',danmaku:'808',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/0a439b319c602673a38abe37d5e0e39252cb5109.jpg',bvid:'BV1jbEd6iEY7',desc:'一口气看完高智商漫改剧',tags:['影Sir','漫改','解说']},
        {title:'一口气看完《猫头鹰谋杀案》',author:'谁是阿WAI',duration:'2:18:32',views:'33.2万',danmaku:'377',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/ac0565e10f1d661926d5ef0172e7e337d2b90dfd.jpg',bvid:'BV1cNVU63Ep9',desc:'耗时一个月爆肝6万字',tags:['推理','解说','悬疑']}
      ],
      weeklyVideos: [
        {title:'《归唐》19分钟完整实机演示',author:'归唐',duration:'19:27',views:'1092.8万',danmaku:'2.6万',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/73972f02b3d2a52c97a1ec81e53403fef6160006.jpg',bvid:'BV1Uj7k6VEPU',desc:'国产单机大作归唐',tags:['归唐','单机','实机演示']},
        {title:'《伊莫》握爪测试招募开启',author:'伊莫',duration:'1:59',views:'533.5万',danmaku:'806',thumb:'https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/archive/06bcb84e42974283a3ade3bf6fbee2302b4b4d96.jpg',bvid:'BV1CQ7o61ERo',desc:'治愈系游戏伊莫',tags:['伊莫','治愈','独立游戏']},
        {title:'菲比：就你们寝室最吵！！',author:'Lm海洋',duration:'4:22',views:'205.2万',danmaku:'1.1千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/348cd3608f962369b093721ee90e2515a127a833.jpg',bvid:'BV1xR7S6hEkB',desc:'原创动画短片',tags:['动画','原创','搞笑']},
        {title:'Blender原创动画 | 工程展示',author:'Lan_er2',duration:'4:41',views:'237.9万',danmaku:'696',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/e5fa56ef7c04180134210420e575b9913adab385.jpg',bvid:'BV1wEEg62EDP',desc:'Blender原创3D动画',tags:['Blender','3D','动画']},
        {title:'我读了你们的评论 把阎王召唤出来了',author:'汤木檀泽',duration:'15:59',views:'253.0万',danmaku:'1.8千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/0b6c7ccb214b6211a4cf9a8741eb473536e42587.jpg',bvid:'BV1JKEw6QEw9',desc:'不小心召唤出了阎王',tags:['搞笑','互动','粉丝']},
        {title:'当我把床改成了一台赛车模拟器...',author:'小狮日记',duration:'5:23',views:'563.4万',danmaku:'1.6千',thumb:'https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/archive/f20b14f142bb84ca7992c88650351d9f817175df.jpg',bvid:'BV1tpEn6XE8N',desc:'把床变成赛车模拟器',tags:['硬核','改造','赛车']},
        {title:'世界杯期间不抓移民？',author:'麻薯波比呀',duration:'16:35',views:'277.4万',danmaku:'9.4千',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/91940674117acf43b95f7a6e2e7ab686470cc3ea.jpg',bvid:'BV15d7D6UE4i',desc:'2026世界杯时事评论',tags:['世界杯','时事','评论']},
        {title:'在AI里抛硬币，概率是50%吗？',author:'影视飓风',duration:'10:17',views:'100.0万',danmaku:'1.2千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/ba06201e7786575aaf3b77304c7f74e9c958eb05.jpg',bvid:'BV1Wzjg65EM6',desc:'AI抛硬币的概率',tags:['AI','概率','科普']},
        {title:'IShowSpeed - World Cup',author:'ishowspeedsui',duration:'4:59',views:'695.6万',danmaku:'3.9千',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/2807f6a9b9342740dfa8764e7bcc6e231196c94f.jpg',bvid:'BV1jwV663EjR',desc:'世界杯主题曲MV',tags:['世界杯','音乐','MV']},
        {title:'《崩坏：星穹铁道》千冶刃角色PV',author:'崩坏星穹铁道',duration:'7:12',views:'307.2万',danmaku:'2.8万',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/d47343df294c1ba36c3329bbe3d1dd954aff0de3.jpg',bvid:'BV1PnV46DEP4',desc:'千冶刃角色PV公开',tags:['星穹铁道','角色PV','千冶']}
      ],
      rankVideos: [
        {title:'峰峦叠嶂间，此城何处寻？',author:'鸣潮',duration:'7:57',views:'274.8万',danmaku:'1.2万',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/0dd7257472d03c50150cd38ff7efb425c8cfd40a.jpg',bvid:'BV1QNJu6bEts',desc:'鸣潮新版本剧情PV',tags:['鸣潮','剧情','PV']},
        {title:'中国的危与机？中美俄欧中东',author:'王骁Albert',duration:'2:18:15',views:'76.8万',danmaku:'1.8万',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/8fdde637aaee057b244a01cd6931a1b0c253714f.jpg',bvid:'BV1UFJN6jEZz',desc:'深度解读当前国际局势',tags:['国际','政治','深度']},
        {title:'花30天用巧克力手搓《清明上河图》',author:'范苏木',duration:'5:18',views:'707.7万',danmaku:'2.1万',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/1e30828ba6af97fb712de0461bc920f3fe3346be.jpg',bvid:'BV16YGC6XEco',desc:'用巧克力复刻清明上河图',tags:['手工','巧克力','艺术']},
        {title:'全世界笑点下降1000倍而我不变',author:'逗比的雀巢',duration:'8:42',views:'1581.7万',danmaku:'3.0万',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/534373c87b367e799685021bc35be620b5e9f1f4.jpg',bvid:'BV1K3Gz6pEoo',desc:'全世界笑点下降1000倍',tags:['搞笑','创意','短剧']},
        {title:'《绝区零》星徽比利EP | Billy Mode',author:'绝区零',duration:'12:57',views:'323.4万',danmaku:'1.6万',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/f75ecc231c8aaf6b0a3588621c13d73c2462ad74.jpg',bvid:'BV16xVw6gEgu',desc:'星徽比利角色EP',tags:['绝区零','EP','比利']},
        {title:'姐要往你眼里撒点灰',author:'Blood_Raven',duration:'2:20',views:'170.9万',danmaku:'759',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/c4824ff1cfc43d11172e32395f6bf532dae48bf7.jpg',bvid:'BV1z19mBvE6k',desc:'原创动画短片',tags:['动画','原创','短片']},
        {title:'《赛博朋克 2077》PS5 Pro更新',author:'CD_PROJEKT',duration:'1:11',views:'11.2万',danmaku:'45',thumb:'https://images.weserv.nl/?url=https://i2.hdslb.com/bfs/archive/33f5d4450215b154db72c892b11b6fa5e2b51b6e.jpg',bvid:'BV1f2D7BrEJj',desc:'赛博朋克2077 PS5 Pro',tags:['赛博朋克','PS5','更新']},
        {title:'《吉时已到》首支预告片',author:'上海烛龙',duration:'3:44',views:'396.8万',danmaku:'1.1万',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/654ee653dddcd02e87d8d84cf134f1613409ea57.jpg',bvid:'BV1z8o5BEEo9',desc:'上海烛龙新作',tags:['吉时已到','上海烛龙','预告']},
        {title:'《古剑》首支预告片',author:'古剑',duration:'6:05',views:'1321.2万',danmaku:'7.4万',thumb:'https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/archive/de225dd0f2574979924a649139c6403c818065ae.jpg',bvid:'BV1WgYYzhEKw',desc:'上海烛龙单机新作古剑',tags:['古剑','上海烛龙','单机']},
        {title:'《遗忘之海》剧情概念PV 余烬',author:'遗忘之海',duration:'2:25',views:'677.3万',danmaku:'3.1千',thumb:'https://images.weserv.nl/?url=https://i1.hdslb.com/bfs/archive/aecfc194245a6b746b7770a94bb44eeb5634c648.jpg',bvid:'BV17Y9UBYEPd',desc:'遗忘之海测试招募开启',tags:['遗忘之海','PV','测试招募']}
      ]
    };
  },
  computed: {
    currentVideoList() {
      if (this.currentTab === 0) return this.hotVideos;
      if (this.currentTab === 1) return this.weeklyVideos;
      return this.rankVideos;
    }
  },
  methods: {
    onVideoScroll(e) {
      this.showBackTop = e.detail.scrollTop > 300;
    },
    switchInnerTab(index) {
      this.currentTab = index;
      this.scrollToTop();
    },
    goDetail(video) {
      var t = video.title || '';
      var a = video.author || '';
      var bv = video.bvid || '';
      var vs = video.views || '0';
      var dm = video.danmaku || '0';
      var dr = video.duration || '00:00';
      var th = video.thumb || '';
      var dc = video.desc || '';
      var tg = JSON.stringify(video.tags || []);
      var url = '/pages/detail/detail?bvid=' + encodeURIComponent(bv)
        + '&title=' + encodeURIComponent(t)
        + '&author=' + encodeURIComponent(a)
        + '&views=' + encodeURIComponent(vs)
        + '&danmaku=' + encodeURIComponent(dm)
        + '&duration=' + encodeURIComponent(dr)
        + '&thumb=' + encodeURIComponent(th)
        + '&desc=' + encodeURIComponent(dc)
        + '&tags=' + encodeURIComponent(tg);
      uni.navigateTo({ url: url });
    },
    scrollToTop() {
      this.scrollTopValue = this.scrollTopValue === 0 ? 1 : 0;
      this.showBackTop = false;
    }
  }
};
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

:deep(img), image {
  referrerpolicy: "no-referrer";
}

.page-container {
  width: 100%;
  height: 100vh;
  background: #f1f2f3;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding-bottom: 100rpx;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", "Microsoft YaHei", sans-serif;
}

/* 头部样式 - 与其他页面统一 */
.page-header {
  height: 96rpx;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  border-bottom: 1rpx solid #e3e5e7;
  flex-shrink: 0;
}

.header-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #18191c;
}

/* 标签栏 */
.tabs {
  display: flex;
  background: #fff;
  flex-shrink: 0;
  border-bottom: 1rpx solid #e3e5e7;
}

.tab-item {
  flex: 1;
  text-align: center;
  padding: 20rpx 0;
  font-size: 28rpx;
  color: #9499a0;
  position: relative;
  transition: color 0.2s;
}

.tab-item.active {
  color: #fb7299;
  font-weight: 600;
}

.tab-item.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 40rpx;
  height: 4rpx;
  background: #fb7299;
  border-radius: 2rpx;
}

/* 视频列表滚动区 */
.video-list-wrap {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
}

.video-list {
  padding: 12px 16px 30px;
}
.video-item { display: flex; padding: 14px 0; border-bottom: 1px solid #e8e8e8; gap: 12px; align-items: flex-start; cursor: pointer; }
.video-item:active { background: #ebebeb; }
.video-item:last-child { border-bottom: none; }

.video-rank { font-size: 16px; font-weight: 700; width: 32px; text-align: center; line-height: 1.4; color: #bbb; flex-shrink: 0; margin-top: 2px; font-style: italic; }
.video-item:nth-child(1) .video-rank,
.video-item:nth-child(2) .video-rank,
.video-item:nth-child(3) .video-rank { font-size: 24px; font-weight: 900; }
.rank-1 { color: #ff4d4d; }
.rank-2 { color: #ff8a2e; }
.rank-3 { color: #ffb347; }
.rank-4, .rank-5, .rank-6, .rank-7, .rank-8, .rank-9, .rank-10 { color: #bbb; }

.video-cover-wrap { position: relative; flex-shrink: 0; width: 120px; height: 75px; border-radius: 6px; overflow: hidden; background: #e0e0e0; }
.cover-image { width: 100%; height: 100%; }
.video-duration { position: absolute; bottom: 4px; right: 4px; background: rgba(0,0,0,0.7); color: #fff; font-size: 11px; padding: 0 5px; border-radius: 4px; line-height: 18px; }

.video-info { flex: 1; min-width: 0; }
.video-title { font-size: 13px; font-weight: 500; line-height: 1.4; color: #222; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; word-break: break-word; }
.video-meta { display: flex; align-items: center; gap: 6px; margin-top: 4px; font-size: 11px; color: #999; flex-wrap: wrap; }
.meta-up { color: #666; }
.meta-plays, .meta-danmaku { color: #999; }

.back-to-top { position: fixed;
  right: 24px; bottom: 96px; width: 40px; height: 40px; border-radius: 50%;
  background: #fff; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; z-index: 200;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); border: none;
}
.back-to-top:hover { transform: translateY(-2px); box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15); }
.back-to-top:active { transform: translateY(0) scale(0.95); box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); }
.back-icon { width: 20px; height: 20px; stroke: #666; transition: stroke 0.3s ease; }
.back-to-top:hover .back-icon { stroke: #fb7299; }
</style>
