<!--
  Refactored for uniapp App (xbuilder):
  - All div->view, span->text, img->image
  - No v-html, no onclick, no window.__vue_instance__
  - Subpages as static template blocks with v-if
  - CSS vars on page selector, dark mode via class
  - px->rpx, image paths to ../../mine_images/
-->
<template>
  <view class="ph" :class="{ 'dark-mode': S.settings.darkMode }">
    <!-- Toast -->
    <view class="toast" :class="{ show: toastVisible }"><text>{{ toastMessage }}</text></view>
    <view class="dlbar" :class="{ show: dlbarVisible }"><text>{{ dlbarMessage }}</text></view>

    <!-- 退出登录弹窗 -->
    <view class="mo" :class="{ s: logoutModalVisible }" @click="logoutModalVisible = false">
      <view class="md" @click.stop>
        <view class="mt"><text>退出登录</text></view>
        <view class="mdesc"><text>确定要退出当前账号吗？</text></view>
        <view class="mbs">
          <view class="bc" hover-class="bc-hover" @click.stop="logoutModalVisible = false"><text>取消</text></view>
          <view class="bf" hover-class="bf-hover" @click.stop="handleLogout"><text>退出</text></view>
        </view>
      </view>
    </view>

    <!-- 通用弹窗 -->
    <view class="mo" :class="{ s: generalModalVisible }" @click="closeGeneralModal">
      <view class="md" @click.stop>
        <!-- 头像选择器 -->
        <view v-if="generalModalType==='avatar-picker'" class="avatar-grid-wrap">
          <view class="avatar-grid">
            <view v-for="a in avatars" :key="a" class="avatar-opt" :class="{sel:a===S.av}" @click="selectAvatar(a)">
              <image v-if="a==='默认头像'" src="../../mine_images/avatar.jpg" class="avatar-img-small" mode="aspectFill"/>
              <text v-else class="avatar-emoji">{{a}}</text>
            </view>
          </view>
        </view>
        <!-- DND -->
        <view v-else-if="generalModalType==='dnd'">
          <view class="dnd-row">
            <input class="mi" v-model="dndStart" placeholder="开始时间" style="width:200rpx;text-align:center"/>
            <text style="margin:0 16rpx">—</text>
            <input class="mi" v-model="dndEnd" placeholder="结束时间" style="width:200rpx;text-align:center"/>
          </view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="saveDND"><text>确定</text></view>
          </view>
        </view>
        <!-- Change Pwd -->
        <view v-else-if="generalModalType==='change-pwd'">
          <input class="mi" v-model="oldPwd" password placeholder="当前密码"/>
          <input class="mi" v-model="newPwd" password placeholder="新密码(至少4位)"/>
          <input class="mi" v-model="newPwdConfirm" password placeholder="确认新密码"/>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="savePwd"><text>修改</text></view>
          </view>
        </view>
        <!-- VIP Purchase -->
        <view v-else-if="generalModalType==='vip-purchase'">
          <view class="mdesc"><text>{{vipTierName}} · ¥{{vipTierPrice}}</text></view>
          <view class="mdesc"><text>余额: ¥{{S.balance.toFixed(2)}} → ¥{{(S.balance-vipTierPrice).toFixed(2)}}</text></view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="confirmVipPurchase(vipTierName,vipTierPrice)"><text>确认 ¥{{vipTierPrice}}</text></view>
          </view>
        </view>
        <!-- VIP Recharge -->
        <view v-else-if="generalModalType==='vip-recharge'">
          <input class="mi" v-model="rechargeAmount" type="number" placeholder="充值金额"/>
          <view class="tag-chips">
            <text class="tag-chip" :class="{sel:rechargeAmount==='50'}" @click="rechargeAmount='50'">¥50</text>
            <text class="tag-chip" :class="{sel:rechargeAmount==='100'}" @click="rechargeAmount='100'">¥100</text>
            <text class="tag-chip" :class="{sel:rechargeAmount==='200'}" @click="rechargeAmount='200'">¥200</text>
          </view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="confirmRecharge"><text>充值</text></view>
          </view>
        </view>
        <!-- Wallet Recharge -->
        <view v-else-if="generalModalType==='wallet-recharge'">
          <input class="mi" v-model="walletRechargeAmount" type="number" placeholder="充值金额"/>
          <view class="tag-chips">
            <text class="tag-chip" :class="{sel:walletRechargeAmount==='50'}" @click="walletRechargeAmount='50'">¥50</text>
            <text class="tag-chip" :class="{sel:walletRechargeAmount==='100'}" @click="walletRechargeAmount='100'">¥100</text>
            <text class="tag-chip" :class="{sel:walletRechargeAmount==='200'}" @click="walletRechargeAmount='200'">¥200</text>
          </view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="confirmWalletRecharge"><text>充值</text></view>
          </view>
        </view>
        <!-- Add Fav -->
        <view v-else-if="generalModalType==='add-fav'">
          <input class="mi" v-model="newFavName" placeholder="收藏夹名称"/>
          <view class="fav-priv-row">
            <text class="fav-priv-opt" :class="{active:newFavPriv==='公开'}" @click="newFavPriv='公开'">公开</text>
            <text class="fav-priv-opt" :class="{active:newFavPriv==='私密'}" @click="newFavPriv='私密'">私密</text>
          </view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="confirmAddFav"><text>创建</text></view>
          </view>
        </view>
        <!-- Rename Fav -->
        <view v-else-if="generalModalType==='rename-fav'">
          <input class="mi" v-model="renameFavName" placeholder="新名称"/>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="confirmRenameFav(renameFavIdx)"><text>确定</text></view>
          </view>
        </view>
        <!-- Confirm Delete -->
        <view v-else-if="generalModalType==='confirm-delete'">
          <view class="mdesc"><text>{{deleteConfirmMsg}}</text></view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bf" hover-class="bf-hover" @click="executeDeleteAction"><text>删除</text></view>
          </view>
        </view>
        <!-- Cart -->
        <view v-else-if="generalModalType==='cart'">
          <view class="cart-header"><text>购物车 ({{S.cart.length}})</text></view>
          <view v-if="S.cart.length===0" class="mdesc"><text>购物车是空的</text></view>
          <view v-else>
            <view v-for="(c,i) in S.cart" :key="i" class="cart-item">
              <text class="cart-item-name">{{c.name}}</text>
              <text class="cart-item-price">¥{{c.price}}</text>
            </view>
            <view class="cart-total"><text>合计: ¥{{cartTotal}}</text></view>
          </view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>关闭</text></view>
            <view v-if="S.cart.length" class="bf" hover-class="bf-hover" @click="clearCart"><text>清空</text></view>
          </view>
        </view>
        <!-- Leave Message -->
        <view v-else-if="generalModalType==='leave-message'">
          <textarea class="mi mi-ta" v-model="msgContent" placeholder="描述问题..."/>
          <input class="mi" v-model="msgContact" placeholder="联系方式"/>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>取消</text></view>
            <view class="bp" hover-class="bp-hover" @click="submitMessage"><text>提交</text></view>
          </view>
        </view>
        <!-- 通用文本 -->
        <view v-else>
          <view class="mdesc"><text>{{generalModalContent}}</text></view>
          <view class="mbs">
            <view class="bc" hover-class="bc-hover" @click="closeGeneralModal"><text>关闭</text></view>
          </view>
        </view>
      </view>
    </view>

    <!-- ====== 主内容区 ====== -->
    <scroll-view class="ms" scroll-y :show-scrollbar="false" @scroll="handleScroll" :scroll-with-animation="true">
      <!-- 头部 -->
      <view class="pf" id="pfh">
        <view class="d1"/><view class="d2"/><view class="d3"/>
        <view class="pft">
          <view class="hact" hover-class="hact-hover" @click="openPage('scan')">
            <image src="../../mine_images/scan.png" class="hact-icon" mode="aspectFit"/>
          </view>
          <text class="header-title">{{T('mine')}}</text>
          <view class="hact" hover-class="hact-hover" @click="openPage('settings')">
            <image src="../../mine_images/settings.png" class="hact-icon-lg" mode="aspectFit"/>
            <view class="bdot"/>
          </view>
        </view>
        <!-- 用户信息 -->
        <view class="ur" @click="openPage('profile')">
          <view class="avw">
            <view class="av">
              <image v-if="S.av==='默认头像'" src="../../mine_images/avatar.jpg" class="av-img" mode="aspectFill"/>
              <text v-else class="av-text">{{S.av}}</text>
            </view>
            <view class="vring"/>
          </view>
          <view class="ui">
            <view class="nm"><text>{{S.name}}</text><text class="lvt">⭐ LV6</text></view>
            <view class="uid"><text>UID: {{S.uid}}</text><text class="copy-uid" @click.stop="copyUID">📋 复制</text></view>
            <text class="vb">👑 {{vipLabel}}</text>
          </view>
        </view>
        <!-- 统计 -->
        <view class="st">
          <view class="si" hover-class="si-hover" @click="openPage('following')"><text class="sn">128</text><text class="sl">{{T('following')}}</text></view>
          <view class="si" hover-class="si-hover" @click="openPage('followers')" style="margin-top:-7rpx"><text class="sn">3.2万</text><text class="sl">{{T('followers')}}</text></view>
          <view class="si" hover-class="si-hover" @click="openPage('likes')"><text class="sn">1,256</text><text class="sl">{{T('likes')}}</text></view>
          <view class="si" hover-class="si-hover" @click="openPage('coins')"><text class="sn">{{S.coins}}</text><text class="sl">{{T('coins')}}</text></view>
        </view>
      </view>

      <!-- 大会员 -->
      <view class="vc" hover-class="vc-hover" @click="openPage('vip-center')">
        <text class="vi">👑</text>
        <view class="vin"><text class="vt">{{T('vipc')}}</text><text class="vs">{{vipLabel}} · {{S.vipExp}} · Lv.{{S.vipLv}}</text></view>
        <text class="va">›</text>
      </view>

      <!-- 常用功能 -->
      <view class="sc"><view class="sh"><view class="sd"/><text>常用功能</text></view>
        <view class="g1">
          <view class="gi" v-for="item in quickActions" :key="item.key" hover-class="gi-hover" @click="openPage(item.key)">
            <view class="ii"><image :src="item.icon" class="ii-img" mode="aspectFit"/></view>
            <text class="il">{{item.label}}</text><text class="ib">{{item.badge}}</text>
          </view>
        </view>
      </view>

      <!-- 创作中心 -->
      <view class="sc"><view class="sh"><view class="sd"/><text>{{T('cc')}}</text></view>
        <view class="g1">
          <view class="gi" v-for="item in creatorActions" :key="item.key" hover-class="gi-hover" @click="openPage(item.key)">
            <view class="ii"><image :src="item.icon" class="ii-img" mode="aspectFit"/></view>
            <text class="il">{{item.label}}</text>
            <text class="it" :class="item.tagClass" v-if="item.tag">{{item.tag}}</text>
            <text class="ib" v-if="item.badge">{{item.badge}}</text>
          </view>
        </view>
      </view>

      <!-- 游戏中心 -->
      <view class="sc"><view class="sh"><view class="sd"/><text>{{T('gc')}}</text></view>
        <view class="g1">
          <view class="gi" v-for="item in gameActions" :key="item.key" hover-class="gi-hover" @click="openPage(item.key)">
            <view class="ii"><image :src="item.icon" class="ii-img" mode="aspectFit"/></view>
            <text class="il">{{item.label}}</text>
            <text class="it" :class="item.tagClass" v-if="item.tag">{{item.tag}}</text>
            <text class="ib" v-if="item.badge">{{item.badge}}</text>
          </view>
        </view>
      </view>

      <!-- 我的服务 -->
      <view class="sc"><view class="sh"><view class="sd"/><text>我的服务</text></view>
        <view class="g1">
          <view class="gi" v-for="item in serviceActions" :key="item.key" hover-class="gi-hover" @click="openPage(item.key)">
            <view class="ii"><image :src="item.icon" class="ii-img" mode="aspectFit"/></view>
            <text class="il">{{item.label}}</text><text class="ib" v-if="item.badge">{{item.badge}}</text>
          </view>
        </view>
        <view class="g1 g1-bordered">
          <view class="gi" v-for="item in serviceActions2" :key="item.key" hover-class="gi-hover" @click="openPage(item.key)">
            <view class="ii"><image :src="item.icon" class="ii-img" mode="aspectFit"/></view>
            <text class="il">{{item.label}}</text><text class="ib" v-if="item.badge">{{item.badge}}</text>
          </view>
        </view>
      </view>

      <!-- 更多服务 -->
      <view class="sc"><view class="sh"><view class="sd"/><text>更多服务</text></view>
        <view class="mr" v-for="item in moreActions" :key="item.key" hover-class="mr-hover" @click="openPage(item.key)">
          <view class="ri"><image :src="item.icon" class="ri-img" mode="aspectFit"/></view>
          <text class="rl">{{item.label}}</text><text class="re" v-if="item.sub">{{item.sub}}</text><text class="ra">›</text>
        </view>
      </view>

      <!-- 底部按钮 -->
      <view class="lw">
        <view class="lb2" hover-class="lb2-hover" @click="switchAccount"><text>{{T('switchAcc')}}</text></view>
        <view class="lb" hover-class="lb-hover" @click="logoutModalVisible=true"><text>{{T('logout')}}</text></view>
      </view>
      <view class="fs"/>
    </scroll-view>    <!-- ====== Spo 子页面浮层 ====== -->
    <view class="spo" :class="subPageVisible?'s':'ha'">
      <view class="sph">
        <view class="spb" hover-class="spb-hover" @click="closeSubPage"><text>〈</text></view>
        <text class="spt">{{subPageTitle}}</text>
        <view class="spa" v-if="currentSubPage==='profile'" @click="openPage('edit-profile')"><text class="spa-text">✎ {{T('edit')}}</text></view>
        <view class="spa" v-else-if="currentSubPage==='history'" @click="clearHistory"><text class="spa-text-danger">清空</text></view>
        <view class="spa" v-else-if="currentSubPage==='favorites'" @click="addFavFolder"><text class="spa-text-link">+ 新建</text></view>
        <view class="spa" v-else-if="currentSubPage==='watch-later'" @click="clearWatchLater"><text class="spa-text-danger">清空</text></view>
        <view class="spa" v-else-if="currentSubPage==='drafts'" @click="openPage('upload-video')"><text class="spa-text-link">+ 投稿</text></view>
        <view class="spa" v-else-if="currentSubPage==='mall-orders'" @click="openCart"><text>🛒</text><text v-if="S.cart.length" class="cart-count-badge">{{S.cart.length}}</text></view>
        <view class="spa" v-else/>
      </view>
      <scroll-view class="spbd" scroll-y :show-scrollbar="false" :key="'subpage-'+currentSubPage">

        <!-- ====== 子页面: profile (个人主页) ====== -->
        <view v-if="currentSubPage==='profile'">
          <view class="pf-banner">
            <view class="pf-bg" :style="{background:bgColors[bgIdx]}" @click="changeBg"/>
            <view class="pf-bg-btn" @click="changeBg"><text>🖼 换背景</text></view>
          </view>
          <view class="pf-info-section">
            <view class="pf-info-top">
              <view class="pfab">
                <image v-if="S.av==='默认头像'" src="../../mine_images/avatar.jpg" class="pfab-img" mode="aspectFill"/>
                <text v-else class="pfab-text">{{S.av}}</text>
              </view>
              <view style="flex:1"/>
            </view>
            <view class="pf-name-row">
              <text class="pf-name">{{S.name}}</text>
              <text class="sbd bl" style="font-size:20rpx">{{S.gender}}</text>
              <text class="sbd pk" style="font-size:20rpx">LV6</text>
            </view>
            <text class="pf-meta">UID: {{S.uid}} · {{vipLabel}} · {{S.school}} · {{S.loc}}</text>
            <text class="pf-bio">{{S.bio}}</text>
          </view>
          <view class="pf-stats-row">
            <view class="pf-stat" @click="openPage('following')"><text class="pf-sn">128</text><text class="pf-sl">{{T('following')}}</text></view>
            <view class="pf-stat" @click="openPage('followers')"><text class="pf-sn">3.2万</text><text class="pf-sl">{{T('followers')}}</text></view>
            <view class="pf-stat" @click="openPage('likes')"><text class="pf-sn">1,256</text><text class="pf-sl">{{T('likes')}}</text></view>
            <view class="pf-stat" @click="openPage('coins')"><text class="pf-sn">{{S.coins}}</text><text class="pf-sl">{{T('coins')}}</text></view>
          </view>
          <!-- Profile Tabs -->
          <view class="stabs" style="position:sticky;top:0;z-index:10">
            <text class="stab" :class="{a:profileTab==='home'}" @click="profileTab='home'">主页</text>
            <text class="stab" :class="{a:profileTab==='dynamic'}" @click="profileTab='dynamic'">动态</text>
            <text class="stab" :class="{a:profileTab==='videos'}" @click="profileTab='videos'">投稿 {{S.posts.length}}</text>
            <text class="stab" :class="{a:profileTab==='collections'}" @click="profileTab='collections'">收藏</text>
            <text class="stab" :class="{a:profileTab==='anime'}" @click="profileTab='anime'">追番</text>
          </view>
          <!-- Profile Tab Panels -->
          <view class="pf-tab-panel" :class="{a:profileTab==='home'}">
            <view class="ss" style="margin-top:16rpx"><view class="ssh"><text>近期投稿</text></view>
              <view class="svi" v-for="(v,i) in S.posts" :key="'ph-'+i">
                <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.icon}}</text><text class="svd" v-if="v.dur">{{v.dur}}</text></view>
                <view class="svf"><text class="svt2">{{v.title}}</text><text class="svm">{{v.meta}}</text></view>
              </view>
            </view>
          </view>
          <view class="pf-tab-panel" :class="{a:profileTab==='dynamic'}">
            <view class="ss" style="margin-top:16rpx">
              <view class="sli" v-for="(d,i) in S.dynamics" :key="'pd-'+i">
                <view class="sth" style="background:#fff0f4"><text>{{S.av}}</text></view>
                <view class="sif"><text class="sit">{{S.name}}</text><text class="sis">{{d.text}}</text></view>
                <text class="sbd" :class="d.tc">{{d.tag}}</text>
              </view>
            </view>
          </view>
          <view class="pf-tab-panel" :class="{a:profileTab==='videos'}">
            <view class="ss" style="margin-top:16rpx">
              <view class="svi" v-for="(v,i) in S.posts" :key="'pv-'+i">
                <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.icon}}</text><text class="svd" v-if="v.dur">{{v.dur}}</text></view>
                <view class="svf"><text class="svt2">{{v.title}}</text><text class="svm">{{v.meta}}</text></view>
              </view>
            </view>
          </view>
          <view class="pf-tab-panel" :class="{a:profileTab==='collections'}">
            <view class="ss" style="margin-top:16rpx">
              <view class="sli" v-for="(f,i) in S.favFolders" :key="'pc-'+i">
                <view class="sth" :class="{r8:true}" :style="{background:f.priv==='公开'?'#fff7e6':'#f4f6f8'}"><text>📁</text></view>
                <view class="sif"><text class="sit">{{f.name}}</text><text class="sis">{{f.items}}个内容·{{f.priv}}</text></view>
                <text class="sbd" :class="f.priv==='公开'?'gn':'gy'">{{f.priv}}</text>
              </view>
            </view>
          </view>
          <view class="pf-tab-panel" :class="{a:profileTab==='anime'}">
            <view class="ss" style="margin-top:16rpx">
              <view class="sli" v-for="(a,i) in S.animeFollow" :key="'pa-'+i">
                <view class="sth r8" style="background:#fee9e6"><text>📺</text></view>
                <view class="sif"><text class="sit">{{a.t}}</text><text class="sis">{{a.s}}</text></view>
                <text class="sbd" :class="a.tc">{{a.tg}}</text>
              </view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: edit-profile (编辑资料) ====== -->
        <view v-if="currentSubPage==='edit-profile'">
          <view class="ss" style="margin-top:20rpx"><view class="ssh"><text>基本信息</text></view>
            <view class="ep-avatar-row">
              <text class="ep-label">头像</text>
              <view class="ep-avatar" @click="openAvatarPicker">
                <image v-if="S.av==='默认头像'" src="../../mine_images/avatar.jpg" class="ep-avatar-img" mode="aspectFill"/>
                <text v-else class="ep-avatar-text">{{S.av}}</text>
              </view>
              <text class="ep-link" @click="openAvatarPicker">更换 ›</text>
            </view>
            <view class="srw"><text class="slb" style="width:120rpx">昵称</text><input class="mi" style="flex:1;margin:0" v-model="editName"/></view>
            <view class="srw"><text class="slb" style="width:120rpx">UID</text><text style="font-size:26rpx;color:var(--t2)">{{S.uid}}(不可修改)</text></view>
            <view class="srw"><text class="slb" style="width:120rpx">性别</text>
              <view class="ep-gender-row">
                <text :class="{active:editGender==='男'}" @click="editGender='男'">男</text>
                <text :class="{active:editGender==='女'}" @click="editGender='女'">女</text>
                <text :class="{active:editGender==='保密'}" @click="editGender='保密'">保密</text>
              </view>
            </view>
            <view class="srw"><text class="slb" style="width:120rpx">生日</text><input class="mi" style="flex:1;margin:0" v-model="editBirth" type="text" placeholder="YYYY-MM-DD"/></view>
          </view>
          <view class="ss"><view class="ssh"><text>详细信息</text></view>
            <view class="srw"><text class="slb" style="width:120rpx">学校</text><input class="mi" style="flex:1;margin:0" v-model="editSchool"/></view>
            <view class="srw"><text class="slb" style="width:120rpx">地区</text><input class="mi" style="flex:1;margin:0" v-model="editLoc"/></view>
            <view class="srw"><text class="slb" style="width:120rpx">简介</text><textarea class="mi mi-ta" style="flex:1;margin:0;height:120rpx" v-model="editBio"/></view>
          </view>
          <view style="padding:32rpx"><view class="sbtn pk save-btn" @click="saveProfile"><text>💾 保存资料</text></view></view>
        </view>

        <!-- ====== 子页面: settings (设置) ====== -->
        <view v-if="currentSubPage==='settings'">
          <!-- 播放设置 -->
          <view class="ss"><view class="ssh"><text>播放设置</text></view>
            <!-- autoPlay -->
            <view class="srw" @click="toggleSetting('autoPlay')"><text class="slb">自动播放下一集</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.autoPlay}"><view class="tg-dot"/></view></view>
            </view>
            <!-- autoFull -->
            <view class="srw" @click="toggleSetting('autoFull')"><text class="slb">自动全屏</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.autoFull}"><view class="tg-dot"/></view></view>
            </view>
            <!-- quality -->
            <view class="srw"><text class="slb">默认清晰度</text>
              <view class="seg-row">
                <text v-for="o in ['360P','480P','720P','1080P','4K']" :key="o" class="seg-opt" :class="{sel:S.settings.quality===o}" @click="updateSetting('quality',o)">{{o}}</text>
              </view>
            </view>
            <!-- hwDecode -->
            <view class="srw" @click="toggleSetting('hwDecode')"><text class="slb">硬解解码</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.hwDecode}"><view class="tg-dot"/></view></view>
            </view>
            <!-- danmakuType -->
            <view class="srw"><text class="slb">弹幕类型</text>
              <view class="seg-row">
                <text v-for="o in ['滚动','顶部','底部']" :key="o" class="seg-opt" :class="{sel:S.settings.danmakuType===o}" @click="updateSetting('danmakuType',o)">{{o}}</text>
              </view>
            </view>
            <!-- danmakuColor -->
            <view class="srw"><text class="slb">弹幕颜色</text>
              <view class="seg-row">
                <text v-for="o in ['白色','红色','蓝色','绿色','黄色']" :key="o" class="seg-opt" :class="{sel:S.settings.danmakuColor===o}" @click="updateSetting('danmakuColor',o)">{{o}}</text>
              </view>
            </view>
            <!-- danmakuSize -->
            <view class="srw"><text class="slb">弹幕字号</text>
              <view class="seg-row">
                <text v-for="o in ['小','中等','大']" :key="o" class="seg-opt" :class="{sel:S.settings.danmakuSize===o}" @click="updateSetting('danmakuSize',o)">{{o}}</text>
              </view>
            </view>
          </view>
          <!-- 通用设置 -->
          <view class="ss"><view class="ssh"><text>通用设置</text></view>
            <!-- darkMode -->
            <view class="srw" @click="toggleDark(!S.settings.darkMode)"><text class="slb">深色模式</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.darkMode}"><view class="tg-dot"/></view></view>
            </view>
            <!-- wifiOnly -->
            <view class="srw" @click="toggleSetting('wifiOnly')"><text class="slb">仅Wi-Fi播放</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.wifiOnly}"><view class="tg-dot"/></view></view>
            </view>
            <!-- lang -->
            <view class="srw"><text class="slb">语言</text>
              <view class="seg-row">
                <text v-for="o in langOptions" :key="o.value" class="seg-opt" :class="{sel:curLang===o.value}" @click="switchLang(o.value)">{{o.label}}</text>
              </view>
            </view>
            <!-- fontSize -->
            <view class="srw"><text class="slb">字体大小</text>
              <view class="seg-row">
                <text v-for="o in ['小','标准','大','特大']" :key="o" class="seg-opt" :class="{sel:S.settings.fontSize===o}" @click="updateSetting('fontSize',o)">{{o}}</text>
              </view>
            </view>
            <!-- qualityPref -->
            <view class="srw"><text class="slb">画质偏好</text>
              <view class="seg-row">
                <text v-for="o in ['智能','高清优先','流畅优先']" :key="o" class="seg-opt" :class="{sel:S.settings.qualityPref===o}" @click="updateSetting('qualityPref',o)">{{o}}</text>
              </view>
            </view>
            <!-- cacheQuality -->
            <view class="srw"><text class="slb">缓存清晰度</text>
              <view class="seg-row">
                <text v-for="o in ['360P','480P','720P','1080P']" :key="o" class="seg-opt" :class="{sel:S.settings.cacheQuality===o}" @click="updateSetting('cacheQuality',o)">{{o}}</text>
              </view>
            </view>
          </view>
          <!-- 隐私设置 -->
          <view class="ss"><view class="ssh"><text>隐私设置</text></view>
            <view class="srw"><text class="slb">私信权限</text>
              <view class="seg-row">
                <text v-for="o in ['所有人','关注的人','关闭']" :key="o" class="seg-opt" :class="{sel:S.settings.pmWho===o}" @click="updateSetting('pmWho',o)">{{o}}</text>
              </view>
            </view>
            <view class="srw" @click="toggleSetting('hideFav')"><text class="slb">隐藏收藏</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.hideFav}"><view class="tg-dot"/></view></view>
            </view>
            <view class="srw" @click="toggleSetting('hideAnime')"><text class="slb">不展示追番</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.hideAnime}"><view class="tg-dot"/></view></view>
            </view>
            <view class="srw" @click="toggleSetting('hideFollow')"><text class="slb">隐藏关注</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.hideFollow}"><view class="tg-dot"/></view></view>
            </view>
          </view>
          <!-- 消息设置 -->
          <view class="ss"><view class="ssh"><text>消息设置</text></view>
            <view class="srw" @click="toggleSetting('likeNotif')"><text class="slb">点赞通知</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.likeNotif}"><view class="tg-dot"/></view></view>
            </view>
            <view class="srw" @click="toggleSetting('commentNotif')"><text class="slb">评论通知</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.commentNotif}"><view class="tg-dot"/></view></view>
            </view>
            <view class="srw" @click="toggleSetting('sysNotif')"><text class="slb">系统通知</text>
              <view class="tg-wrap"><view class="tg-track" :class="{on:S.settings.sysNotif}"><view class="tg-dot"/></view></view>
            </view>
            <view class="srw" @click="openDND"><text class="slb">免打扰</text>
              <text class="sva">{{S.settings.dndStart}} - {{S.settings.dndEnd}}</text><text class="sir">›</text>
            </view>
          </view>
          <!-- 安全中心 -->
          <view class="ss"><view class="ssh"><text>安全中心</text></view>
            <view class="srw" @click="openChangePwd"><text class="slb">修改密码</text><text class="sva">修改 ›</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">绑定手机</text><text class="sva">{{S.phone}}</text></view>
            <view class="srw"><text class="slb">绑定邮箱</text><text class="sva">{{S.email}}</text></view>
          </view>
          <!-- 其他 -->
          <view class="ss"><view class="ssh"><text>其他</text></view>
            <view class="srw"><text class="slb">用户协议</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">隐私政策</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">开源许可</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">检查更新</text><text class="sva">v7.82.0</text></view>
          </view>
        </view>

        <!-- ====== 子页面: history (历史记录) ====== -->
        <view v-if="currentSubPage==='history'">
          <template v-if="S.historyVids.length">
            <view v-for="(group,day) in groupedHistory" :key="day">
              <view class="sddiv"><text>{{day}}</text></view>
              <view class="ss">
                <view class="svi" v-for="(v,i) in group" :key="'h-'+i" @click="removeHistoryItem(getHistoryIdx(day,i))">
                  <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.ic}}</text><text class="svd" v-if="v.dur">{{v.dur}}</text></view>
                  <view class="svf"><text class="svt2">{{v.t}}</text><text class="svm">{{v.m}}</text><view class="svp"><view class="svb" :style="{width:v.pr+'%'}"/></view></view>
                </view>
              </view>
            </view>
          </template>
          <view v-else class="se"><text class="sei">📭</text><text class="set">暂无历史</text><text class="seb" @click="closeSubPage">返回</text></view>
        </view>

        <!-- ====== 子页面: vip-center (大会员) ====== -->
        <view v-if="currentSubPage==='vip-center'">
          <view class="shero pb" style="margin-top:20rpx"><text class="sht">👑 {{S.vipType}}大会员</text><text class="shs">有效期至 {{S.vipExp}} · Lv.{{S.vipLv}}</text><text class="shi">💎</text></view>
          <view class="ss"><view class="ssh"><text>选择方案 · 当前: {{S.vipType}}</text></view>
            <view class="svc">
              <view class="svtier" :class="{cur:S.vipType==='月度'}" @click="selectVipTier('月度',25)">
                <text class="svti">⭐</text><text class="svtn">月度</text><text class="svtp">¥25</text><text class="svtb">1个月</text>
              </view>
              <view class="svtier" :class="{cur:S.vipType==='季度'}" @click="selectVipTier('季度',68)">
                <text class="svti">🌟</text><text class="svtn">季度</text><text class="svtp">¥68</text><text class="svtb">3个月·省¥7</text>
              </view>
              <view class="svtier" :class="{cur:S.vipType==='年度'}" @click="selectVipTier('年度',233)">
                <text class="svti">👑</text><text class="svtn">年度</text><text class="svtp">¥233</text><text class="svtb">12个月·省¥67</text>
              </view>
            </view>
            <view class="sqa"><text class="sbtn pk" @click="rechargeVip">💰 充值 (¥{{S.balance.toFixed(2)}})</text></view>
          </view>
          <view class="ss" style="margin-top:20rpx"><view class="ssh"><text>会员权益</text></view>
            <view class="sg4">
              <view class="sgi"><text class="sgic">🎬</text><text class="sgil">4K超清</text></view>
              <view class="sgi"><text class="sgic">⚡</text><text class="sgil">高速下载</text></view>
              <view class="sgi"><text class="sgic">📺</text><text class="sgil">抢先看</text></view>
              <view class="sgi"><text class="sgic">🎵</text><text class="sgil">会员音乐</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: favorites (收藏) ====== -->
        <view v-if="currentSubPage==='favorites'">
          <view class="stabs">
            <text class="stab" v-for="(f,i) in S.favFolders" :key="'ft-'+i" :class="{a:favActiveTab===i}" @click="favActiveTab=i">{{f.name}} <text style="font-size:20rpx;color:#999">{{f.items}}</text></text>
          </view>
          <view class="fav-btns">
            <text class="sbtn sm gy" @click="renameFav(favActiveTab)">✎ 重命名</text>
            <text class="sbtn sm gy" @click="toggleFavPriv(favActiveTab)">🔒 {{S.favFolders[favActiveTab]?S.favFolders[favActiveTab].priv:''}}</text>
            <text class="sbtn sm rd" @click="deleteFav(favActiveTab)">🗑 删除</text>
          </view>
          <view class="ss" v-if="S.favFolders[favActiveTab]">
            <view class="sli" v-for="(c,ci) in S.favFolders[favActiveTab].content" :key="'fc-'+ci">
              <view class="sth r8" :style="{background:c.bg}"><text>{{c.ic}}</text></view>
              <view class="sif"><text class="sit">{{c.t}}</text><text class="sis">{{c.m}}</text></view>
              <text class="fav-del" @click="removeFavItem(favActiveTab,ci)">✕</text>
            </view>
            <view v-if="!S.favFolders[favActiveTab].content.length" class="se"><text class="sei">📭</text><text class="set">空</text></view>
          </view>
        </view>

        <!-- ====== 子页面: watch-later (稍后再看) ====== -->
        <view v-if="currentSubPage==='watch-later'">
          <view class="ss" style="margin-top:20rpx">
            <view class="svi" v-for="(v,i) in S.watchLater" :key="'wl-'+i">
              <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.ic}}</text><text class="svd" v-if="v.dur">{{v.dur}}</text></view>
              <view class="svf"><text class="svt2">{{v.t}}</text><text class="svm">{{v.m}}</text></view>
              <text class="fav-del" @click="removeWatchLater(i)">✕</text>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: offline-cache (离线缓存) ====== -->
        <view v-if="currentSubPage==='offline-cache'">
          <view class="ss" style="margin-top:20rpx">
            <view style="padding:32rpx">
              <view style="display:flex;justify-content:space-between;margin-bottom:12rpx">
                <text style="font-size:28rpx;color:var(--t)">已用空间</text>
                <text style="font-size:28rpx;font-weight:600;color:var(--pink)">{{S.storageUsed}}GB / {{S.storageTotal}}GB</text>
              </view>
              <view class="spr"><view class="sprb" :style="{width:S.storageUsed/S.storageTotal*100+'%'}"/></view>
            </view>
          </view>
          <view class="ss">
            <view class="svi" v-for="(v,i) in S.cacheVids" :key="'ca-'+i">
              <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.ic}}</text></view>
              <view class="svf"><text class="svt2">{{v.t}}</text><text class="svm">{{v.m}}</text><view class="svp"><view class="svb" :style="{width:v.pr+'%'}"/></view></view>
              <text class="fav-del" @click="removeCache(i)">✕</text>
            </view>
          </view>
        </view>        <!-- ====== 子页面: game-center (游戏中心) ====== -->
        <view v-if="currentSubPage==='game-center'">
          <view class="shero prb" style="margin-top:20rpx"><text class="sht">🎮 {{T('gc')}}</text><text class="shs">存储: {{S.storageUsed.toFixed(1)}}GB/{{S.storageTotal}}GB</text><text class="shi">🎯</text></view>
          <view class="ss"><view class="ssh"><text>{{T('myg')}} </text><text style="font-size:22rpx;color:var(--t2)">{{S.games.length}}款</text></view>
            <view v-if="S.games.length">
              <view class="sli" v-for="(g,i) in S.games" :key="'mg-'+i">
                <view class="sth r8" :style="{background:getGameBg(g)}"><text>🎮</text></view>
                <view class="sif"><text class="sit">{{g}}</text><text class="sis">{{getGameInfo(g)}}·已安装</text></view>
                <text class="sbtn sm pk" @click="showToast('启动: '+g)">启动</text>
                <text class="sbtn sm rd" @click="uninstallGame(g)">卸载</text>
              </view>
            </view>
            <view v-else class="se"><text class="sei">🎮</text><text class="set">还没有游戏</text><text class="seb" @click="openPage('game-store')">去商店</text></view>
          </view>
          <view class="ss"><view class="ssh"><text>热门推荐</text><text class="ssh-link" @click="openPage('game-store')">全部 ›</text></view>
            <view class="sli" v-for="(g,i) in hotGames" :key="'hg-'+i">
              <view class="sth r8" :style="{background:g.bg}"><text>🎮</text></view>
              <view class="sif"><text class="sit">{{g.n}}</text><text class="sis">{{g.sz}}·{{g.pub}}</text></view>
              <text v-if="g.installed" class="sbtn sm gn" @click="showToast('已安装')">已安装</text>
              <text v-else class="sbtn sm pk" @click="downloadGame(g.n,g.sz)">下载</text>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: game-store (游戏商店) ====== -->
        <view v-if="currentSubPage==='game-store'">
          <view style="padding:20rpx 32rpx"><input class="mi" style="margin:0" v-model="gameSearch" placeholder="🔍 搜索..."/></view>
          <view class="ss">
            <view class="sli" v-for="(g,i) in filteredStoreGames" :key="'gs-'+i">
              <view class="sth r8" :style="{background:g.bg}"><text>🎮</text></view>
              <view class="sif"><text class="sit">{{g.n}}</text><text class="sis">{{g.cat}}·评分{{g.rate}}·{{g.pub}}·{{g.sz}}</text></view>
              <text v-if="g.installed" class="sbtn sm gn" @click="showToast('已安装')">已安装</text>
              <text v-else class="sbtn sm pk" @click="downloadGameFromStore(g.n,g.sz)">下载</text>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: my-games (我的游戏) ====== -->
        <view v-if="currentSubPage==='my-games'">
          <view class="ss" style="margin-top:20rpx">
            <view v-if="S.games.length">
              <view class="sli" v-for="(g,i) in S.games" :key="'mg2-'+i">
                <view class="sth r8" :style="{background:getGameBg(g)}"><text>🎮</text></view>
                <view class="sif"><text class="sit">{{g}}</text><text class="sis">{{getGameInfo(g)}}</text></view>
                <text class="sbtn sm pk" @click="showToast('启动: '+g)">启动</text>
                <text class="sbtn sm rd" @click="uninstallGame2(g)">卸载</text>
              </view>
            </view>
            <view v-else class="se"><text class="sei">🎮</text><text class="set">暂无游戏</text><text class="seb" @click="openPage('game-store')">去商店</text></view>
          </view>
        </view>

        <!-- ====== 子页面: my-reservations (我的预约) ====== -->
        <view v-if="currentSubPage==='my-reservations'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli" v-for="(r,i) in reservations" :key="'mr-'+i">
              <view class="sth r8" :style="{background:r.bg}"><text>{{r.ic}}</text></view>
              <view class="sif"><text class="sit">{{r.t}}</text><text class="sis">{{r.s}}</text></view>
              <text class="sbd pk">{{r.tg}}</text>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: creator-center (创作中心) ====== -->
        <view v-if="currentSubPage==='creator-center'">
          <view class="sic" style="background:linear-gradient(135deg,#fff0f4,var(--card-bg));margin-top:20rpx">
            <text class="sicic">🎬</text>
            <text class="sict">欢迎回来，{{S.name}}！</text>
          </view>
          <view class="ss" style="margin-top:20rpx"><view class="ssh"><text>数据概览</text></view>
            <view class="sg2stat">
              <view class="scd"><text class="scdn pk">56,230</text><text class="scdl">昨日播放</text></view>
              <view class="scd"><text class="scdn bl">230</text><text class="scdl">粉丝净增</text></view>
              <view class="scd"><text class="scdn">4.8%</text><text class="scdl">互动率</text></view>
            </view>
          </view>
          <view class="ss"><view class="ssh"><text>快捷操作</text></view>
            <view class="sg3">
              <view class="sgi" @click="openPage('upload-video')"><text class="sgic">📤</text><text class="sgil">{{T('upVideo')}}</text></view>
              <view class="sgi" @click="openPage('post-dynamic')"><text class="sgic">✍</text><text class="sgil">{{T('postDyn')}}</text></view>
              <view class="sgi" @click="showToast('开启直播')"><text class="sgic">🔴</text><text class="sgil">开启直播</text></view>
              <view class="sgi" @click="showToast('专栏投稿')"><text class="sgic">📝</text><text class="sgil">专栏投稿</text></view>
              <view class="sgi" @click="showToast('音频投稿')"><text class="sgic">🎤</text><text class="sgil">音频投稿</text></view>
              <view class="sgi" @click="showToast('互动管理')"><text class="sgic">💬</text><text class="sgil">互动管理</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: upload-video (上传视频) ====== -->
        <view v-if="currentSubPage==='upload-video'">
          <view class="ss" style="margin-top:20rpx">
            <view class="upload-area" @click="simulateFilePick">
              <text class="upload-icon">📤</text>
              <text class="upload-text">点击选择视频文件</text>
            </view>
          </view>
          <view class="ss"><view class="ssh"><text>视频信息</text></view>
            <view style="padding:0 32rpx 32rpx">
              <input class="mi" v-model="videoTitle" placeholder="标题"/>
              <textarea class="mi mi-ta" style="height:120rpx" v-model="videoDesc" placeholder="简介..."/>
              <view class="ep-gender-row">
                <text v-for="cat in videoCategories" :key="cat" :class="{active:videoCategory===cat}" @click="videoCategory=cat">{{cat}}</text>
              </view>
            </view>
          </view>
          <view style="padding:32rpx"><view class="sbtn pk save-btn" @click="publishVideo"><text>📤 发布</text></view></view>
        </view>

        <!-- ====== 子页面: post-dynamic (发布动态) ====== -->
        <view v-if="currentSubPage==='post-dynamic'">
          <view class="ss" style="margin-top:20rpx">
            <view style="padding:32rpx">
              <textarea class="mi mi-ta" style="height:200rpx;margin:0" v-model="dynamicContent" placeholder="分享想法..." @input="onDynamicInput"/>
              <text style="text-align:right;font-size:24rpx;color:var(--t2);display:block">{{dynamicCharCount}}/200</text>
            </view>
          </view>
          <view style="padding:32rpx"><view class="sbtn pk save-btn" @click="publishDynamic"><text>✍ 发布</text></view></view>
        </view>

        <!-- ====== 子页面: drafts (稿件管理) ====== -->
        <view v-if="currentSubPage==='drafts'">
          <view class="ss" style="margin-top:20rpx">
            <view class="svi" v-for="(v,i) in S.posts" :key="'dr-'+i">
              <view class="svt" :style="{background:v.bg}"><text class="svt-ic">{{v.icon}}</text><text class="svd" v-if="v.dur">{{v.dur}}</text></view>
              <view class="svf"><text class="svt2">{{v.title}}</text><text class="svm">已发布·{{v.time}}·{{v.meta}}</text></view>
            </view>
            <view class="svi">
              <view class="svt" style="background:#fff7e6"><text class="svt-ic">📝</text><text class="svd">12:45</text></view>
              <view class="svf"><text class="svt2">【草稿】夏日美食测评</text><text class="svm">草稿·6月15日</text></view>
            </view>
            <view class="svi">
              <view class="svt" style="background:#f0f5ff"><text class="svt-ic">⏳</text><text class="svd">16:20</text></view>
              <view class="svf"><text class="svt2">科技产品开箱</text><text class="svm">审核中·6月16日</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: wallet (钱包) ====== -->
        <view v-if="currentSubPage==='wallet'">
          <view class="swb"><text class="swbt">¥{{S.balance.toFixed(2)}}</text><text class="swbl">余额</text>
            <view class="swbr">
              <view class="swbi"><text class="swbn">{{S.coins}}</text><text class="swbs">硬币</text></view>
              <view class="swbi"><text class="swbn">2,350</text><text class="swbs">积分</text></view>
              <view class="swbi"><text class="swbn">¥58</text><text class="swbs">优惠券</text></view>
            </view>
          </view>
          <view class="sqa">
            <text class="sbtn pk" @click="rechargeWallet">💰 充值</text>
            <text class="sbtn ol" @click="showToast('提现')">💳 提现</text>
          </view>
          <view class="ss"><view class="ssh"><text>交易记录</text></view>
            <view class="sli"><view class="sth"><text>⬆</text></view><view class="sif"><text class="sit">充值</text><text class="sis">6月15日</text></view><text class="sib">+¥68</text></view>
            <view class="sli"><view class="sth"><text>🛍</text></view><view class="sif"><text class="sit">会员购</text><text class="sis">6月14日</text></view><text class="sib">-¥30</text></view>
            <view class="sli"><view class="sth"><text>⬆</text></view><view class="sif"><text class="sit">充值</text><text class="sis">6月10日</text></view><text class="sib">+¥128</text></view>
          </view>
        </view>

        <!-- ====== 子页面: mall-orders (会员购) ====== -->
        <view v-if="currentSubPage==='mall-orders'">
          <view class="ss">
            <view class="sli" v-for="(p,i) in mallOrders" :key="'mo-'+i">
              <view class="sth r8" :style="{background:p.bg}"><text>{{p.ic}}</text></view>
              <view class="sif"><text class="sit">{{p.t}}</text><text class="sis">{{p.s}}</text></view>
              <text class="sbd" :class="p.tc">{{p.tg}}</text>
              <text class="sbtn sm ol" @click="addToCart(p.t,p.price)">加购</text>
            </view>
          </view>
          <view class="ss"><view class="ssh"><text>逛商城</text></view>
            <view class="sg4">
              <view class="sgi" @click="showToast('手办')"><text class="sgic">👗</text><text class="sgil">手办</text></view>
              <view class="sgi" @click="showToast('画集')"><text class="sgic">📚</text><text class="sgil">画集</text></view>
              <view class="sgi" @click="showToast('数码')"><text class="sgic">🎧</text><text class="sgil">数码</text></view>
              <view class="sgi" @click="showToast('服饰')"><text class="sgic">👕</text><text class="sgil">服饰</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: comics (漫画) ====== -->
        <view v-if="currentSubPage==='comics'">
          <view class="shero pb" style="margin-top:20rpx"><text class="sht">📚 哔哩哔哩漫画</text><text class="shs">海量正版·随时阅读</text><text class="shi">📖</text></view>
          <view class="ss"><view class="ssh"><text>我的书架</text></view>
            <view v-if="S.comicShelf.length">
              <view class="sli" v-for="(c,i) in S.comicShelf" :key="'cm-'+i">
                <view class="sth r8" :style="{background:c.bg}"><text>📚</text></view>
                <view class="sif"><text class="sit">{{c.t}}</text><text class="sis">{{c.s}}</text></view>
                <text class="sbd" :class="c.tc">{{c.tg}}</text>
                <text class="sbtn sm rd" @click="removeComic(i)">移除</text>
              </view>
            </view>
            <view v-else class="se"><text class="set">书架空空</text></view>
          </view>
          <view class="ss"><view class="ssh"><text>热门推荐</text></view>
            <view class="sli" v-for="(c,i) in hotComics" :key="'hc-'+i">
              <view class="sth r8" :style="{background:c.bg}"><text>🔥</text></view>
              <view class="sif"><text class="sit">{{c.t}}</text><text class="sis">{{c.s}}</text></view>
              <text class="sbtn sm pk" @click="addComic(c.t,c.s)">追漫</text>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: my-courses (课程) ====== -->
        <view v-if="currentSubPage==='my-courses'">
          <view class="c2">
            <view class="cc"><view class="ssh"><text>{{T('courses')}} ({{S.courses.length}})</text></view>
              <view v-for="(c,i) in S.courses" :key="'co-'+i">
                <view class="sli">
                  <view class="sth r8" :style="{background:c.bg}"><text>{{c.ic}}</text></view>
                  <view class="sif"><text class="sit">{{c.t}}</text><text class="sis">{{c.s}}</text></view>
                  <text class="fav-del" @click="removeCourse(i)">✕</text>
                </view>
                <view class="spr" style="margin:0 32rpx 16rpx"><view class="sprb" :class="c.prc" :style="{width:c.pr+'%'}"/></view>
              </view>
            </view>
            <view class="cc"><view class="ssh"><text>推荐课程</text></view>
              <view class="sli" v-for="(c,i) in recCourses" :key="'rc-'+i">
                <view class="sth r8" :style="{background:c.bg}"><text>{{c.ic}}</text></view>
                <view class="sif"><text class="sit">{{c.t}}</text><text class="sis">{{c.s}}</text></view>
                <text class="sbtn sm ol" @click="addCourse(c.t,c.s,c.ic,c.bg)">+</text>
              </view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: downloads (下载管理) ====== -->
        <view v-if="currentSubPage==='downloads'">
          <view class="ss" style="margin-top:20rpx">
            <view v-if="Object.keys(S.downloads).length">
              <view class="svi" v-for="(d,k) in S.downloads" :key="'dl-'+k">
                <view class="svt" :style="{background:d.status==='done'?'#e8f8e8':'#e8f0fe'}"><text class="svt-ic">⬇</text></view>
                <view class="svf"><text class="svt2">{{d.name}}</text><text class="svm">{{d.size}}GB·{{d.status==='done'?'完成':'下载中'}}</text><view class="svp"><view class="svb" :style="{width:d.progress+'%'}"/></view></view>
              </view>
            </view>
            <view v-else class="se"><text class="sei">📭</text><text class="set">暂无下载</text></view>
          </view>
        </view>

        <!-- ====== 子页面: my-live (直播) ====== -->
        <view v-if="currentSubPage==='my-live'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli"><view class="sth r8" style="background:#fff1f0"><text>🔴</text></view><view class="sif"><text class="sit">老番茄</text><text class="sis">1.2万人观看</text></view><text class="sbd rd">直播中</text></view>
            <view class="sli"><view class="sth r8" style="background:#e6f7fc"><text>🟢</text></view><view class="sif"><text class="sit">罗翔讲刑法</text><text class="sis">今晚20:00</text></view><text class="sbd bl">已预约</text></view>
            <view class="sli"><view class="sth r8" style="background:#f4f6f8"><text>⚫</text></view><view class="sif"><text class="sit">何同学</text><text class="sis">上次6月10日</text></view><text class="sbd gy">离线</text></view>
          </view>
        </view>

        <!-- ====== 子页面: data-center (数据中心) ====== -->
        <view v-if="currentSubPage==='data-center'">
          <view class="ss" style="margin-top:20rpx"><view class="ssh"><text>近7天</text></view>
            <view class="sg2stat">
              <view class="scd"><text class="scdn pk">12.8万</text><text class="scdl">播放</text></view>
              <view class="scd"><text class="scdn bl">1,230</text><text class="scdl">涨粉</text></view>
              <view class="scd"><text class="scdn">128h</text><text class="scdl">时长</text></view>
              <view class="scd"><text class="scdn">4.8%</text><text class="scdl">互动</text></view>
              <view class="scd"><text class="scdn">890</text><text class="scdl">弹幕</text></view>
              <view class="scd"><text class="scdn">2,350</text><text class="scdl">评论</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: activities (有奖活动) ====== -->
        <view v-if="currentSubPage==='activities'">
          <view class="shero pb" style="margin-top:20rpx"><text class="sht">🎁 {{T('act')}}</text></view>
          <view class="ss">
            <view class="sli"><view class="sth r8" style="background:#fff7e6"><text>🏆</text></view><view class="sif"><text class="sit">2026夏日创作季</text><text class="sis">7月15日截止·¥100,000</text></view><text class="sbd pk">进行中</text></view>
            <view class="sli"><view class="sth r8" style="background:#e6f7fc"><text>🎬</text></view><view class="sif"><text class="sit">新UP主扶持</text><text class="sis">长期·流量+现金</text></view><text class="sbd bl">进行中</text></view>
            <view class="sli"><view class="sth r8" style="background:#f6f0ff"><text>🎵</text></view><view class="sif"><text class="sit">翻唱大赛</text><text class="sis">6月30日·¥50,000</text></view><text class="sbd or">将截止</text></view>
          </view>
        </view>

        <!-- ====== 子页面: free-data (免流量服务) ====== -->
        <view v-if="currentSubPage==='free-data'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli"><view class="sth r8" style="background:#e6f7fc"><text>📶</text></view><view class="sif"><text class="sit">中国联通</text><text class="sis">22卡/33卡</text></view><text class="sbd gn">已开通</text></view>
            <view class="sli"><view class="sth r8" style="background:#fff7e6"><text>📶</text></view><view class="sif"><text class="sit">中国电信</text><text class="sis">星卡</text></view><text class="sbd gy">未开通</text><text class="sbtn sm ol" @click="showToast('已申请')">开通</text></view>
            <view class="sli"><view class="sth r8" style="background:#e6f7fc"><text>📶</text></view><view class="sif"><text class="sit">中国移动</text><text class="sis">花卡</text></view><text class="sbd gy">未开通</text><text class="sbtn sm ol" @click="showToast('已申请')">开通</text></view>
          </view>
        </view>

        <!-- ====== 子页面: promotion (必火推广) ====== -->
        <view v-if="currentSubPage==='promotion'">
          <view class="shero pb" style="margin-top:20rpx"><text class="sht">🔥 {{T('promotion')}}</text></view>
          <view class="ss">
            <view class="svi">
              <view class="svt" style="background:#fee9e6"><text class="svt-ic">🎬</text><text class="svd">08:32</text></view>
              <view class="svf"><text class="svt2">周末Vlog</text><text class="svm">预算¥200·已消耗¥156·曝光12,800</text><view class="svp"><view class="svb" style="width:78%"/></view></view>
            </view>
          </view>
          <view class="sqa"><text class="sbtn pk" @click="showToast('新建推广')">+ 新建</text></view>
        </view>

        <!-- ====== 子页面: community (社区中心) ====== -->
        <view v-if="currentSubPage==='community'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli"><view class="sth"><text>⚖</text></view><view class="sif"><text class="sit">小黑屋</text><text class="sis">违规用户公示</text></view><text class="sir">›</text></view>
            <view class="sli"><view class="sth"><text>🛡</text></view><view class="sif"><text class="sit">风纪委员会</text><text class="sis">参与社区治理</text></view><text class="sir">›</text></view>
            <view class="sli"><view class="sth"><text>📋</text></view><view class="sif"><text class="sit">举报记录</text><text class="sis">3条·2条已处理</text></view><text class="sir">›</text></view>
            <view class="sli"><view class="sth"><text>📜</text></view><view class="sif"><text class="sit">社区规范</text><text class="sis">行为准则</text></view><text class="sir">›</text></view>
            <view class="sli"><view class="sth"><text>💡</text></view><view class="sif"><text class="sit">意见反馈</text><text class="sis">帮助改进</text></view><text class="sir">›</text></view>
          </view>
        </view>

        <!-- ====== 子页面: charity (公益) ====== -->
        <view v-if="currentSubPage==='charity'">
          <view class="sic" style="margin-top:20rpx;background:linear-gradient(135deg,#fff1f0,var(--card-bg))">
            <text class="sicic">❤</text><text class="sict">累计捐助¥120</text>
          </view>
          <view class="ss">
            <view class="sli"><view class="sth r8" style="background:#fff7e6"><text>🏫</text></view><view class="sif"><text class="sit">乡村教育</text><text class="sis">¥50·5月12日</text></view><text class="sbd gn">完成</text></view>
            <view class="sli"><view class="sth r8" style="background:#e6f7fc"><text>🐾</text></view><view class="sif"><text class="sit">流浪动物</text><text class="sis">¥30·4月8日</text></view><text class="sbd gn">完成</text></view>
            <view class="sli"><view class="sth r8" style="background:#e8f8e8"><text>🌳</text></view><view class="sif"><text class="sit">植树造林</text><text class="sis">¥40·3月15日</text></view><text class="sbd gn">完成</text></view>
          </view>
        </view>

        <!-- ====== 子页面: workshop (工房) ====== -->
        <view v-if="currentSubPage==='workshop'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sg3">
              <view class="sgi" @click="showToast('AI绘图')"><text class="sgic">🎨</text><text class="sgil">AI绘图</text></view>
              <view class="sgi" @click="showToast('视频剪辑')"><text class="sgic">✂</text><text class="sgil">视频剪辑</text></view>
              <view class="sgi" @click="showToast('图片处理')"><text class="sgic">🖼</text><text class="sgil">图片处理</text></view>
              <view class="sgi" @click="showToast('音频工具')"><text class="sgic">🎤</text><text class="sgil">音频工具</text></view>
              <view class="sgi" @click="showToast('字幕生成')"><text class="sgic">💬</text><text class="sgil">字幕生成</text></view>
              <view class="sgi" @click="showToast('封面制作')"><text class="sgic">📔</text><text class="sgil">封面制作</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: energy (能量加油站) ====== -->
        <view v-if="currentSubPage==='energy'">
          <view class="sic" style="margin-top:20rpx">
            <text class="sicic">⚡</text><text class="sict">今日能量: 85/100</text>
            <view class="spr" style="margin:24rpx 40rpx 0"><view class="sprb" style="width:85%"/></view>
          </view>
          <view class="ss"><view class="ssh"><text>每日任务</text></view>
            <view class="sli"><view class="sth"><text>✅</text></view><view class="sif"><text class="sit">登录B站</text></view><text class="sib">+10</text><text class="sbd gn">完成</text></view>
            <view class="sli"><view class="sth"><text>✅</text></view><view class="sif"><text class="sit">看3个视频</text></view><text class="sib">+20</text><text class="sbd gn">完成</text></view>
            <view class="sli"><view class="sth"><text>⬜</text></view><view class="sif"><text class="sit">投币1次</text></view><text class="sib">+15</text><text class="sbd gy">未完成</text></view>
            <view class="sli"><view class="sth"><text>⬜</text></view><view class="sif"><text class="sit">分享视频</text></view><text class="sib">+15</text><text class="sbd gy">未完成</text></view>
          </view>
        </view>

        <!-- ====== 子页面: teen-guardian (未成年人守护) ====== -->
        <view v-if="currentSubPage==='teen-guardian'">
          <view class="ss" style="margin-top:20rpx">
            <view class="srw"><text class="slb">启用守护</text><view class="tg-wrap" @click="showToast('设置')"><view class="tg-track" :class="{on:false}"><view class="tg-dot"/></view></view></view>
            <view class="srw"><text class="slb">每日时长</text><text class="sva" @click="showToast('设置每日时长')">未设置</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">内容过滤</text><text class="sva" @click="showToast('内容过滤')">标准</text><text class="sir">›</text></view>
            <view class="srw"><text class="slb">守护密码</text><text class="sir">›</text></view>
          </view>
        </view>

        <!-- ====== 子页面: audio-mode (听视频) ====== -->
        <view v-if="currentSubPage==='audio-mode'">
          <view class="sap" style="margin-top:20rpx"><text class="sapic">🎧</text><text class="sapt">赛博朋克城市漫步</text>
            <view class="sapc">
              <view class="sapb" @click="showToast('上一首')"><text>⏮</text></view>
              <view class="sapb play" @click="showToast('播放')"><text>▶</text></view>
              <view class="sapb" @click="showToast('下一首')"><text>⏭</text></view>
            </view>
          </view>
          <view class="ss">
            <view class="sli"><view class="sth r8" style="background:#fff0e6"><text>🎵</text></view><view class="sif"><text class="sit">崩铁PV</text><text class="sis">5:21</text></view></view>
            <view class="sli"><view class="sth r8" style="background:#f0e6ff"><text>📱</text></view><view class="sif"><text class="sit">iPhone评测</text><text class="sis">18:45</text></view></view>
          </view>
        </view>

        <!-- ====== 子页面: scan (扫一扫) ====== -->
        <view v-if="currentSubPage==='scan'">
          <view class="sscan"><view class="sscf"><text>📷</text></view><text class="ssct">将二维码放入框内自动扫描</text>
            <view style="margin-top:60rpx;display:flex;gap:60rpx;justify-content:center">
              <view style="text-align:center" @click="showToast('从相册选择')"><text style="font-size:64rpx">🖼</text><text style="font-size:24rpx;color:#999;display:block">相册</text></view>
              <view style="text-align:center" @click="showToast('手电筒')"><text style="font-size:64rpx">🔦</text><text style="font-size:24rpx;color:#999;display:block">手电筒</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: following (关注) / followers (粉丝) / likes (获赞) / coins (硬币) ====== -->
        <view v-if="currentSubPage==='following'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli" v-for="(f,i) in followingList" :key="'fg-'+i">
              <view class="sth" :style="{background:f.bg}"><text>{{f.ic}}</text></view>
              <view class="sif"><text class="sit">{{f.t}}</text><text class="sis">{{f.s}}</text></view>
              <text class="sbd pk">{{f.tg}}</text>
            </view>
          </view>
        </view>
        <view v-if="currentSubPage==='followers'">
          <view class="ss" style="margin-top:20rpx">
            <view class="sli" v-for="(f,i) in followersList" :key="'fr-'+i">
              <view class="sth" :style="{background:f.bg}"><text>{{f.ic}}</text></view>
              <view class="sif"><text class="sit">{{f.t}}</text><text class="sis">{{f.s}}</text></view>
              <text v-if="f.tg" class="sbd bl">{{f.tg}}</text>
            </view>
          </view>
        </view>
        <view v-if="currentSubPage==='likes'">
          <view class="sic" style="margin-top:20rpx"><text class="sicic">👍</text><text class="sict">累计获赞1,256</text></view>
          <view class="ss">
            <view class="sli"><view class="sth" style="background:#e6f7fc"><text>👤</text></view><view class="sif"><text class="sit">林**</text><text class="sis">赞了《周末Vlog》·6月16日</text></view></view>
            <view class="sli"><view class="sth" style="background:#fff7e6"><text>👤</text></view><view class="sif"><text class="sit">王**</text><text class="sis">赞了你的评论·6月16日</text></view></view>
          </view>
        </view>
        <view v-if="currentSubPage==='coins'">
          <view class="sic" style="margin-top:20rpx;background:linear-gradient(135deg,#fff7e6,var(--card-bg))"><text class="sicic">🪙</text><text class="sict">硬币: {{S.coins}}</text></view>
          <view class="ss">
            <view class="sli"><view class="sth"><text>🪙</text></view><view class="sif"><text class="sit">每日登录</text><text class="sis">6月16日</text></view><text class="sib">+1</text></view>
            <view class="sli"><view class="sth"><text>⬆</text></view><view class="sif"><text class="sit">投币·崩铁攻略</text><text class="sis">6月14日</text></view><text class="sib">-2</text></view>
            <view class="sli"><view class="sth"><text>🎁</text></view><view class="sif"><text class="sit">活动奖励</text><text class="sis">6月12日</text></view><text class="sib">+5</text></view>
          </view>
        </view>

        <!-- ====== 子页面: customer-service (客服) ====== -->
        <view v-if="currentSubPage==='customer-service'">
          <view class="ss" style="margin-top:20rpx">
            <view style="padding:40rpx 32rpx;text-align:center">
              <text style="font-size:88rpx">💬</text>
              <text style="font-size:32rpx;font-weight:600;color:var(--t);display:block;margin:16rpx 0">需要帮助？</text>
              <view style="margin-top:28rpx;display:flex;gap:20rpx;justify-content:center">
                <text class="sbtn pk" @click="showToast('客服已接入')">💬 在线咨询</text>
                <text class="sbtn ol" @click="leaveMessage">📝 留言</text>
              </view>
            </view>
          </view>
          <view class="ss"><view class="ssh"><text>常见问题</text></view>
            <view v-for="(faq,i) in faqList" :key="'faq-'+i" class="sfaq" :class="{op:faqOpen===i}" @click="faqOpen=faqOpen===i?-1:i">
              <view class="sfq"><text>{{faq.q}}</text><text>∨</text></view>
              <view class="sfa"><text>{{faq.a}}</text></view>
            </view>
          </view>
        </view>

        <!-- ====== 子页面: themes (个性装扮) ====== -->
        <view v-if="currentSubPage==='themes'">
          <view class="shero pb" style="margin-top:20rpx"><text class="sht">🎨 {{T('themes')}}</text><text class="shs">当前: {{getThemeName(S.theme)}}</text><text class="shi">✨</text></view>
          <view class="ss"><view class="ssh"><text>选择主题</text></view>
            <view class="sli" v-for="(t,i) in themeList" :key="'th-'+i" @click="applyTheme(t.id)" :style="S.theme===t.id?'background:var(--pink3)':''">
              <view class="sth" :style="{background:t.color}"><text>{{t.ic}}</text></view>
              <view class="sif"><text class="sit">{{t.name}}</text><text class="sis">{{t.color}}</text></view>
              <text v-if="S.theme===t.id" class="sbd pk">使用中</text>
            </view>
          </view>
        </view>

        <!-- ====== 开发中占位 ====== -->
        <view v-if="isDevPage" class="se"><text class="sei">🚧</text><text class="set">该页面功能开发中</text><text class="seb" @click="closeSubPage">返回</text></view>

      </scroll-view>
    </view>
  </view>
</template>
<script>
// ====== 用户资料库 ======
const USER_PROFILES = {
	'test': {
		name: 'test', uid: '111111', av: '🐱', bio: '这个人很懒，什么都没有写...🍃',
		gender: '保密', birth: '2000-01-01', school: '未知', loc: '未知',
		coins: 50, balance: 50.00, vipType: '月度', vipExp: '2026-12-31', vipLv: 1,
		phone: '138****0000', email: 'test@test.com', pwd: '123456',
		theme: 'pink'
	},
	'王润姿': {
		name: '王润姿', uid: '2312505003', av: '🐼', bio: '热爱生活，热爱分享 ✨',
		gender: '女', birth: '2004-03-15', school: '三亚学院', loc: '海南',
		coins: 120, balance: 256.80, vipType: '年度', vipExp: '2027-08-20', vipLv: 5,
		phone: '139****2164', email: '2312505003@qq.com', pwd: '123456',
		theme: 'pink'
	},
	'郭瑾涵': {
		name: '郭瑾涵', uid: '2312505005', av: '🦊', bio: '动画/游戏/音乐 🎮🎵',
		gender: '女', birth: '2005-11-08', school: '三亚学院', loc: '海南',
		coins: 95, balance: 180.20, vipType: '季度', vipExp: '2026-09-15', vipLv: 3,
		phone: '138****5678', email: '2312505005@qq.com', pwd: '123456',
		theme: 'blue'
	},
	'李睿': {
		name: '李睿', uid: '2312505039', av: '🐱', bio: '这个人很懒，什么都没有写...🍃',
		gender: '女', birth: '2005-07-01', school: '三亚学院', loc: '海南',
		coins: 86, balance: 128.50, vipType: '年度', vipExp: '2027-06-17', vipLv: 4,
		phone: '13930629164', email: '260551385@qq.com', pwd: '123456',
		theme: 'pink'
	},
	'李一佳': {
		name: '李一佳', uid: '2312505047', av: '🐰', bio: '摄影爱好者 📷 记录美好瞬间',
		gender: '女', birth: '2005-02-22', school: '三亚学院', loc: '海南',
		coins: 72, balance: 95.00, vipType: '月度', vipExp: '2026-07-31', vipLv: 2,
		phone: '139****9012', email: '2312505047@qq.com', pwd: '123456',
		theme: 'green'
	},
	'袁冶': {
		name: '袁冶', uid: '2312505002', av: '🐻', bio: '代码改变世界 💻',
		gender: '男', birth: '2005-05-10', school: '三亚学院', loc: '海南',
		coins: 105, balance: 320.00, vipType: '年度', vipExp: '2027-12-01', vipLv: 4,
		phone: '138****3456', email: '2312505002@qq.com', pwd: '123456',
		theme: 'pink'
	}
}

// 默认用户资料（未登录或未知用户）
const DEFAULT_PROFILE = {
	name: '未登录', uid: '--------', av: '👤', bio: '请先登录账号',
	gender: '保密', birth: '----', school: '--', loc: '--',
	coins: 0, balance: 0, vipType: '暂无', vipExp: '--', vipLv: 0,
	phone: '--------', email: '--------', pwd: '',
	theme: 'pink'
}

export default {
  name: 'BilibiliMine',
  data() {
    return {
      // ====== UI 状态 ======
      toastVisible: false,
      toastMessage: '',
      toastTimer: null,
      dlbarVisible: false,
      dlbarMessage: '',
      logoutModalVisible: false,
      generalModalVisible: false,
      generalModalType: '',
      generalModalContent: '',
      subPageVisible: false,
      subPageTitle: '',
      currentSubPage: null,

      // ====== 弹窗临时数据 ======
      dndStart: '',
      dndEnd: '',
      oldPwd: '',
      newPwd: '',
      newPwdConfirm: '',
      vipTierName: '',
      vipTierPrice: 0,
      rechargeAmount: '100',
      walletRechargeAmount: '100',
      newFavName: '',
      newFavPriv: '公开',
      renameFavName: '',
      renameFavIdx: 0,
      deleteConfirmMsg: '',
      deleteCallback: null,
      msgContent: '',
      msgContact: '',
      gameSearch: '',
      videoTitle: '',
      videoDesc: '',
      videoCategory: '游戏',
      dynamicContent: '',
      dynamicCharCount: 0,

      // ====== 子页面内部状态 ======
      profileTab: 'home',
      favActiveTab: 0,
      faqOpen: -1,
      editName: '',
      editGender: '',
      editBirth: '',
      editSchool: '',
      editLoc: '',
      editBio: '',

      // ====== 用户数据 (S 对象，登录后动态加载) ======
      S: {
        name: DEFAULT_PROFILE.name,
        uid: DEFAULT_PROFILE.uid,
        av: DEFAULT_PROFILE.av,
        bio: DEFAULT_PROFILE.bio,
        gender: DEFAULT_PROFILE.gender,
        birth: DEFAULT_PROFILE.birth,
        school: DEFAULT_PROFILE.school,
        loc: DEFAULT_PROFILE.loc,
        coins: DEFAULT_PROFILE.coins,
        balance: DEFAULT_PROFILE.balance,
        vipType: DEFAULT_PROFILE.vipType,
        vipExp: DEFAULT_PROFILE.vipExp,
        vipLv: DEFAULT_PROFILE.vipLv,
        phone: DEFAULT_PROFILE.phone,
        email: DEFAULT_PROFILE.email,
        pwd: DEFAULT_PROFILE.pwd,
        storageTotal: 1024,
        storageUsed: 3.2,
        downloads: {},
        games: ['崩坏：星穹铁道', '原神', '绝区零'],
        posts: [
          { type: 'video', title: '我的新视频_2026/6/17', time: '刚刚', meta: '审核中·游戏', icon: '🎬', bg: '#fee9e6', dur: '--:--' },
          { type: 'video', title: '周末Vlog·上海探店合集', time: '2天前', meta: '▶ 2.3w 💬 456', icon: '🎬', bg: '#fee9e6', dur: '08:32' },
          { type: 'video', title: '崩铁2.7全成就指南', time: '4天前', meta: '▶ 5.6w 💬 1.2k', icon: '🎮', bg: '#e6f7fc', dur: '25:18' }
        ],
        dynamics: [
          { type: 'post', text: '发布了新视频：《周末Vlog·上海探店合集》', time: '2天前', tag: '投稿', tc: 'pk' },
          { type: 'share', text: '转发了：这个游戏太好玩了！推荐给大家🎮', time: '3天前', tag: '转发', tc: 'bl' },
          { type: 'like', text: '赞了视频《崩铁2.7版本角色PV》', time: '4天前', tag: '点赞', tc: 'or' },
          { type: 'follow', text: '关注了UP主「老番茄」', time: '5天前', tag: '关注', tc: 'gn' }
        ],
        favFolders: [
          { name: '默认收藏夹', priv: '公开', items: 4, content: [
            { t: '线性代数本质', m: '3Blue1Brown', bg: '#e8f0fe', ic: '🎬' },
            { t: 'AI辅助写作', m: '所长林超', bg: '#fff0e6', ic: '🎵' },
            { t: 'React全栈开发', m: '技术胖', bg: '#f0e6ff', ic: '📱' },
            { t: 'C++内存管理', m: 'Cherno', bg: '#e6f7e8', ic: '🎮' }
          ]},
          { name: '学习资料', priv: '私密', items: 2, content: [
            { t: 'Python数据分析', m: '廖雪峰', bg: '#e6f7fc', ic: '📖' },
            { t: 'JavaScript高级', m: '阮一峰', bg: '#f0f5ff', ic: '💻' }
          ]},
          { name: '美食教程', priv: '公开', items: 2, content: [
            { t: '红烧肉教程', m: '王刚', bg: '#fff7e6', ic: '🍳' },
            { t: '手冲咖啡', m: '咖啡笔记', bg: '#fff0e6', ic: '☕' }
          ]}
        ],
        watchLater: [
          { t: '奥斯卡最佳动画短片合集', m: '动画学术趴·89万播放', bg: '#e8f0fe', ic: '🎬', dur: '15:22' },
          { t: '古琴《广陵散》完整版', m: '国风音乐·12万播放', bg: '#fff0e6', ic: '🎵', dur: '08:45' },
          { t: 'Linux内核源码分析', m: 'GeekCode·34万播放', bg: '#f0e6ff', ic: '📱', dur: '32:10' }
        ],
        historyVids: [
          { t: '赛博朋克城市漫步4K', m: '2小时前·2.3万播放', bg: '#e8f0fe', ic: '🎬', dur: '12:34', pr: 62, day: '今天' },
          { t: '崩铁2.7角色PV', m: '3小时前·128万播放', bg: '#fff0e6', ic: '🎵', dur: '05:21', pr: 100, day: '今天' },
          { t: 'iPhone18Pro深度评测', m: '5小时前·56万播放', bg: '#f0e6ff', ic: '📱', dur: '18:45', pr: 35, day: '今天' }
        ],
        cacheVids: [
          { t: '赛博朋克城市漫步4K', m: '3.2GB·已缓存', bg: '#e8f0fe', ic: '🎬', pr: 100 },
          { t: '崩坏星穹铁道PV', m: '1.8GB·已缓存', bg: '#fff0e6', ic: '🎵', pr: 100 },
          { t: 'iPhone18Pro评测', m: '2.4GB·缓存中...', bg: '#f0e6ff', ic: '📱', pr: 68 }
        ],
        courses: [
          { t: 'Python数据分析实战', s: '已学65%·廖雪峰', pr: 65, prc: 'bl', bg: '#e6f7fc', ic: '📖' },
          { t: 'UI设计入门到精通', s: '已学32%·邵飞', pr: 32, bg: '#fff0f4', ic: '🎨' },
          { t: 'JavaScript高级编程', s: '已学88%·阮一峰', pr: 88, prc: 'gn', bg: '#f0f5ff', ic: '💻' }
        ],
        cart: [],
        comicShelf: [
          { t: '镖人', s: '128话·许先哲', tg: '继续看', tc: 'pk', bg: '#f6f0ff' },
          { t: '伍六七之暗影刺客', s: '56话·何小疯', tg: '继续看', tc: 'pk', bg: '#fff7e6' },
          { t: '镇魂街', s: '312话·许辰', tg: '继续看', tc: 'bl', bg: '#e6f7fc' }
        ],
        animeFollow: [
          { t: '葬送的芙莉莲第二季', s: '更新至第18话', tg: '追番中', tc: 'pk' },
          { t: '鬼灭之刃无限城篇', s: '2026年7月开播', tg: '已预约', tc: 'bl' },
          { t: '间谍过家家第三季', s: '更新至第22话', tg: '追番中', tc: 'pk' }
        ],
        settings: {
          autoPlay: true, autoFull: false, quality: '1080P', hwDecode: true,
          danmakuType: '滚动', danmakuColor: '白色', danmakuSize: '中等',
          darkMode: false, wifiOnly: true, lang: 'zh', fontSize: '标准',
          qualityPref: '智能', cacheQuality: '1080P', pmWho: '所有人',
          hideFav: true, hideAnime: false, hideFollow: true,
          dndStart: '23:00', dndEnd: '07:00',
          likeNotif: true, commentNotif: true, sysNotif: false
        },
        theme: 'pink'
      },

      // ====== 静态数据 (头像/背景/图标/语言映射) ======
      avatars: ['默认头像','🐱','🐶','🐼','🐨','🦊','🐰','🐻','🐸','🐵','🦁','🐯','🐮','🐷','🐭','🐹','🐔','🐧','🐦','🐤','🦄'],
      bgColors: [
        'url(../../mine_images/banner_bg.jpg) center center/cover no-repeat',
        'linear-gradient(160deg,#a8d8ea,#c4e0e8,#d4c8d8,#e8c4d0)',
        'linear-gradient(160deg,#c8d8c4,#d4e0c8,#c8d0e0,#c8c4d4)',
        'linear-gradient(160deg,#e8d8c4,#e0d0c0,#c8c8d4,#a0b8d0)',
        'linear-gradient(160deg,#484860,#585878,#686890,#7878a8)'
      ],
      bgIdx: 0,
      langMap: {
        zh: {
          settings:'设置',profile:'个人主页',edit:'编辑',history:'历史记录',favorites:'我的收藏',wl:'稍后再看',
          cache:'离线缓存',cc:'创作中心',gc:'游戏中心',myg:'我的游戏',wallet:'我的钱包',mall:'会员购',
          comics:'漫画',courses:'我的课程',cs:'联系客服',scan:'扫一扫',logout:'退出登录',switchAcc:'切换账号',
          home:'首页',search:'搜索',trending:'热门',msgs:'消息',mine:'我的',
          following:'关注列表',followers:'粉丝列表',likes:'获赞列表',coins:'硬币记录',
          themes:'个性装扮',promotion:'必火推广',community:'社区中心',charity:'公益',workshop:'工房',
          energy:'能量加油站',audio:'听视频',teen:'未成年人守护',
          drafts:'稿件管理',dlm:'下载管理',datac:'数据中心',act:'有奖活动',reserve:'我的预约',
          freeData:'免流量服务',live:'我的直播',vipc:'大会员中心',gameStore:'游戏商店',
          upVideo:'上传视频',postDyn:'发布动态',editProf:'编辑资料'
        },
        en: {
          settings:'Settings',profile:'Profile',edit:'Edit',history:'History',favorites:'Favorites',wl:'Watch Later',
          cache:'Offline',cc:'Creator',gc:'Games',myg:'My Games',wallet:'Wallet',mall:'Shop',
          comics:'Comics',courses:'Courses',cs:'Support',scan:'Scan',logout:'Logout',switchAcc:'Switch',
          home:'Home',search:'Search',trending:'Trending',msgs:'Messages',mine:'Me',
          following:'Following',followers:'Followers',likes:'Likes',coins:'Coins',
          themes:'Themes',promotion:'Promote',community:'Community',charity:'Charity',workshop:'Studio',
          energy:'Energy',audio:'Audio',teen:'Guardian',
          drafts:'Content',dlm:'Downloads',datac:'Analytics',act:'Events',reserve:'Pre-order',
          freeData:'Free Data',live:'Live',vipc:'VIP Center',gameStore:'Game Store',
          upVideo:'Upload',postDyn:'Post',editProf:'Edit Profile'
        },
        ja: {
          settings:'設定',profile:'プロフ',edit:'編集',history:'履歴',favorites:'お気に入り',wl:'後で見る',
          cache:'オフライン',cc:'創作',gc:'ゲーム',myg:'マイゲーム',wallet:'財布',mall:'ショップ',
          comics:'漫画',courses:'講座',cs:'サポート',scan:'スキャン',logout:'ログアウト',switchAcc:'切替',
          home:'ホーム',search:'検索',trending:'話題',msgs:'メッセージ',mine:'マイ',
          following:'フォロー',followers:'フォロワー',likes:'いいね',coins:'コイン',
          themes:'テーマ',promotion:'プロモ',community:'コミュ',charity:'慈善',workshop:'工房',
          energy:'エネ',audio:'オーディオ',teen:'保護',
          drafts:'投稿管理',dlm:'ダウンロード',datac:'データ',act:'イベント',reserve:'予約',
          freeData:'無料通信',live:'ライブ',vipc:'VIP',gameStore:'ゲーム店',
          upVideo:'アップロード',postDyn:'投稿',editProf:'プロフ編集'
        },
        ko: {
          settings:'설정',profile:'프로필',edit:'편집',history:'기록',favorites:'즐겨찾기',wl:'나중에보기',
          cache:'오프라인',cc:'창작',gc:'게임',myg:'내게임',wallet:'지갑',mall:'쇼핑',
          comics:'만화',courses:'강의',cs:'고객센터',scan:'스캔',logout:'로그아웃',switchAcc:'전환',
          home:'홈',search:'검색',trending:'트렌딩',msgs:'메시지',mine:'마이',
          following:'팔로잉',followers:'팔로워',likes:'좋아요',coins:'코인',
          themes:'테마',promotion:'프로모션',community:'커뮤니티',charity:'자선',workshop:'공방',
          energy:'에너지',audio:'오디오',teen:'보호',
          drafts:'콘텐츠',dlm:'다운로드',datac:'데이터',act:'이벤트',reserve:'예약',
          freeData:'무료데이터',live:'라이브',vipc:'VIP',gameStore:'게임샵',
          upVideo:'업로드',postDyn:'글쓰기',editProf:'프로필편집'
        }
      },
      langOptions: [
        { value: 'zh', label: '中文' },
        { value: 'en', label: 'English' },
        { value: 'ja', label: '日本語' },
        { value: 'ko', label: '한국어' }
      ],
      quickActions: [
        { key: 'offline-cache', label: '离线缓存', badge: '3.2GB', icon: '../../mine_images/offline.png' },
        { key: 'history', label: '历史记录', badge: '今日12条', icon: '../../mine_images/history.png' },
        { key: 'favorites', label: '我的收藏', badge: '48', icon: '../../mine_images/favorites.png' },
        { key: 'watch-later', label: '稍后再看', badge: '3', icon: '../../mine_images/watch_later.png' }
      ],
      creatorActions: [
        { key: 'creator-center', label: '创作中心', tag: '有更新', tagClass: 'ht', icon: '../../mine_images/creator.png' },
        { key: 'drafts', label: '稿件管理', badge: '4', icon: '../../mine_images/drafts.png' },
        { key: 'data-center', label: '数据中心', icon: '../../mine_images/data.png' },
        { key: 'activities', label: '有奖活动', tag: 'NEW', tagClass: 'nw', icon: '../../mine_images/activities.png' }
      ],
      gameActions: [
        { key: 'game-center', label: '游戏中心', tag: '🔥', tagClass: 'ht', icon: '../../mine_images/game.png' },
        { key: 'my-games', label: '我的游戏', badge: '3', icon: '../../mine_images/my_games.png' },
        { key: 'my-reservations', label: '我的预约', badge: '5', icon: '../../mine_images/reservations.png' },
        { key: 'downloads', label: '下载管理', badge: '0', icon: '../../mine_images/downloads.png' }
      ],
      serviceActions: [
        { key: 'my-courses', label: '我的课程', badge: '3门', icon: '../../mine_images/courses.png' },
        { key: 'free-data', label: '免流量服务', icon: '../../mine_images/free_data.png' },
        { key: 'themes', label: '个性装扮', icon: '../../mine_images/themes.png' },
        { key: 'wallet', label: '我的钱包', badge: '¥128', icon: '../../mine_images/wallet.png' }
      ],
      serviceActions2: [
        { key: 'mall-orders', label: '会员购', icon: '../../mine_images/mall.png' },
        { key: 'my-live', label: '我的直播', icon: '../../mine_images/live.png' },
        { key: 'comics', label: '漫画', icon: '../../mine_images/comics.png' },
        { key: 'promotion', label: '必火推广', icon: '../../mine_images/promotion.png' }
      ],
      moreActions: [
        { key: 'customer-service', label: '联系客服', sub: '在线', icon: '../../mine_images/service.png' },
        { key: 'community', label: '社区中心', icon: '../../mine_images/community.png' },
        { key: 'charity', label: '公益', icon: '../../mine_images/charity.png' },
        { key: 'workshop', label: '工房', icon: '../../mine_images/workshop.png' },
        { key: 'energy', label: '能量加油站', icon: '../../mine_images/energy.png' },
        { key: 'audio-mode', label: '听视频', icon: '../../mine_images/audio.png' },
        { key: 'teen-guardian', label: '未成年人守护', icon: '../../mine_images/teen.png' }
      ],

      // ====== 内联数据 (原 render 中的硬编码数据) ======
      hotGames: [],
      allStoreGames: [],
      reservations: [],
      followingList: [],
      followersList: [],
      mallOrders: [],
      hotComics: [],
      recCourses: [],
      faqList: [],
      themeList: [],
      videoCategories: ['游戏','知识','生活','美食','音乐','科技']
    };
  },
  computed: {
    curLang() { return this.S.settings.lang || 'zh'; },
    vipLabel() {
      const map = { '年度': '年度大会员', '季度': '季度大会员', '月度': '月度大会员' };
      return map[this.S.vipType] || '大会员';
    },
    cartTotal() { return this.S.cart.reduce((a, b) => a + (b.price || 0), 0).toFixed(2); },
    // 子页面标题映射
    subPageTitle() {
      if (!this.currentSubPage) return '';
      return this.T(this.currentSubPage) || this.currentSubPage;
    },
    // 是否开发中页面
    isDevPage() {
      if (!this.currentSubPage || !this.subPageVisible) return false;
      const known = ['profile','edit-profile','settings','history','vip-center','favorites','watch-later',
        'offline-cache','game-center','game-store','my-games','my-reservations','creator-center',
        'upload-video','post-dynamic','drafts','wallet','mall-orders','comics','my-courses','downloads',
        'my-live','data-center','activities','free-data','promotion','community','charity','workshop',
        'energy','teen-guardian','audio-mode','scan','following','followers','likes','coins',
        'customer-service','themes'];
      return !known.includes(this.currentSubPage);
    },
    // 历史记录按天分组
    groupedHistory() {
      const days = {};
      this.S.historyVids.forEach(v => {
        const d = v.day || '未知';
        if (!days[d]) days[d] = [];
        days[d].push(v);
      });
      return days;
    },
    // 过滤游戏商店
    filteredStoreGames() {
      if (!this.gameSearch) return this.allStoreGames;
      const kw = this.gameSearch.toLowerCase();
      return this.allStoreGames.filter(g => g.n.toLowerCase().includes(kw));
    }
  },
  watch: {
    'S.settings.darkMode': {
      handler: function(val) {
        if (val) {
          uni.setNavigationBarColor({ frontColor: '#ffffff', backgroundColor: '#1a1a2e' });
        } else {
          uni.setNavigationBarColor({ frontColor: '#000000', backgroundColor: '#F8F8F8' });
        }
      },
      immediate: false
    }
  },
  mounted() {
    this.loadUserProfile();   // ⭐ 先根据登录账号加载用户资料
    this.loadState();         // 再覆盖本地保存的状态（如果有）
    this.initStaticData();
    this.resetEditData();
    if (this.S.settings.darkMode) {
      uni.setNavigationBarColor({ frontColor: '#ffffff', backgroundColor: '#1a1a2e' });
    }
  },
  methods: {
    // ========== 用户资料加载 ==========
    loadUserProfile() {
      try {
        var username = uni.getStorageSync('bili_username');
        if (username && USER_PROFILES[username]) {
          var profile = USER_PROFILES[username];
          // 只覆盖个人资料字段，保留 settings/theme/games 等本地数据
          this.S.name = profile.name;
          this.S.uid = profile.uid;
          this.S.av = profile.av;
          this.S.bio = profile.bio;
          this.S.gender = profile.gender;
          this.S.birth = profile.birth;
          this.S.school = profile.school;
          this.S.loc = profile.loc;
          this.S.coins = profile.coins;
          this.S.balance = profile.balance;
          this.S.vipType = profile.vipType;
          this.S.vipExp = profile.vipExp;
          this.S.vipLv = profile.vipLv;
          this.S.phone = profile.phone;
          this.S.email = profile.email;
          this.S.pwd = profile.pwd;
          this.S.theme = profile.theme || 'pink';
        }
      } catch (e) {}
    },

    // ========== 初始化 ==========
    initStaticData() {
      // 热门游戏
      this.hotGames = [
        { n: '崩坏：星穹铁道', sz: '28.5GB', bg: '#eae6ff', pub: '米哈游·RPG', installed: false },
        { n: '原神', sz: '42.1GB', bg: '#fff0f4', pub: '米哈游·开放世界', installed: false },
        { n: '绝区零', sz: '32.7GB', bg: '#e6f7fc', pub: '米哈游·动作RPG', installed: false },
        { n: '黑神话：悟空', sz: '78.3GB', bg: '#fff7e6', pub: '游戏科学·动作', installed: false }
      ];
      // 更新安装状态
      this.hotGames.forEach(g => { g.installed = this.S.games.indexOf(g.n) >= 0; });

      // 游戏商店
      this.allStoreGames = [
        { n: '崩坏：星穹铁道', cat: 'RPG', tag: '二次元', sz: '28.5GB', rate: '4.8', pub: '米哈游', bg: '#eae6ff', installed: false },
        { n: '原神', cat: '开放世界', tag: '开放世界', sz: '42.1GB', rate: '4.7', pub: '米哈游', bg: '#fff0f4', installed: false },
        { n: '绝区零', cat: '动作', tag: '二次元', sz: '32.7GB', rate: '4.6', pub: '米哈游', bg: '#e6f7fc', installed: false },
        { n: '黑神话：悟空', cat: '动作', tag: '国风', sz: '78.3GB', rate: '4.9', pub: '游戏科学', bg: '#fff7e6', installed: false },
        { n: '明日方舟', cat: '卡牌', tag: '二次元', sz: '8.2GB', rate: '4.5', pub: '鹰角网络', bg: '#f0f5ff', installed: false },
        { n: '王者荣耀', cat: '动作', tag: '联机', sz: '15.6GB', rate: '4.3', pub: '腾讯', bg: '#fff7e6', installed: false },
        { n: '和平精英', cat: '射击', tag: '联机', sz: '18.9GB', rate: '4.2', pub: '腾讯', bg: '#e6f7fc', installed: false },
        { n: 'Minecraft', cat: '沙盒', tag: '像素', sz: '2.1GB', rate: '4.8', pub: 'Mojang', bg: '#e8f8e8', installed: false }
      ];
      this.allStoreGames.forEach(g => { g.installed = this.S.games.indexOf(g.n) >= 0; });

      // 预约
      this.reservations = [
        { ic: '🌟', t: '无限暖暖', s: '2026年7月·89万人预约', tg: '已预约', bg: '#eae6ff' },
        { ic: '🌙', t: '明日方舟：终末地', s: '2026年12月·67万人预约', tg: '已预约', bg: '#fff0f4' },
        { ic: '🎯', t: 'Project: Mugen', s: '2026年Q4·34万人预约', tg: '已预约', bg: '#e6f7fc' },
        { ic: '⚡', t: '王者荣耀：世界', s: '2027年·128万人预约', tg: '已预约', bg: '#f6f0ff' }
      ];

      // 关注
      this.followingList = [
        { ic: '🐱', t: '老番茄', s: '2345万粉丝', tg: '已关注', bg: '#fff0f4' },
        { ic: '🦊', t: '罗翔讲刑法', s: '1890万粉丝', tg: '已关注', bg: '#e6f7fc' },
        { ic: '🐵', t: '何同学', s: '1234万粉丝', tg: '已关注', bg: '#fff7e6' },
        { ic: '🐧', t: '木鱼水心', s: '987万粉丝', tg: '已关注', bg: '#e9f0ff' }
      ];

      // 粉丝
      this.followersList = [
        { ic: '🌟', t: '星光点点', s: '256粉丝', tg: '互关', bg: '#fff0f4' },
        { ic: '🌙', t: '月下独酌', s: '1.2万粉丝', bg: '#e6f7fc' },
        { ic: '🔥', t: '烈焰风暴', s: '890粉丝', tg: '互关', bg: '#fff7e6' }
      ];

      // 会员购
      this.mallOrders = [
        { t: 'GSC初音未来韶华ver.', s: '¥899·2026年9月', tg: '待发货', tc: 'or', bg: '#fff7e6', ic: '📦', price: 899 },
        { t: '崩铁·银狼Q版手办', s: '¥299·运输中', tg: '运输中', tc: 'bl', bg: '#e6f7fc', ic: '📦', price: 299 },
        { t: '原神·刻晴主题T恤', s: '¥128·待付款', tg: '待付款', tc: 'or', bg: '#fff7e6', ic: '👕', price: 128 },
        { t: '绝区零·比利钥匙扣', s: '¥39·已完成', tg: '已完成', tc: 'gn', bg: '#e8f8e8', ic: '🔑', price: 39 }
      ];

      // 热门漫画
      this.hotComics = [
        { t: '鬼灭之刃', s: '评分4.9·热血', bg: '#fff7e6' },
        { t: '间谍过家家', s: '评分4.8·喜剧', bg: '#e6f7fc' },
        { t: '葬送的芙莉莲', s: '评分4.9·奇幻', bg: '#f6f0ff' }
      ];

      // 推荐课程
      this.recCourses = [
        { t: '机器学习入门', s: '吴恩达·38课时', ic: '📖', bg: '#e6f7fc' },
        { t: 'Go语言实战', s: '李文周·42课时', ic: '💻', bg: '#f0f5ff' }
      ];

      // FAQ
      this.faqList = [
        { q: '如何成为大会员？', a: '月度¥25/季度¥68/年度¥233。' },
        { q: '视频审核多久？', a: '15分钟-2小时。' }
      ];

      // 主题
      this.themeList = [
        { id: 'pink', name: '哔哩粉', color: '#fb7299', ic: '💗' },
        { id: 'yellow', name: '柠檬黄', color: '#f5c842', ic: '💛' },
        { id: 'green', name: '薄荷绿', color: '#4ecdc4', ic: '💚' },
        { id: 'blue', name: '天空蓝', color: '#5b9bd5', ic: '💙' },
        { id: 'red', name: '中国红', color: '#e8453c', ic: '❤' }
      ];
    },

    // ========== 翻译 & 存储 ==========
    T(key) {
      const langData = this.langMap[this.curLang] || this.langMap.zh;
      return langData[key] || this.langMap.zh[key] || key;
    },
    loadState() {
      try {
        var saved = uni.getStorageSync('bili_mine_state');
        if (saved) {
          var parsed = JSON.parse(saved);
          // 只有 _savedFor 与当前登录用户完全匹配时才恢复已保存状态
          var currentUser = uni.getStorageSync('bili_username') || '';
          if (parsed._savedFor && parsed._savedFor === currentUser) {
            Object.assign(this.S, parsed);
          }
        }
      } catch (e) {}
      this.resetEditData();
    },
    saveState() {
      try {
        this.S._savedFor = uni.getStorageSync('bili_username') || '';
        uni.setStorageSync('bili_mine_state', JSON.stringify(this.S));
      } catch (e) {}
    },
    resetEditData() {
      this.editName = this.S.name;
      this.editGender = this.S.gender;
      this.editBirth = this.S.birth;
      this.editSchool = this.S.school;
      this.editLoc = this.S.loc;
      this.editBio = this.S.bio;
    },

    // ========== Toast / 弹窗 ==========
    showToast(msg) {
      clearTimeout(this.toastTimer);
      this.toastMessage = msg;
      this.toastVisible = true;
      this.toastTimer = setTimeout(() => { this.toastVisible = false; }, 2500);
    },
    showDlbar(msg) {
      this.dlbarMessage = msg;
      this.dlbarVisible = true;
      setTimeout(() => { this.dlbarVisible = false; }, 2000);
    },
    openGeneralModal(type, content) {
      this.generalModalType = type || '';
      this.generalModalContent = content || '';
      this.generalModalVisible = true;
    },
    closeGeneralModal() {
      this.generalModalVisible = false;
      this.generalModalType = '';
      this.generalModalContent = '';
    },

    // ========== 退出 / 切换 ==========
    handleLogout() {
      this.logoutModalVisible = false;
      // 清除登录状态
      uni.removeStorageSync('bili_is_logged_in');
      uni.removeStorageSync('bili_username');
      this.showToast('👋已退出');
      var self = this;
      setTimeout(function() { uni.reLaunch({ url: '/pages/login/login' }); }, 800);
    },
    switchAccount() { uni.reLaunch({ url: '/pages/login/login' }); },
    goPage(name) {
      if (name === 'home') { uni.reLaunch({ url: '/pages/index/index' }); }
      else if (name === 'search') { uni.navigateTo({ url: '/pages/search/bilibili-search' }); }
      else if (name === 'trending') { uni.navigateTo({ url: '/pages/trending/bilibili-trending' }); }
    },
    showMessage() { this.showToast('💬 消息中心·3条未读'); },
    copyUID() {
      var self = this;
      uni.setClipboardData({
        data: String(this.S.uid),
        success: function() { self.showToast('✅ UID已复制'); },
        fail: function() { self.showToast('复制失败'); }
      });
    },

    // ========== 页面滚动 ==========
    handleScroll(e) {
      // uniapp scroll-view 的 scroll 事件
      var scrollTop = (e.detail && e.detail.scrollTop) || 0;
      // ⚠️ uniapp 中不能用 document.getElementById, 改为 ref + style 绑定
      // 滚动阴影效果通过 class 绑定实现
      this.scrollTop = scrollTop;
    },

    // ========== 背景切换 ==========
    changeBg() {
      this.bgIdx = (this.bgIdx + 1) % this.bgColors.length;
      this.showToast('✅背景已更换');
    },

    // ========== 子页面导航 ==========
    openPage(id) {
      var pageId = String(id || '');
      // 初始化编辑数据（如果是编辑资料页）
      if (pageId === 'edit-profile') {
        this.resetEditData();
      }
      // 设置当前子页面
      this.currentSubPage = pageId;
      // 确保浮层可见
      this.subPageVisible = true;
      // 重置子页面内部状态
      this.profileTab = 'home';
      this.favActiveTab = 0;
      this.faqOpen = -1;
    },
    closeSubPage() {
      this.subPageVisible = false;
      this.currentSubPage = null;
    },

    // ========== 头像选择 ==========
    openAvatarPicker() {
      this.openGeneralModal('avatar-picker');
    },
    selectAvatar(a) {
      this.S.av = a;
      this.closeGeneralModal();
      this.showToast('✅头像已选择');
      this.saveState();
    },

    // ========== 编辑资料 ==========
    saveProfile() {
      this.S.name = this.editName || this.S.name;
      this.S.gender = this.editGender;
      this.S.birth = this.editBirth;
      this.S.school = this.editSchool;
      this.S.loc = this.editLoc;
      this.S.bio = this.editBio;
      this.saveState();
      this.closeSubPage();
      this.openPage('profile');
      this.showToast('✅资料已保存');
    },

    // ========== 设置相关 ==========
    toggleSetting(key) {
      this.S.settings[key] = !this.S.settings[key];
      this.saveState();
      this.showToast(key + ': ' + (this.S.settings[key] ? '已开启' : '已关闭'));
    },
    updateSetting(key, val) {
      this.S.settings[key] = val;
      this.saveState();
    },
    toggleDark(on) {
      this.S.settings.darkMode = on;
      if (on) {
        uni.setNavigationBarColor({ frontColor: '#ffffff', backgroundColor: '#1a1a2e' });
      } else {
        uni.setNavigationBarColor({ frontColor: '#000000', backgroundColor: '#F8F8F8' });
      }
      this.saveState();
      this.showToast(on ? '✅深色模式已开启' : '✅深色模式已关闭');
    },
    switchLang(l) {
      this.S.settings.lang = l;
      this.saveState();
      this.showToast('✅语言已切换');
    },
    openDND() {
      this.dndStart = this.S.settings.dndStart;
      this.dndEnd = this.S.settings.dndEnd;
      this.openGeneralModal('dnd');
    },
    saveDND() {
      this.S.settings.dndStart = this.dndStart || '23:00';
      this.S.settings.dndEnd = this.dndEnd || '07:00';
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已设置');
    },
    openChangePwd() {
      this.oldPwd = '';
      this.newPwd = '';
      this.newPwdConfirm = '';
      this.openGeneralModal('change-pwd');
    },
    savePwd() {
      if (this.oldPwd !== this.S.pwd) { this.showToast('当前密码错误'); return; }
      if (!this.newPwd || this.newPwd.length < 4) { this.showToast('新密码至少4位'); return; }
      if (this.newPwd !== this.newPwdConfirm) { this.showToast('两次密码不一致'); return; }
      this.S.pwd = this.newPwd;
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅密码已修改');
    },

    // ========== 历史记录 ==========
    getHistoryIdx(day, i) {
      // 在分组历史中找到全局索引
      const days = Object.keys(this.groupedHistory);
      let offset = 0;
      for (let d of days) {
        if (d === day) return offset + i;
        offset += this.groupedHistory[d].length;
      }
      return i;
    },
    removeHistoryItem(i) {
      this.S.historyVids.splice(i, 1);
      this.saveState();
      this.showToast('已删除');
    },
    clearHistory() {
      this.deleteConfirmMsg = '确定清空所有历史？';
      this.deleteCallback = 'confirmClearHistory';
      this.openGeneralModal('confirm-delete');
    },
    confirmClearHistory() {
      this.S.historyVids = [];
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已清空');
    },
    executeDeleteAction() {
      this.closeGeneralModal();
      if (this.deleteCallback === 'confirmClearHistory') {
        this.confirmClearHistory();
      } else if (this.deleteCallback === 'confirmClearWatchLater') {
        this.confirmClearWatchLater();
      } else if (this.deleteCallback === 'confirmDeleteFav') {
        this.confirmDeleteFav(this.renameFavIdx);
      } else if (this.deleteCallback === 'confirmUninstallGame') {
        this.confirmUninstallGame(this.deleteConfirmMsg);
      } else if (this.deleteCallback === 'confirmUninstallGame2') {
        this.confirmUninstallGame2(this.deleteConfirmMsg);
      }
    },

    // ========== 收藏 ==========
    addFavFolder() {
      this.newFavName = '';
      this.newFavPriv = '公开';
      this.openGeneralModal('add-fav');
    },
    confirmAddFav() {
      if (!this.newFavName.trim()) { this.showToast('请输入名称'); return; }
      this.S.favFolders.push({ name: this.newFavName.trim(), priv: this.newFavPriv, items: 0, content: [] });
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已创建');
    },
    renameFav(i) {
      this.renameFavIdx = i;
      this.renameFavName = this.S.favFolders[i] ? this.S.favFolders[i].name : '';
      this.openGeneralModal('rename-fav');
    },
    confirmRenameFav(i) {
      if (this.renameFavName.trim()) this.S.favFolders[parseInt(i)].name = this.renameFavName.trim();
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已重命名');
    },
    toggleFavPriv(i) {
      var f = this.S.favFolders[parseInt(i)];
      if (f) f.priv = f.priv === '公开' ? '私密' : '公开';
      this.saveState();
      this.showToast('已设为' + this.S.favFolders[parseInt(i)].priv);
    },
    deleteFav(i) {
      var idx = parseInt(i);
      if (this.S.favFolders.length <= 1) { this.showToast('至少保留一个'); return; }
      this.renameFavIdx = idx;
      this.deleteConfirmMsg = '删除「' + this.S.favFolders[idx].name + '」？';
      this.deleteCallback = 'confirmDeleteFav';
      this.openGeneralModal('confirm-delete');
    },
    confirmDeleteFav(i) {
      this.S.favFolders.splice(parseInt(i), 1);
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已删除');
    },
    removeFavItem(fi, ii) {
      this.S.favFolders[parseInt(fi)].content.splice(parseInt(ii), 1);
      this.S.favFolders[parseInt(fi)].items = this.S.favFolders[parseInt(fi)].content.length;
      this.saveState();
      this.showToast('已移除');
    },

    // ========== 稍后再看 ==========
    removeWatchLater(i) {
      this.S.watchLater.splice(parseInt(i), 1);
      this.saveState();
      this.showToast('已移除');
    },
    clearWatchLater() {
      this.deleteConfirmMsg = '确定清空稍后再看？';
      this.deleteCallback = 'confirmClearWatchLater';
      this.openGeneralModal('confirm-delete');
    },
    confirmClearWatchLater() {
      this.S.watchLater = [];
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已清空');
    },

    // ========== 缓存 ==========
    removeCache(i) {
      this.S.cacheVids.splice(parseInt(i), 1);
      this.saveState();
      this.showToast('已删除');
    },

    // ========== 游戏 ==========
    getGameBg(g) {
      var map = { '崩坏：星穹铁道': '#eae6ff', '原神': '#fff0f4', '绝区零': '#e6f7fc', '黑神话：悟空': '#fff7e6' };
      return map[g] || '#eae6ff';
    },
    getGameInfo(g) {
      var map = { '崩坏：星穹铁道': '28.5GB·米哈游·RPG', '原神': '42.1GB·米哈游·开放世界', '绝区零': '32.7GB·米哈游·动作RPG', '黑神话：悟空': '78.3GB·游戏科学·动作' };
      return map[g] || '15GB·未知';
    },
    uninstallGame(n) {
      this.deleteConfirmMsg = n;
      this.deleteCallback = 'confirmUninstallGame';
      this.openGeneralModal('confirm-delete');
    },
    confirmUninstallGame(n) {
      this.S.games = this.S.games.filter(function(g) { return g !== n; });
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已卸载');
    },
    downloadGame(n, sz) {
      if (this.S.games.indexOf(n) >= 0) { this.showToast('已安装'); return; }
      var s = parseFloat(sz) || 15;
      if (this.S.storageUsed + s > this.S.storageTotal) { this.showToast('空间不足'); return; }
      this.showDlbar('⬇ ' + n + ' 下载中...');
      var id = 'dl_' + Date.now();
      this.S.downloads[id] = { name: n, size: s, progress: 0, status: 'ing' };
      this.saveState();
      var progress = 0;
      var self = this;
      var iv = setInterval(function() {
        progress += Math.random() * 20 + 10;
        if (progress >= 100) {
          progress = 100;
          clearInterval(iv);
          self.S.downloads[id].progress = 100;
          self.S.downloads[id].status = 'done';
          self.S.storageUsed += s;
          self.S.games.push(n);
          self.saveState();
          self.showDlbar('✅ ' + n + ' 下载完成！');
          setTimeout(function() { self.showToast('✅下载完成'); }, 1200);
        }
        self.S.downloads[id].progress = Math.min(progress, 100);
        self.saveState();
      }, 600);
    },
    uninstallGame2(n) {
      this.deleteConfirmMsg = n;
      this.deleteCallback = 'confirmUninstallGame2';
      this.openGeneralModal('confirm-delete');
    },
    confirmUninstallGame2(n) {
      this.S.games = this.S.games.filter(function(g) { return g !== n; });
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已卸载');
    },
    downloadGameFromStore(n, sz) {
      if (this.S.games.indexOf(n) >= 0) { this.showToast('已安装'); return; }
      var s = parseFloat(sz) || 15;
      if (this.S.storageUsed + s > this.S.storageTotal) { this.showToast('空间不足'); return; }
      this.showDlbar('⬇ ' + n + ' 下载中...');
      this.S.storageUsed += s;
      this.S.games.push(n);
      this.saveState();
      var self = this;
      setTimeout(function() {
        self.showDlbar('✅ ' + n + ' 下载完成！');
        self.showToast('✅下载完成');
      }, 2000);
    },

    // ========== VIP ==========
    selectVipTier(t, p) {
      if (this.S.balance < p) {
        this.showToast('余额不足！需¥' + p + '，当前¥' + this.S.balance.toFixed(2));
        return;
      }
      this.vipTierName = t;
      this.vipTierPrice = p;
      this.openGeneralModal('vip-purchase');
    },
    confirmVipPurchase(t, p) {
      this.S.balance -= p;
      this.S.vipType = t;
      var d = new Date(this.S.vipExp);
      if (t === '年度') d.setFullYear(d.getFullYear() + 1);
      else if (t === '季度') d.setMonth(d.getMonth() + 3);
      else d.setMonth(d.getMonth() + 1);
      this.S.vipExp = d.toISOString().split('T')[0];
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅已开通' + t + '大会员！');
    },
    rechargeVip() {
      this.rechargeAmount = '100';
      this.openGeneralModal('vip-recharge');
    },
    confirmRecharge() {
      var a = parseFloat(this.rechargeAmount) || 0;
      if (a <= 0) return;
      this.S.balance += a;
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅充值¥' + a);
    },

    // ========== 钱包 ==========
    rechargeWallet() {
      this.walletRechargeAmount = '100';
      this.openGeneralModal('wallet-recharge');
    },
    confirmWalletRecharge() {
      var a = parseFloat(this.walletRechargeAmount) || 0;
      if (a <= 0) return;
      this.S.balance += a;
      this.saveState();
      this.closeGeneralModal();
      this.showToast('✅充值¥' + a);
    },

    // ========== 购物车 ==========
    addToCart(n, p) {
      this.S.cart.push({ name: n, price: p });
      this.saveState();
      this.showToast('🛒已加入购物车');
    },
    openCart() {
      this.openGeneralModal('cart');
    },
    clearCart() {
      this.S.cart = [];
      this.saveState();
      this.closeGeneralModal();
      this.showToast('已清空购物车');
    },

    // ========== 漫画 ==========
    addComic(n, s) {
      if (this.S.comicShelf.find(function(c) { return c.t === n; })) {
        this.showToast('已在书架中'); return;
      }
      this.S.comicShelf.push({ t: n, s: s, tg: '新添加', tc: 'pk', bg: '#f6f0ff' });
      this.saveState();
      this.showToast('✅已加入书架');
    },
    removeComic(i) {
      this.S.comicShelf.splice(i, 1);
      this.saveState();
      this.showToast('已从书架移除');
    },

    // ========== 课程 ==========
    addCourse(n, s, ic, bg) {
      if (this.S.courses.find(function(c) { return c.t === n; })) {
        this.showToast('已添加'); return;
      }
      this.S.courses.push({ t: n, s: s, pr: 0, bg: bg || '#e6f7fc', ic: ic || '📖' });
      this.saveState();
      this.showToast('✅已添加');
    },
    removeCourse(i) {
      this.S.courses.splice(i, 1);
      this.saveState();
      this.showToast('已移除');
    },

    // ========== 投稿 ==========
    simulateFilePick() {
      this.videoTitle = '我的视频_' + new Date().toLocaleDateString();
      this.showToast('已选择: video.mp4 (856MB)');
    },
    publishVideo() {
      if (!this.videoTitle.trim()) { this.showToast('请输入标题'); return; }
      this.S.posts.unshift({
        type: 'video', title: this.videoTitle.trim(), time: '刚刚',
        meta: '审核中·' + this.videoCategory, icon: '🎬', bg: '#fee9e6', dur: '--:--'
      });
      this.S.dynamics.unshift({
        type: 'post', text: '发布了新视频：《' + this.videoTitle.trim() + '》',
        time: '刚刚', tag: '投稿', tc: 'pk'
      });
      this.saveState();
      this.closeSubPage();
      this.showToast('✅已提交审核');
    },
    onDynamicInput() {
      this.dynamicCharCount = (this.dynamicContent || '').length;
    },
    publishDynamic() {
      if (!this.dynamicContent || !this.dynamicContent.trim()) { this.showToast('请输入内容'); return; }
      this.S.dynamics.unshift({ type: 'post', text: this.dynamicContent.trim(), time: '刚刚', tag: '动态', tc: 'bl' });
      this.saveState();
      this.closeSubPage();
      this.showToast('✅动态已发布');
    },

    // ========== 客服 ==========
    leaveMessage() {
      this.msgContent = '';
      this.msgContact = this.S.phone;
      this.openGeneralModal('leave-message');
    },
    submitMessage() {
      if (!this.msgContent.trim()) { this.showToast('请输入内容'); return; }
      this.closeGeneralModal();
      this.showToast('✅留言已提交');
    },

    // ========== 主题 ==========
    applyTheme(id) {
      this.S.theme = id;
      this.saveState();
      this.showToast('✅主题已应用');
    },
    getThemeName(id) {
      var t = this.themeList.find(function(x) { return x.id === id; });
      return t ? t.name : '哔哩粉';
    }
  }
};
</script>
<style>
/* ====== CSS 变量: 挂载 page 根选择器 (非 :root, uniapp 无 body) ====== */
page {
  --pink: #fb7299;
  --pink2: #fc8bab;
  --pink3: #fff0f4;
  --blue: #00a1d6;
  --blue2: #e6f7fc;
  --bg: #f4f5f7;
  --bg2: #f1f2f5;
  --w: #fff;
  --t: #18191c;
  --t2: #9499a0;
  --t3: #c0c4cc;
  --t4: #61666d;
  --brd: #e3e5e7;
  --brd2: #f0f2f5;
  --sh: 0 2rpx 6rpx rgba(0,0,0,.06);
  --sh2: 0 2rpx 4rpx rgba(0,0,0,.04);
  --r: 16rpx;
  --r2: 24rpx;
  --r3: 28rpx;
  --nav-bg: #fff;
  --card-bg: #fff;
  --modal-bg: #fff;
  --input-bg: #f4f5f7;
  --overlay-bg: rgba(0,0,0,.45);
}

/* 深色模式: .ph.dark-mode 挂载在根 view (page 原生节点无法绑 class) */
.ph.dark-mode {
  --bg: #1a1a2e;
  --bg2: #222238;
  --w: #252540;
  --t: #e8e8f0;
  --t2: #9999aa;
  --t3: #666680;
  --t4: #8888a0;
  --brd: #333350;
  --brd2: #2a2a40;
  --sh: 0 2rpx 6rpx rgba(0,0,0,.3);
  --sh2: 0 2rpx 4rpx rgba(0,0,0,.2);
  --nav-bg: #1e1e32;
  --card-bg: #252540;
  --modal-bg: #252540;
  --input-bg: #1a1a2e;
  --overlay-bg: rgba(0,0,0,.7);
  --pink3: #3a1a28;
  --blue2: #1a2830;
}

/* ⚠️ 移除 * 选择器(App 不兼容), 各组件自行 box-sizing */

/* ====== 根容器 ====== */
.ph {
  width: 100%;
  height: 100vh;
  background: var(--bg);
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  padding-bottom: 100rpx;
  box-sizing: border-box;
  transition: background .4s;
}

/* ====== Toast ====== */
.toast {
  position: fixed;
  top: 48rpx;
  left: 50%;
  transform: translateX(-50%) translateY(-240rpx);
  background: #18191c;
  color: #fff;
  padding: 24rpx 56rpx;
  border-radius: 40rpx;
  font-size: 28rpx;
  z-index: 9999;
  pointer-events: none;
  transition: transform .35s cubic-bezier(.16,1,.3,1);
  box-shadow: 0 16rpx 48rpx rgba(0,0,0,.18);
  white-space: nowrap;
  max-width: 90vw;
  font-weight: 500;
}
.toast.show { transform: translateX(-50%) translateY(0); }

/* ====== 下载提示条 ====== */
.dlbar {
  position: fixed;
  top: 100rpx;
  left: 50%;
  transform: translateX(-50%);
  background: #18191c;
  color: #fff;
  padding: 20rpx 48rpx;
  border-radius: 40rpx;
  font-size: 26rpx;
  z-index: 999;
  pointer-events: none;
  opacity: 0;
  transition: opacity .3s;
  white-space: nowrap;
}
.dlbar.show { opacity: 1; }

/* ====== 主内容区 ====== */
.ms {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  position: relative;
  z-index: 1;
}

/* ====== 头部 ====== */
.pf {
  background: linear-gradient(160deg, #feabc2 0%, #e08a9e 40%, #f87193 100%);
  padding: 0 36rpx 40rpx;
  color: #fff;
  position: relative;
  overflow: hidden;
  z-index: 1;
  transition: box-shadow .3s;
}
.pf .pft {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20rpx 0 16rpx;
  position: relative;
  z-index: 2;
}
.hact {
  width: 72rpx;
  height: 72rpx;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  color: #fff;
  position: relative;
}
.hact-hover { background: rgba(255,255,255,.18); }
.hact-icon { width: 44rpx; height: 44rpx; }
.hact-icon-lg { width: 60rpx; height: 60rpx; }
.header-title { font-size: 32rpx; font-weight: 600; letter-spacing: 1rpx; }
.bdot {
  position: absolute;
  top: 12rpx;
  right: 12rpx;
  width: 14rpx;
  height: 14rpx;
  background: #ff4d4f;
  border-radius: 50%;
  border: 3rpx solid #fb7299;
}
.d1,.d2,.d3 { position: absolute; border-radius: 50%; background: rgba(255,255,255,.07); pointer-events: none; }
.d1 { width: 320rpx; height: 320rpx; top: -100rpx; right: -80rpx; }
.d2 { width: 200rpx; height: 200rpx; top: 100rpx; right: 160rpx; background: rgba(255,255,255,.05); }
.d3 { width: 120rpx; height: 120rpx; bottom: 20rpx; left: -40rpx; background: rgba(255,255,255,.04); }

/* 用户信息 */
.ur { display: flex; align-items: center; gap: 28rpx; position: relative; z-index: 2; }
.avw { position: relative; flex-shrink: 0; }
.avw .av {
  width: 112rpx; height: 112rpx; border-radius: 50%;
  background: rgba(255,255,255,.25);
  display: flex; align-items: center; justify-content: center;
  font-size: 60rpx; border: 4rpx solid rgba(255,255,255,.55);
}
.av-img { width: 100%; height: 100%; border-radius: 50%; }
.av-text { font-size: 60rpx; }
.avw .vring {
  position: absolute; inset: -6rpx; border-radius: 50%;
  border: 4rpx solid transparent;
  background: linear-gradient(135deg, #ffd700, #ffaa00, #ffd700) border-box;
  -webkit-mask: linear-gradient(#fff 0 0) padding-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}
.ui { flex: 1; min-width: 0; }
.ui .nm { font-size: 36rpx; font-weight: 650; letter-spacing: .6rpx; display: flex; align-items: center; gap: 16rpx; }
.ui .nm .lvt {
  display: inline-flex; align-items: center; gap: 4rpx;
  background: rgba(255,255,255,.22); padding: 4rpx 16rpx;
  border-radius: 20rpx; font-size: 20rpx; font-weight: 500; flex-shrink: 0;
}
.ui .uid { font-size: 24rpx; opacity: .8; font-weight: 400; display: flex; align-items: center; gap: 16rpx; }
.ui .uid .copy-uid { font-size: 20rpx; opacity: .65; padding: 2rpx 12rpx; border-radius: 16rpx; }
.ui .vb {
  display: inline-flex; align-items: center; gap: 4rpx;
  background: rgba(255,215,0,.22); padding: 6rpx 20rpx;
  border-radius: 40rpx; font-size: 22rpx; font-weight: 500; margin-top: 10rpx;
}

/* 统计 */
.st { display: flex; gap: 56rpx; margin-top: 36rpx; position: relative; z-index: 2; }
.st .si { text-align: center; }
.si-hover { opacity: .75; }
.st .si .sn { font-size: 34rpx; font-weight: 700; letter-spacing: .6rpx; }
.st .si .sl { font-size: 22rpx; opacity: .78; margin-top: 4rpx; }

/* 大会员卡片 */
.vc {
  margin: 24rpx 36rpx 0; padding: 26rpx 32rpx;
  background: rgba(255,255,255,.13); backdrop-filter: blur(8rpx);
  border-radius: var(--r2); display: flex; align-items: center;
  gap: 20rpx; position: relative; z-index: 2;
  border: 1rpx solid rgba(255,255,255,.18);
}
.vc-hover { background: rgba(255,255,255,.2); }
.vc .vi { width: 68rpx; height: 68rpx; border-radius: 16rpx; background: rgba(255,215,0,.25); display: flex; align-items: center; justify-content: center; font-size: 36rpx; flex-shrink: 0; }
.vc .vin { flex: 1; min-width: 0; }
.vc .vin .vt { font-size: 28rpx; font-weight: 600; letter-spacing: .6rpx; }
.vc .vin .vs { font-size: 22rpx; opacity: .75; margin-top: 4rpx; }
.vc .va { font-size: 36rpx; opacity: .7; flex-shrink: 0; }

/* ====== 功能板块 ====== */
.sc { background: var(--card-bg); margin: 20rpx 20rpx 0; border-radius: var(--r3); overflow: hidden; box-shadow: var(--sh2); transition: background .4s; }
.sh { padding: 26rpx 32rpx 10rpx; font-size: 26rpx; font-weight: 550; color: var(--t2); letter-spacing: .6rpx; display: flex; align-items: center; justify-content: space-between; }
.sh .sd { width: 8rpx; height: 32rpx; background: var(--pink); border-radius: 4rpx; margin-right: 12rpx; }

.g1 { display: flex; }
.g1-bordered { border-top: 2rpx solid var(--brd2); }
.g1 .gi { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 28rpx 12rpx 24rpx; text-align: center; gap: 12rpx; min-height: 160rpx; border-right: 2rpx solid var(--brd2); }
.g1 .gi:last-child { border-right: none; }
.gi-hover { background: var(--bg); }
.g1 .gi .ii { width: 72rpx; height: 72rpx; border-radius: 18rpx; display: flex; align-items: center; justify-content: center; font-size: 36rpx; flex-shrink: 0; }
.ii-img { width: 48rpx; height: 48rpx; }
.g1 .gi .il { font-size: 24rpx; color: var(--t); font-weight: 460; }
.g1 .gi .ib { font-size: 20rpx; color: var(--t2); background: var(--bg2); padding: 2rpx 16rpx; border-radius: 40rpx; }
.g1 .gi .it { font-size: 18rpx; padding: 2rpx 12rpx; border-radius: 20rpx; font-weight: 500; }
.g1 .gi .it.ht { background: var(--pink3); color: var(--pink); }
.g1 .gi .it.nw { background: var(--blue2); color: var(--blue); }

/* 更多服务行 */
.mr { display: flex; align-items: center; padding: 28rpx 32rpx; gap: 24rpx; border-bottom: 2rpx solid var(--brd2); }
.mr:last-child { border-bottom: none; }
.mr-hover { background: var(--bg); }
.mr .ri { width: 68rpx; height: 68rpx; border-radius: 16rpx; display: flex; align-items: center; justify-content: center; font-size: 36rpx; flex-shrink: 0; }
.ri-img { width: 48rpx; height: 48rpx; }
.mr .rl { flex: 1; font-size: 30rpx; color: var(--t); font-weight: 460; }
.mr .re { font-size: 24rpx; color: var(--t2); margin-right: 4rpx; }
.mr .ra { color: var(--t3); font-size: 32rpx; flex-shrink: 0; }

/* ====== 底部按钮 ====== */
.lw { padding: 44rpx 20rpx 50rpx; text-align: center; display: flex; gap: 20rpx; }
.lb { flex: 1; height: 92rpx; border: 2rpx solid #ff4d4f; background: var(--card-bg); color: #ff4d4f; border-radius: 46rpx; font-size: 30rpx; font-weight: 500; display: flex; align-items: center; justify-content: center; transition: all .2s; letter-spacing: 2rpx; }
.lb-hover { background: #fff1f0; transform: scale(.985); }
.lb2 { flex: 1; height: 92rpx; border: 2rpx solid var(--pink); background: var(--card-bg); color: var(--pink); border-radius: 46rpx; font-size: 30rpx; font-weight: 500; display: flex; align-items: center; justify-content: center; }
.lb2-hover { background: var(--pink3); }

/* ====== 弹窗 ====== */
.mo { position: absolute; inset: 0; background: var(--overlay-bg); z-index: 500; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity .25s; }
.mo.s { opacity: 1; pointer-events: auto; }
.md { background: var(--modal-bg); border-radius: var(--r3); width: 600rpx; padding: 48rpx 40rpx; text-align: center; box-shadow: 0 40rpx 120rpx rgba(0,0,0,.2); transform: scale(.9); transition: transform .25s cubic-bezier(.16,1,.3,1); max-height: 80vh; overflow-y: auto; }
.mo.s .md { transform: scale(1); }
.md .mt { font-size: 34rpx; font-weight: 600; color: var(--t); margin-bottom: 16rpx; }
.md .mdesc { font-size: 26rpx; color: var(--t2); margin-bottom: 32rpx; line-height: 1.5; }
.md .mbs { display: flex; gap: 20rpx; }
.md .mbs view, .md .mbs .bc, .md .mbs .bf, .md .mbs .bp {
  flex: 1; height: 84rpx; border-radius: 42rpx; display: flex; align-items: center; justify-content: center; font-size: 30rpx; font-weight: 500; border: none;
}
.md .bc { background: var(--bg); color: var(--t); }
.bc-hover { background: #e4e6e9; }
.md .bf { background: #ff4d4f; color: #fff; }
.bf-hover { background: #e0393b; }
.md .bp { background: var(--pink); color: #fff; }
.bp-hover { background: #e86082; }

/* ====== Spo 子页面浮层 ====== */
.spo { position: absolute; inset: 0; z-index: 300; background: var(--bg); display: flex; flex-direction: column; transform: translateX(105%); transition: transform .32s cubic-bezier(.22,1,.36,1); }
.spo.s { transform: translateX(0); }
.spo.ha { transform: translateX(105%); }
.sph { height: 96rpx; min-height: 96rpx; background: var(--nav-bg); display: flex; align-items: center; padding: 0 24rpx; border-bottom: 2rpx solid var(--brd); gap: 16rpx; flex-shrink: 0; }
.spb { width: 72rpx; height: 72rpx; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 40rpx; color: var(--t); flex-shrink: 0; }
.spb-hover { background: var(--bg); }
.spt { flex: 1; font-size: 34rpx; font-weight: 600; color: var(--t); text-align: center; margin-right: 72rpx; }
.spa { width: 72rpx; height: 72rpx; font-size: 36rpx; display: flex; align-items: center; justify-content: center; flex-shrink: 0; color: var(--t2); position: relative; }
.spa-text { font-size: 26rpx; color: var(--blue); }
.spa-text-danger { font-size: 26rpx; color: #ff4d4f; }
.spa-text-link { font-size: 26rpx; color: var(--blue); }
.spbd { flex: 1; overflow-y: auto; padding-bottom: 40rpx; }

/* ====== 子页面通用组件 ====== */
.ss { background: var(--card-bg); margin: 20rpx 24rpx; border-radius: var(--r2); overflow: hidden; box-shadow: var(--sh2); transition: background .4s; }
.ssh { padding: 28rpx 32rpx 16rpx; font-size: 26rpx; font-weight: 550; color: var(--t2); letter-spacing: .6rpx; display: flex; align-items: center; justify-content: space-between; }
.ssh-link { font-size: 24rpx; color: var(--blue); font-weight: 400; }

/* 列表行 */
.sli { display: flex; align-items: center; padding: 28rpx 32rpx; gap: 24rpx; border-bottom: 2rpx solid var(--brd2); }
.sli:last-child { border-bottom: none; }
.sth { width: 88rpx; height: 88rpx; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 44rpx; flex-shrink: 0; overflow: hidden; }
.sth.r8 { border-radius: 16rpx; }
.sif { flex: 1; min-width: 0; }
.sif .sit { font-size: 30rpx; color: var(--t); font-weight: 460; line-height: 1.3; }
.sif .sis { font-size: 24rpx; color: var(--t2); margin-top: 4rpx; }
.sib { font-size: 24rpx; color: var(--t2); margin-right: 8rpx; flex-shrink: 0; }
.sir { color: var(--t3); font-size: 32rpx; flex-shrink: 0; }

/* 视频卡片 */
.svi { display: flex; padding: 24rpx 32rpx; gap: 24rpx; border-bottom: 2rpx solid var(--brd2); }
.svi:last-child { border-bottom: none; }
.svt { width: 224rpx; height: 140rpx; border-radius: 12rpx; flex-shrink: 0; display: flex; align-items: center; justify-content: center; font-size: 56rpx; position: relative; overflow: hidden; }
.svt-ic { font-size: 56rpx; }
.svt .svd { position: absolute; bottom: 6rpx; right: 8rpx; background: rgba(0,0,0,.7); color: #fff; font-size: 20rpx; padding: 2rpx 10rpx; border-radius: 6rpx; }
.svi .svf { flex: 1; min-width: 0; display: flex; flex-direction: column; justify-content: center; }
.svi .svf .svt2 { font-size: 28rpx; color: var(--t); line-height: 1.4; overflow: hidden; margin-bottom: 8rpx; }
.svi .svf .svm { font-size: 22rpx; color: var(--t2); }
.svi .svf .svp { height: 6rpx; background: #e4e6e9; border-radius: 4rpx; margin-top: 12rpx; overflow: hidden; }
.svi .svf .svp .svb { height: 100%; background: var(--pink); border-radius: 4rpx; }

/* 设置行 */
.srw { display: flex; align-items: center; padding: 30rpx 32rpx; border-bottom: 2rpx solid var(--brd2); gap: 16rpx; }
.srw:last-child { border-bottom: none; }
.srw .slb { flex: 1; font-size: 30rpx; color: var(--t); }
.srw .sva { font-size: 26rpx; color: var(--t2); margin-right: 8rpx; }

/* 开关组件 (Toggle) */
.tg-wrap { position: relative; display: inline-block; width: 96rpx; height: 56rpx; flex-shrink: 0; }
.tg-track { width: 100%; height: 100%; border-radius: 28rpx; background: #d1d5db; transition: background .2s; position: relative; }
.tg-track.on { background: #fb7299; }
.tg-dot { position: absolute; top: 6rpx; left: 6rpx; width: 44rpx; height: 44rpx; border-radius: 50%; background: #fff; transition: left .2s; box-shadow: 0 2rpx 6rpx rgba(0,0,0,.15); }
.tg-track.on .tg-dot { left: 46rpx; }

/* 网格 */
.sg3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 2rpx; background: var(--brd2); }
.sg3 .sgi { background: var(--card-bg); padding: 32rpx 16rpx; text-align: center; }
.sg3 .sgi .sgic { font-size: 52rpx; margin-bottom: 8rpx; }
.sg3 .sgi .sgil { font-size: 22rpx; color: var(--t); }

.sg4 { display: grid; grid-template-columns: repeat(4, 1fr); gap: 2rpx; background: var(--brd2); }
.sg4 .sgi { background: var(--card-bg); padding: 28rpx 12rpx; text-align: center; }
.sg4 .sgi .sgic { font-size: 48rpx; margin-bottom: 6rpx; }
.sg4 .sgi .sgil { font-size: 20rpx; color: var(--t); }

.sg2stat { display: grid; grid-template-columns: repeat(3, 1fr); gap: 2rpx; background: var(--brd2); }
.sg2stat .scd { background: var(--card-bg); padding: 36rpx 20rpx; text-align: center; transition: background .4s; }
.sg2stat .scd .scdn { font-size: 44rpx; font-weight: 700; color: var(--t); }
.sg2stat .scd .scdn.pk { color: var(--pink); }
.sg2stat .scd .scdn.bl { color: var(--blue); }
.sg2stat .scd .scdl { font-size: 24rpx; color: var(--t2); margin-top: 8rpx; }

/* 标签页 */
.stabs { display: flex; background: var(--card-bg); border-bottom: 2rpx solid var(--brd); overflow-x: auto; flex-shrink: 0; }
.stab { padding: 24rpx 28rpx; font-size: 28rpx; color: var(--t2); border-bottom: 4rpx solid transparent; transition: all .2s; white-space: nowrap; flex-shrink: 0; }
.stab.a { color: var(--pink); border-bottom-color: var(--pink); font-weight: 500; }

/* Badge 标签 */
.sbd { display: inline-block; padding: 4rpx 16rpx; border-radius: 20rpx; font-size: 22rpx; font-weight: 500; }
.sbd.pk { background: var(--pink3); color: var(--pink); }
.sbd.bl { background: var(--blue2); color: var(--blue); }
.sbd.gn { background: #e8f8e8; color: #52c41a; }
.sbd.or { background: #fff7e6; color: #fa8c16; }
.sbd.gy { background: var(--bg2); color: var(--t2); }
.sbd.rd { background: #fff1f0; color: #ff4d4f; }

/* 空状态 */
.se { display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 120rpx 40rpx; text-align: center; }
.se .sei { font-size: 104rpx; margin-bottom: 28rpx; opacity: .6; }
.se .set { font-size: 28rpx; color: var(--t2); margin-bottom: 32rpx; }
.se .seb { display: inline-block; padding: 16rpx 48rpx; background: var(--pink); color: #fff; border-radius: 40rpx; font-size: 28rpx; font-weight: 500; }

/* 按钮 */
.sbtn { display: inline-flex; align-items: center; justify-content: center; gap: 8rpx; padding: 14rpx 32rpx; border-radius: 32rpx; font-size: 24rpx; font-weight: 500; border: none; }
.sbtn.pk { background: var(--pink); color: #fff; }
.sbtn.ol { background: transparent; color: var(--pink); border: 2rpx solid var(--pink); }
.sbtn.gy { background: var(--bg); color: var(--t); }
.sbtn.sm { padding: 8rpx 20rpx; font-size: 22rpx; border-radius: 20rpx; }
.sbtn.rd { background: transparent; color: #ff4d4f; border: 2rpx solid #ff4d4f; }
.sbtn.gn { background: #e8f8e8; color: #52c41a; }
.save-btn { width: 100%; height: 88rpx; border-radius: 44rpx; font-size: 32rpx; }

/* 进度条 */
.spr { height: 12rpx; background: #e4e6e9; border-radius: 6rpx; overflow: hidden; }
.spr .sprb { height: 100%; border-radius: 6rpx; background: var(--pink); }
.spr .sprb.bl { background: var(--blue); }
.spr .sprb.gn { background: #52c41a; }

/* 图标卡片 */
.sic { margin: 0 24rpx; padding: 40rpx 36rpx; background: var(--card-bg); border-radius: var(--r2); text-align: center; box-shadow: var(--sh2); transition: background .4s; }
.sic .sicic { font-size: 80rpx; margin-bottom: 20rpx; }
.sic .sict { font-size: 36rpx; font-weight: 600; color: var(--t); margin-bottom: 12rpx; }

/* Hero 卡片 */
.shero { margin: 20rpx 24rpx; padding: 48rpx 36rpx; border-radius: var(--r2); color: #fff; position: relative; overflow: hidden; box-shadow: 0 8rpx 24rpx rgba(0,0,0,.15); }
.shero .sht { font-size: 36rpx; font-weight: 650; margin-bottom: 8rpx; position: relative; z-index: 1; }
.shero .shs { font-size: 26rpx; opacity: .85; position: relative; z-index: 1; }
.shero .shi { position: absolute; right: 28rpx; top: 50%; transform: translateY(-50%); font-size: 96rpx; opacity: .3; }
.shero.pb { background: linear-gradient(135deg, #fb7299, #f85a7f); }
.shero.prb { background: linear-gradient(135deg, #7c5cfc, #6a4ff0); }
.sddiv { padding: 32rpx 32rpx 12rpx; font-size: 26rpx; font-weight: 550; color: var(--t2); }
.sqa { display: flex; gap: 20rpx; padding: 24rpx 32rpx; flex-wrap: wrap; }

/* 钱包 */
.swb { background: linear-gradient(135deg, #2d2d3a, #1a1a2e); margin: 20rpx 24rpx; padding: 40rpx; border-radius: var(--r2); color: #fff; }
.swb .swbt { font-size: 64rpx; font-weight: 700; }
.swb .swbl { font-size: 24rpx; opacity: .7; margin-top: 4rpx; }
.swb .swbr { display: flex; gap: 48rpx; margin-top: 32rpx; }
.swb .swbr .swbi { flex: 1; }
.swb .swbr .swbi .swbn { font-size: 36rpx; font-weight: 600; }
.swb .swbr .swbi .swbs { font-size: 22rpx; opacity: .7; margin-top: 4rpx; }

/* 音频播放器 */
.sap { background: linear-gradient(135deg, #1a1a2e, #2d2d3a); margin: 20rpx 24rpx; padding: 56rpx 40rpx; border-radius: var(--r2); text-align: center; color: #fff; }
.sap .sapic { font-size: 96rpx; margin-bottom: 24rpx; }
.sap .sapt { font-size: 32rpx; font-weight: 500; margin-bottom: 32rpx; }
.sap .sapc { display: flex; align-items: center; justify-content: center; gap: 48rpx; }
.sap .sapb { width: 96rpx; height: 96rpx; border-radius: 50%; background: rgba(255,255,255,.15); display: flex; align-items: center; justify-content: center; font-size: 44rpx; }
.sap .sapb.play { width: 120rpx; height: 120rpx; font-size: 56rpx; background: var(--pink); }

/* 扫码 */
.sscan { text-align: center; padding: 80rpx 40rpx; }
.sscan .sscf { width: 400rpx; height: 400rpx; margin: 0 auto 40rpx; border: 4rpx dashed var(--brd); border-radius: var(--r2); display: flex; align-items: center; justify-content: center; font-size: 128rpx; opacity: .5; }
.sscan .ssct { font-size: 28rpx; color: var(--t2); }

/* VIP 方案 */
.svc { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20rpx; padding: 0 32rpx; }
.svc .svtier { background: var(--card-bg); border-radius: var(--r2); padding: 40rpx 28rpx; text-align: center; box-shadow: var(--sh2); border: 4rpx solid transparent; }
.svc .svtier.cur { border-color: #ffd700; background: #fffef5; }
.svc .svtier .svti { font-size: 72rpx; margin-bottom: 16rpx; }
.svc .svtier .svtn { font-size: 32rpx; font-weight: 600; color: var(--t); margin-bottom: 8rpx; }
.svc .svtier .svtp { font-size: 36rpx; font-weight: 700; color: var(--pink); margin-bottom: 16rpx; }
.svc .svtier .svtb { font-size: 22rpx; color: var(--t2); line-height: 1.6; }

/* FAQ */
.sfaq { padding: 28rpx 32rpx; border-bottom: 2rpx solid var(--brd2); }
.sfaq:last-child { border-bottom: none; }
.sfaq .sfq { font-size: 28rpx; color: var(--t); font-weight: 500; display: flex; align-items: center; justify-content: space-between; }
.sfaq .sfa { font-size: 26rpx; color: var(--t2); margin-top: 16rpx; line-height: 1.5; display: none; }
.sfaq.op .sfa { display: block; }
.sfaq.op .sfq { color: var(--pink); }

/* ====== Profile 子页面专用 ====== */
.pf-banner { background: url('../../mine_images/banner_bg.jpg') no-repeat center center; background-size: cover; position: relative; height: 320rpx; overflow: hidden; }
.pf-bg { position: absolute; inset: 0; transition: background .4s ease; }
.pf-bg-btn { position: absolute; bottom: 20rpx; right: 28rpx; background: rgba(0,0,0,.45); color: #fff; padding: 8rpx 24rpx; border-radius: 28rpx; font-size: 22rpx; z-index: 2; }
.pf-info-section { padding: 0 32rpx; background: var(--card-bg); position: relative; transition: background .4s; }
.pf-info-top { display: flex; align-items: flex-end; gap: 24rpx; }
.pfab { width: 136rpx; height: 136rpx; border-radius: 50%; border: 6rpx solid var(--card-bg); margin-top: -68rpx; position: relative; z-index: 2; display: flex; align-items: center; justify-content: center; font-size: 72rpx; background: linear-gradient(135deg, #f0e0e8, #e8d8e0); box-shadow: 0 4rpx 16rpx rgba(0,0,0,.1); overflow: hidden; }
.pfab-img { width: 100%; height: 100%; border-radius: 50%; }
.pfab-text { font-size: 72rpx; }
.pf-name-row { display: flex; align-items: center; gap: 16rpx; margin-top: 16rpx; }
.pf-name { font-size: 40rpx; font-weight: 650; color: var(--t); }
.pf-meta { font-size: 24rpx; color: var(--t2); padding: 8rpx 0; display: block; }
.pf-bio { font-size: 24rpx; color: var(--t4); padding-bottom: 20rpx; display: block; }
.pf-stats-row { display: flex; gap: 0; padding: 28rpx 32rpx; background: var(--card-bg); border-top: 2rpx solid var(--brd2); border-bottom: 2rpx solid var(--brd2); }
.pf-stats-row .pf-stat { flex: 1; text-align: center; }
.pf-stats-row .pf-stat + .pf-stat { border-left: 2rpx solid var(--brd2); }
.pf-stats-row .pf-stat .pf-sn { font-size: 34rpx; font-weight: 600; color: var(--t); }
.pf-stats-row .pf-stat .pf-sl { font-size: 22rpx; color: var(--t2); margin-top: 4rpx; }
.pf-tab-panel { display: none; padding-bottom: 40rpx; }
.pf-tab-panel.a { display: block; }

/* ====== 收藏夹 ====== */
.fav-btns { padding: 16rpx 32rpx; display: flex; gap: 12rpx; background: var(--card-bg); }
.fav-del { font-size: 36rpx; padding: 0 8rpx; color: var(--t3); flex-shrink: 0; }

/* ====== 购物车弹窗 ====== */
.cart-header { font-weight: 600; color: var(--t); padding: 16rpx 0; font-size: 30rpx; }
.cart-item { display: flex; justify-content: space-between; padding: 16rpx 0; border-bottom: 2rpx solid var(--brd2); }
.cart-item-name { font-size: 26rpx; color: var(--t); }
.cart-item-price { font-size: 26rpx; color: var(--pink); }
.cart-total { text-align: right; padding: 16rpx 0; font-weight: 600; color: var(--t); font-size: 28rpx; }
.cart-count-badge { position: absolute; top: -12rpx; right: -16rpx; background: #ff4d4f; color: #fff; font-size: 20rpx; padding: 2rpx 10rpx; border-radius: 16rpx; min-width: 32rpx; text-align: center; }

/* ====== 标签选择 ====== */
.tag-chips { display: flex; flex-wrap: wrap; gap: 12rpx; padding: 8rpx 0; justify-content: center; }
.tag-chip { padding: 10rpx 24rpx; border-radius: 28rpx; font-size: 24rpx; border: 2rpx solid var(--brd); background: var(--bg); color: var(--t2); transition: all .15s; white-space: nowrap; }
.tag-chip.sel { border-color: var(--pink); background: var(--pink3); color: var(--pink); }

/* ====== 头像选择器 ====== */
.avatar-grid-wrap { max-height: 400rpx; overflow-y: auto; }
.avatar-grid { display: flex; flex-wrap: wrap; gap: 20rpx; justify-content: center; margin: 32rpx 0; }
.avatar-opt { font-size: 72rpx; padding: 16rpx; border-radius: 24rpx; border: 4rpx solid transparent; background: var(--bg); display: inline-flex; align-items: center; justify-content: center; width: 104rpx; height: 104rpx; }
.avatar-opt.sel { border-color: var(--pink); background: var(--pink3); }
.avatar-img-small { width: 72rpx; height: 72rpx; border-radius: 50%; }
.avatar-emoji { font-size: 72rpx; }

/* ====== 编辑资料专用 ====== */
.ep-avatar-row { padding: 20rpx 32rpx; display: flex; align-items: center; gap: 24rpx; border-bottom: 2rpx solid var(--brd2); }
.ep-label { font-size: 26rpx; color: var(--t); width: 120rpx; }
.ep-avatar { font-size: 80rpx; border: 4rpx solid var(--pink); border-radius: 24rpx; padding: 8rpx; display: flex; align-items: center; justify-content: center; width: 80rpx; height: 80rpx; overflow: hidden; }
.ep-avatar-img { width: 80rpx; height: 80rpx; border-radius: 50%; }
.ep-avatar-text { font-size: 80rpx; }
.ep-link { font-size: 24rpx; color: var(--blue); }
.ep-gender-row { display: flex; gap: 16rpx; }
.ep-gender-row text { padding: 8rpx 24rpx; border-radius: 12rpx; font-size: 26rpx; border: 2rpx solid var(--brd); color: var(--t2); }
.ep-gender-row text.active { border-color: var(--pink); background: var(--pink3); color: var(--pink); }
.fav-priv-row { display: flex; gap: 24rpx; justify-content: center; margin: 16rpx 0; }
.fav-priv-opt { padding: 10rpx 32rpx; border-radius: 16rpx; font-size: 26rpx; border: 2rpx solid var(--brd); color: var(--t2); }
.fav-priv-opt.active { border-color: var(--pink); background: var(--pink3); color: var(--pink); }
.dnd-row { display: flex; gap: 20rpx; align-items: center; justify-content: center; margin: 24rpx 0; }

/* ====== 上传区域 ====== */
.upload-area { padding: 80rpx; text-align: center; border: 4rpx dashed var(--brd); border-radius: var(--r2); margin: 32rpx; }
.upload-icon { font-size: 96rpx; display: block; }
.upload-text { font-size: 28rpx; color: var(--t2); margin-top: 16rpx; display: block; }

/* ====== 底部导航 ====== */
.bn { display: flex; height: 112rpx; background: var(--nav-bg); border-top: 2rpx solid var(--brd); flex-shrink: 0; padding-bottom: env(safe-area-inset-bottom, 0); z-index: 100; transition: background .4s; }
.bn .ni { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 4rpx; color: var(--t2); font-size: 22rpx; position: relative; }
.bn .ni .nis { width: 52rpx; height: 52rpx; display: flex; align-items: center; justify-content: center; font-size: 44rpx; line-height: 1; }
.bn .ni.a { color: var(--pink); font-weight: 600; }
.bn .ni .ndot { position: absolute; top: 8rpx; width: 12rpx; height: 12rpx; background: #ff4d4f; border-radius: 50%; }
.fs { height: 20rpx; }

/* ====== 输入框 ====== */
.mi { width: 100%; padding: 20rpx 28rpx; border: 2rpx solid var(--brd); border-radius: var(--r); font-size: 30rpx; outline: none; color: var(--t); background: var(--input-bg); margin-bottom: 24rpx; transition: background .4s; box-sizing: border-box; }
.mi-ta { height: 120rpx; resize: none; }

/* 分段选择器 (Segmented Control) */
.seg-row { display: flex; gap: 8rpx; }
.seg-opt { padding: 8rpx 20rpx; border-radius: 12rpx; font-size: 24rpx; border: 2rpx solid var(--brd); color: var(--t2); white-space: nowrap; }
.seg-opt.sel { border-color: var(--pink); background: var(--pink3); color: var(--pink); }

/* ====== 课程网格 ====== */
.c2 { display: grid; grid-template-columns: 1fr 1fr; gap: 20rpx; padding: 20rpx 24rpx; }
.c2 .cc { background: var(--card-bg); border-radius: var(--r2); overflow: hidden; box-shadow: var(--sh2); }

/* ====== 入场动画 ====== */
@keyframes fi { from { opacity: 0; transform: translateY(16rpx); } to { opacity: 1; transform: translateY(0); } }
.sc { animation: fi .35s ease backwards; }
.sc:nth-child(2) { animation-delay: .04s; }
.sc:nth-child(3) { animation-delay: .08s; }
.sc:nth-child(4) { animation-delay: .12s; }
.sc:nth-child(5) { animation-delay: .16s; }
.sc:nth-child(6) { animation-delay: .20s; }
.sc:nth-child(7) { animation-delay: .24s; }
</style>
