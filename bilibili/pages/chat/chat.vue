<template>
    <view class="chat-page">
        <!-- 自定义头部 - 修复高度问题 -->
        <view class="detail-header">
            <view class="back-btn" @click="goBack">
                <text class="icon-back">‹</text>
            </view>
            <view class="header-title-wrapper">
                <text class="header-title">{{ chatInfo.title }}</text>
                <view v-if="chatInfo.isLive" class="live-tag">
                    <svg class="live-icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
                        <path
                            d="M886.4768 273.0496c19.456 0 35.1232 15.3088 35.1232 34.1504v477.8496a34.6112 34.6112 0 0 1-35.1232 34.1504 34.6112 34.6112 0 0 1-35.072-34.1504V307.2c0-18.8416 15.7184-34.1504 35.072-34.1504z m-187.2384 227.584c19.4048 0 35.1232 15.2576 35.1232 34.0992v250.368a34.6112 34.6112 0 0 1-35.1232 34.0992 34.6112 34.6112 0 0 1-35.072-34.1504v-250.3168c0-18.8416 15.6672-34.0992 35.072-34.0992z m-187.2384-204.8c19.4048 0 35.1232 15.2576 35.1232 34.0992v455.168A34.6112 34.6112 0 0 1 512 819.2a34.6112 34.6112 0 0 1-35.1232-34.1504V329.984c0-18.8928 15.7184-34.1504 35.1232-34.1504zM324.7616 204.8c19.4048 0 35.072 15.3088 35.072 34.1504v546.0992a34.6112 34.6112 0 0 1-35.072 34.1504 34.6112 34.6112 0 0 1-35.1232-34.1504V238.9504c0-18.8416 15.7184-34.1504 35.1232-34.1504zM137.5232 477.8496c19.3536 0 35.072 15.3088 35.072 34.1504v273.0496a34.6112 34.6112 0 0 1-35.072 34.1504 34.6112 34.6112 0 0 1-35.1232-34.1504V512c0-18.8416 15.7184-34.1504 35.1232-34.1504z"
                            fill="currentColor" />
                    </svg>
                    <text>直播中</text>
                </view>
            </view>
            <view class="header-right" @click="openSettings">
                <svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
                    <path
                        d="M512 332.8a179.2 179.2 0 1 0 0 358.4 179.2 179.2 0 0 0 0-358.4z m0 308.992c-70.6048 0-128-58.2144-128-129.792S441.3952 382.208 512 382.208c70.5536 0 128 58.2144 128 129.792s-57.4464 129.792-128 129.792z"
                        fill="currentColor" />
                    <path
                        d="M795.9552 193.7408l-153.6-91.3408a254.8224 254.8224 0 0 1-260.7104 0l-153.6 91.3408c0 90.9312-47.7696 175.104-125.6448 221.44v193.6896a257.536 257.536 0 0 1 125.6448 221.44l153.6 91.3408A254.8224 254.8224 0 0 1 642.3552 921.6l153.6-91.3408c0-90.9312 47.7696-175.104 125.6448-221.44V415.1808a257.5872 257.5872 0 0 1-125.6448-221.44zM870.4 581.2224a311.0912 311.0912 0 0 0-124.16 218.7776l-105.472 62.72a305.9712 305.9712 0 0 0-257.6384 0l-105.472-62.72A310.784 310.784 0 0 0 153.6 581.2224V442.7776a311.0912 311.0912 0 0 0 124.16-218.7776l105.472-62.72a305.8688 305.8688 0 0 0 257.5872-0.0512l105.472 62.72A310.9888 310.9888 0 0 0 870.4 442.7776v138.4448z"
                        fill="currentColor" />
                </svg>
            </view>
        </view>

        <!-- 聊天内容区 -->
        <scroll-view scroll-y class="chat-content" :scroll-into-view="scrollIntoView" scroll-with-animation
            enable-back-to-top>

            <!-- ================= ID 2: 系统通知列表 ================= -->
            <block v-if="chatInfo.id === 2">
                <!-- 顶部操作栏 -->
                <view class="action-bar">
                    <text class="action-left">全部消息</text>
                    <text class="action-right" @click="editMode">编辑</text>
                </view>

                <!-- 通知列表 -->
                <view class="notice-list">
                    <view class="notice-item" v-for="(item, index) in notices" :key="index" @click="openNotice(item)">
                        <view class="notice-content">
                            <text class="notice-title">{{ item.title }}</text>
                            <text class="notice-body">{{ item.content }}</text>
                            <view v-if="item.link" class="notice-link">
                                <text class="link-text">{{ item.linkText }}</text>
                            </view>
                        </view>
                        <text class="notice-time">{{ item.time }}</text>
                    </view>
                </view>
            </block>

            <!-- ================= ID 1: 系统通知卡片 ================= -->
            <block v-else-if="chatInfo.id === 1">
                <!-- 顶部操作栏 - 优化样式 -->
                <view class="notice-actions">
                    <view class="action-group-left">
                        <svg class="action-icon" viewBox="0 0 1024 1024" width="200" height="200">
                            <path
                                d="M795.9552 193.7408l-153.6-91.3408a254.8224 254.8224 0 0 1-260.7104 0l-153.6 91.3408c0 90.9312-47.7696 175.104-125.6448 221.44v193.6896a257.536 257.536 0 0 1 125.6448 221.44l153.6 91.3408A254.8224 254.8224 0 0 1 642.3552 921.6l153.6-91.3408c0-90.9312 47.7696-175.104 125.6448-221.44V415.1808a257.5872 257.5872 0 0 1-125.6448-221.44zM870.4 581.2224a311.0912 311.0912 0 0 0-124.16 218.7776l-105.472 62.72a305.9712 305.9712 0 0 0-257.6384 0l-105.472-62.72A310.784 310.784 0 0 0 153.6 581.2224V442.7776a311.0912 311.0912 0 0 0 124.16-218.7776l105.472-62.72a305.8688 305.8688 0 0 0 257.5872-0.0512l105.472 62.72A310.9888 310.9888 0 0 0 870.4 442.7776v138.4448z"
                                fill="currentColor" />
                        </svg>
                        <text class="action-text">不接受推送</text>
                        <text class="action-subtext">通知正常接收</text>
                    </view>
                    <view class="divider-vertical"></view>
                    <view class="action-group-right">
                        <svg class="action-icon" viewBox="0 0 1024 1024" width="200" height="200">
                            <path
                                d="M512.153548 1024C229.830338 1024 0.143311 794.312974 0.143311 511.989763S229.830338 0 512.153548 0s512.010237 229.687026 512.010236 512.010237-229.687026 511.989763-512.010236 511.989763z m0-970.114961C259.557191 53.885039 54.048823 259.393406 54.048823 511.989763S259.557191 970.094488 512.153548 970.094488s458.104724-205.508367 458.104724-458.104725S764.749905 53.885039 512.153548 53.885039z"
                                fill="currentColor" />
                            <path
                                d="M716.8 460.8c0-113.101-91.699-204.8-204.8-204.8S307.2 347.699 307.2 460.8 398.899 665.6 512 665.6s204.8-91.699 204.8-204.8z"
                                fill="currentColor" />
                        </svg>
                        <text class="action-text">加关注</text>
                    </view>
                </view>

                <!-- 通知卡片 -->
                <view class="notice-card">
                    <text class="card-title">登录操作通知</text>
                    <text class="card-body">你的账号在新设备或平台登录成功，如非本人操作，请及时修改密码（密码修改成功，全平台清空登录态）</text>
                    <view class="card-row">
                        <text class="label">设备/平台</text>
                        <text class="value">Edge浏览器</text>
                    </view>
                    <view class="card-row">
                        <text class="label">登录方式</text>
                        <text class="value">第三方登录</text>
                    </view>
                    <view class="card-row">
                        <text class="label">参考登录地</text>
                        <text class="value">中国海南三亚</text>
                    </view>
                    <view class="card-row">
                        <text class="label">登录时间</text>
                        <text class="value">2026-03-08 09:09:13</text>
                    </view>
                    <view class="card-footer">
                        <text>查看详情</text>
                        <text class="arrow">›</text>
                    </view>
                </view>

                <view class="time-divider">2026年3月25日 13:30</view>

                <view class="notice-card">
                    <text class="card-title">登录操作通知</text>
                    <text class="card-body">你的账号在新设备或平台登录成功，如非本人操作，请及时修改密码（密码修改成功，全平台清空登录态）</text>
                    <view class="card-row">
                        <text class="label">设备/平台</text>
                        <text class="value">Edge浏览器</text>
                    </view>
                    <view class="card-row">
                        <text class="label">登录方式</text>
                        <text class="value">第三方登录</text>
                    </view>
                    <view class="card-row">
                        <text class="label">参考登录地</text>
                        <text class="value">中国海南三亚</text>
                    </view>
                    <view class="card-row">
                        <text class="label">登录时间</text>
                        <text class="value">2026-03-25 13:30:51</text>
                    </view>
                    <view class="card-footer">
                        <text>查看详情</text>
                        <text class="arrow">›</text>
                    </view>
                </view>
            </block>

            <!-- ================= ID 7: 陌生人消息特殊布局 ================= -->
            <block v-else-if="chatInfo.id === 7">
                <view class="stranger-layout">
                    <view class="stranger-item">
                        <image class="stranger-avatar" src="/static/images/stranger.jpg" mode="aspectFill" />
                        <view class="stranger-info">
                            <view class="stranger-header">
                                <text class="stranger-name">万能小羊_</text>
                                <text class="stranger-time">2024年8月11日</text>
                            </view>
                            <text class="stranger-desc">感谢你的关注！分享的所有zi源均来自...</text>
                        </view>
                    </view>
                    <view class="divider-line"></view>
                    <text class="bottom-line-text">我可是有底线的_(:3 ∠)_</text>
                </view>
            </block>

            <!-- ================= 其他 ID: 标准气泡消息 ================= -->
            <block v-else>
                <block v-for="(msg, idx) in messages" :key="idx">
                    <!-- 时间分割线 -->
                    <view v-if="msg.time" class="time-divider">{{ msg.time }}</view>

                    <!-- 隐私限制提示 -->
                    <view v-if="msg.isPrivacyTip" class="privacy-tip">
                        对方主动回复或关注你前，最多发送1条消息
                    </view>

                    <!-- 普通消息行 -->
                    <view v-if="!msg.isPrivacyTip" class="msg-row" :class="msg.type">
                        <!-- 对方头像 -->
                        <image v-if="msg.type === 'other'" class="avatar" :src="chatInfo.avatar" mode="aspectFill" />

                        <!-- 气泡内容 -->
                        <view class="bubble-wrapper">
                            <view class="bubble">
                                <!-- 特殊处理 ID 8 的富文本链接 -->
                                <rich-text v-if="msg.isRich" :nodes="msg.content"></rich-text>
                                <text v-else class="bubble-text">{{ msg.content }}</text>
                            </view>
                            <!-- 自动回复标识 -->
                            <text v-if="msg.isAutoReply" class="auto-reply-tag">此条消息为自动回复</text>
                        </view>
                    </view>
                </block>
            </block>

            <!-- 正在输入指示器 -->
            <view v-if="isTyping" class="typing-indicator">
                <view class="dot"></view>
                <view class="dot"></view>
                <view class="dot"></view>
            </view>

            <!-- 底部锚点 -->
            <view id="chat-bottom-anchor" style="height: 1px;"></view>
        </scroll-view>

        <!-- 底部输入栏（系统通知和部分场景不显示） -->
        <view v-if="chatInfo.id !== 7 && chatInfo.id !== 2" class="input-bar">
            <view class="icon-btn left">
                <svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
                    <path
                        d="M512.153548 1024C229.830338 1024 0.143311 794.312974 0.143311 511.989763S229.830338 0 512.153548 0s512.010237 229.687026 512.010236 512.010237-229.687026 511.989763-512.010236 511.989763z m0-970.114961C259.557191 53.885039 54.048823 259.393406 54.048823 511.989763S259.557191 970.094488 512.153548 970.094488s458.104724-205.508367 458.104724-458.104725S764.749905 53.885039 512.153548 53.885039z"
                        fill="currentColor" />
                    <path
                        d="M364.194894 347.52984h-14.61775c-25.591299 0-47.538397 20.165944-47.538397 42.174461v42.17446c0 23.830618 21.926625 42.174461 47.538397 42.174461h14.61775c25.591299 0 47.538397-20.165944 47.538397-42.174461v-42.17446c0-22.008517-21.926625-42.174461-47.538397-42.174461m310.780734 0h-14.61775c-25.591299 0-47.538397 20.165944-47.538397 42.174461v42.17446c0 23.830618 21.926625 42.174461 47.538397 42.174461h14.61775c25.591299 0 47.538397-20.165944 47.538397-42.174461v-42.17446c0-22.008517-21.947098-42.174461-47.538397-42.174461m-162.69924 2427.210908c-93.23422 0-177.316992-56.833157-213.88184-143.024651-7.308875-18.343843 0-40.331887 20.104524-47.681709 18.282424-7.329348 40.209049 0 47.538397 20.165944 25.591299 58.67573 82.260671 97.185517 146.238919 97.185517 63.978247 0 122.490193-38.509787 146.238919-97.185517 7.308875-18.343843 29.255973-27.495292 47.538397-20.165944s27.413399 29.337865 20.104524 47.681709c-36.564848 86.191495-120.64762 143.024652-213.88184 143.024651"
                        fill="currentColor" />
                </svg>
            </view>
            <view class="input-box">
                <input class="input-field" v-model="inputText" placeholder="发个消息聊聊呗" confirm-type="send"
                    @confirm="sendMessage" />
            </view>
            <view class="icon-btn right" @click="toggleEmoji">
                <svg class="icon-svg" viewBox="0 0 1024 1024" width="200" height="200">
                    <path
                        d="M717.2 870c-10.1 0-20.2-3-29-8.8L493 731.6c-11.7-7.8-14.9-23.6-7.1-35.2 7.8-11.8 23.5-14.8 35.2-7.1L716.3 819l86-677.3L209 479.1l184.1 125.1c11.7 7.8 14.9 23.6 7.1 35.2-7.8 11.8-23.5 14.8-35.2 7.1L180.8 524.2c-15.2-10.1-24-27-23.4-45.4 0.6-18.3 10.4-34.7 26.3-43.8L774.6 95.8c17.4-9.9 38.5-9.2 55.2 1.8 16.6 11.1 25.5 30.3 23 50.1L769.3 824c-2.3 18.2-13.5 33.6-30.2 41.3-7 3.1-14.5 4.7-21.9 4.7z"
                        fill="currentColor" />
                    <path
                        d="M381.6 870c-14 0-25.4-11.4-25.4-25.4V727.3l433-607.3c8.2-11.4 24.1-14 35.4-6 11.4 8.1 14.1 24 6 35.4L407 743.6v101c0 14-11.4 25.4-25.4 25.4z"
                        fill="currentColor" />
                </svg>
            </view>
        </view>

        <Toast :visible="toastVisible" :message="toastMsg" />
    </view>
</template>

<script>
import Toast from '@/components/Toast.vue'

export default {
    components: { Toast },
    data() {
        return {
            chatId: '',
            chatInfo: {},
            messages: [],
            notices: [
                {
                    id: 1,
                    title: '《哔哩哔哩隐私政策》修订通知',
                    content: '亲爱的用户，根据业务开展实际情况，哔哩哔哩于近期更新了《哔哩哔哩隐私政策》中的相关内容，您可以前往哔哩哔哩客户端【我的-设置-隐私政策-哔哩哔哩隐私政策全文】查看主要更新提示以及更新后的全部内容。',
                    time: '3个月前',
                    link: '',
                    linkText: ''
                },
                {
                    id: 2,
                    title: '哔哩哔哩协议修订通知',
                    content: '亲爱的用户，哔哩哔哩近期更新了《哔哩哔哩弹幕网用户使用协议》《哔哩哔哩隐私政策》《哔哩哔哩亲子平台服务协议》中的相关内容。\n您可以前往【哔哩哔哩客户端-我的-设置-用户协议/隐私政策】【哔哩哔哩客户端-我的-未成年人守护-亲子平台】或【哔哩哔哩网页端-协议汇总】查看更新后协议的全部内容。',
                    time: '2025-05-07',
                    link: '',
                    linkText: ''
                },
                {
                    id: 3,
                    title: '你的B站2024年度报告来啦！',
                    content: '珍贵的回忆已经为你打包完毕！今年你登录B站多少天？谁成为了你的最爱？戳链接回顾2024，查收UP主送来的惊喜彩蛋 >>',
                    time: '2024-12-20',
                    link: 'https://www.bilibili.com',
                    linkText: '网页链接'
                },
                {
                    id: 4,
                    title: '正在直播S14总决赛：BLG vs T1！',
                    content: 'BLG战队首度闯入S赛总决赛舞台，时隔11年，全华班再次与T1在决赛舞台相遇，谁将加冕为王？11月2日 22点，上6号直播间看S14总决赛，天选好礼送不停，高能观赛团更是惊喜不断，25万现金红包等你来抽！主直播间 >>',
                    time: '2024-11-02',
                    link: 'https://live.bilibili.com',
                    linkText: '网页链接'
                }
            ],
            inputText: '',
            isTyping: false,
            scrollIntoView: '',
            toastVisible: false,
            toastMsg: '',

            autoReplies: {
                3: `LR软件安装包
                下载链接：
                https://pan.xunlei.com/s/VO58Ixo2-0gcywyobHgIhhDOA1
                提取码：w53i

                剪映软件安装包
                下载链接：
                https://pan.xunlei.com/s/VOGZzJuR-zFxC5b3A8YAhdeGA1
                提取码：whep

                PR软件安装包
                下载链接：
                https://pan.xunlei.com/s/VONcl-jBE6pZAANBN_OYCAaVA1#
                提取码：3cpx

                AE软件安装包
                下载链接：
                https://pan.xunlei.com/s/VO58IymuD8fePTodZ5hJAleA1
                提取码：6knr

                PS软件安装包
                下载链接：
                https://pan.quark.cn/s/2c1b24ae0998
                提取码：ffjr`,
                4: '感谢您的关注',
                5: `【粉丝福利：领优惠券】
                1天搞定6000单词！考纲全覆盖！
                方法大于努力，课程差距大！
                错学十年，不如一天。
                跟随光速，最后一次背单词！

                选拔性考试：
                中高考、专升本、考研
                能力性考试：
                四六级、专四八、出国
                都需要单词量作为基础。
                单词量=数量+质量
                ——————————
                报名在公众号：爱学习的单词君
                wx：gsxdc001`,
                6: `因B站上无法直接发下载链接，所以需要获取安装包链接的，
                请关注微信公众号：
                PSCC12345
                关注后回复：PS下载，就可以获得安装包下载链接了。`,
                8: '', // 富文本单独处理，下面会说明
                9: '哈喽，正在备考25考研的同学...\n此条消息为自动回复'
            }
        }
    },
    onLoad(options) {
        if (!options.id) {
            uni.switchTab({ url: '/pages/index/index' });
            return;
        }
        this.chatId = options.id;
        this.loadChatData();
    },
    methods: {
        loadChatData() {
            // 所有联系人数据（包含 id: 2 系统通知）
            const allData = [
                { id: 1, title: '哔哩哔哩智能机', avatar: '/static/images/bilibili.jpg', isLive: false },
                { id: 2, title: '系统通知', avatar: '/static/images/system.jpg', isLive: false },
                { id: 3, title: 'pr免费版怎么下载', avatar: '/static/images/pr.jpg', isLive: false },
                { id: 4, title: '框框老师课堂', avatar: '/static/images/kk.jpg', isLive: false },
                { id: 5, title: 'TED官方英语', avatar: '/static/images/ted.jpg', isLive: true },
                { id: 6, title: 'cad教程零基础入门', avatar: '/static/images/cad.jpg', isLive: false },
                { id: 7, title: '陌生人消息', avatar: '/static/images/stranger.jpg', isLive: false },
                { id: 8, title: '黑马程序员', avatar: '/static/images/heima.jpg', isLive: false },
                { id: 9, title: '宋浩老师官方', avatar: '/static/images/songhao.jpg', isLive: true }
            ]
            this.chatInfo = allData.find(d => d.id == this.chatId) || allData[0]
            this.messages = []

            // ID 2: 系统通知（已在模板中单独渲染，无需消息数据）
            if (this.chatInfo.id === 2) {
                // 不需要填充 messages，模板会直接渲染 notices
                return;
            }

            // ID 3: PR软件
            if (this.chatInfo.id === 3) {
                this.messages.push({
                    type: 'other', time: '2025年10月21日 10:17',
                    content: 'LR软件安装包\n下载链接：\nhttps://pan.xunlei.com/s/VO58Ixo2-0gcywyobHgIhhDOA1\n提取码：w53i\n\n剪映软件安装包\n下载链接：\nhttps://pan.xunlei.com/s/VOGZzJuR-zFxC5b3A8YAhdeGA1\n提取码：whep\n\nPR软件安装包\n下载链接：\nhttps://pan.xunlei.com/s/VONcl-jBE6pZAANBN_OYCAaVA1#\n提取码：3cpx\n\nAE软件安装包\n下载链接：\nhttps://pan.xunlei.com/s/VO58IymuD8fePTodZ5hJAleA1\n提取码：6knr\n\nPS软件安装包\n下载链接：\nhttps://pan.quark.cn/s/2c1b24ae0998\n提取码：ffjr',
                    isAutoReply: true
                })
            }
            // ID 4: 框框老师
            else if (this.chatInfo.id === 4) {
                this.messages.push({ type: 'other', time: '2024年11月7日 20:24', content: '感谢您的关注', isAutoReply: true })
                this.messages.push({ type: 'system', time: '2026年3月2日 09:31', isPrivacyTip: true })
            }
            // ID 5: TED
            else if (this.chatInfo.id === 5) {
                this.messages.push({
                    type: 'other', time: '2024年9月13日 08:33',
                    content: '【粉丝福利：领优惠券】\n1天搞定6000单词！考纲全覆盖！\n方法大于努力，课程差距大！\n错学十年，不如一天。\n跟随光速，最后一次背单词！\n\n选拔性考试：\n中高考、专升本、考研\n能力性考试：\n四六级、专四八、出国\n都需要单词量作为基础。\n单词量=数量+质量\n——————————\n报名在公众号：爱学习的单词君\nwx：gsxdc001',
                    isAutoReply: true
                })
                this.messages.push({ type: 'system', time: '2024年9月22日 20:04', isPrivacyTip: true })
            }
            // ID 6: CAD教程
            else if (this.chatInfo.id === 6) {
                this.messages.push({
                    type: 'other', time: '2024年9月6日 20:33',
                    content: '因B站上无法直接发下载链接，所以需要获取安装包链接的，\n请关注微信公众号：\nPSCC12345\n关注后回复：PS下载，就可以获得安装包下载链接了。',
                    isAutoReply: true
                })
                this.messages.push({ type: 'system', time: '2024年9月6日 20:34', isPrivacyTip: true })
            }
            // ID 8: 黑马程序员
            else if (this.chatInfo.id === 8) {
                const richHtml = `
					<div style="font-family: 'STKaiti', serif; font-size: 15px; line-height: 1.6; color: #18191c;">
						感谢关注黑马程序员<br>
						黑马程序员为B站科技计算机技术区第一UP主，粉丝数357万+，是国内公认的好口碑IT教育培训机构，累计培养IT人才30万+，其中每入学10名学员有7名来自老学员推荐。<br><br>
						输入对应数字编号，即可获取相应资源：<br>
						1.如何快速下载配套资料<br>
						2.获取2024年最新自学路线图<br>
						3.快速了解黑马程序员<br>
						4.领取B站专属粉丝福利<br>
						========================<br>
						其余问题可以直接私信<br>
						或者 <span style="color: #00a1d6; text-decoration: underline;">网页链接</span>
					</div>
				`;
                this.messages.push({
                    type: 'other', time: '2024年8月1日 13:24',
                    content: richHtml, isRich: true, isAutoReply: true
                })
                this.messages.push({ type: 'system', time: '2024年8月11日 17:31', isPrivacyTip: true })
            }
            // ID 9: 宋浩老师
            else if (this.chatInfo.id === 9) {
                this.messages.push({ type: 'other', time: '2023年11月10日', content: '哈喽，正在备考25考研的同学...\n此条消息为自动回复', isAutoReply: true })
            }

            this.scrollToBottom()
        },
        toggleEmoji() { this.showToast('表情面板开发中...') },
        sendMessage() {
            const text = this.inputText.trim()
            if (!text) return
            const now = new Date()
            const timeStr = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`
            this.messages.push({ type: 'self', content: text, time: timeStr })
            this.inputText = ''
            this.scrollToBottom()

            // 自动回复（排除系统通知和陌生人）
            if (this.chatInfo.id !== 7 && this.chatInfo.id !== 2) {
                // 判断是否为 ID 8
                if (this.chatInfo.id === 8) {
                    const richHtml = `
        <div style="font-family: 'STKaiti', serif; font-size: 15px; line-height: 1.6; color: #18191c;">
          感谢关注黑马程序员<br>
          黑马程序员为B站科技计算机技术区第一UP主，粉丝数357万+，是国内公认的好口碑IT教育培训机构，累计培养IT人才30万+，其中每入学10名学员有7名来自老学员推荐。<br><br>
          输入对应数字编号，即可获取相应资源：<br>
          1.如何快速下载配套资料<br>
          2.获取2024年最新自学路线图<br>
          3.快速了解黑马程序员<br>
          4.领取B站专属粉丝福利<br>
          ========================<br>
          其余问题可以直接私信<br>
          或者 <span style="color: #00a1d6; text-decoration: underline;">网页链接</span>
        </div>
      `
                    this.isTyping = true
                    setTimeout(() => {
                        this.isTyping = false
                        this.messages.push({
                            type: 'other',
                            content: richHtml,
                            isRich: true,
                            isAutoReply: true,
                            time: '刚刚'
                        })
                        this.scrollToBottom()
                    }, 1000)
                } else {
                    const replyContent = this.autoReplies[this.chatInfo.id]
                    if (replyContent) {
                        this.isTyping = true
                        setTimeout(() => {
                            this.isTyping = false
                            this.messages.push({
                                type: 'other',
                                content: replyContent,
                                time: '刚刚',
                                isAutoReply: true
                            })
                            this.scrollToBottom()
                        }, 1000)
                    }
                }
            }
        },
        scrollToBottom() {
            this.$nextTick(() => {
                this.scrollIntoView = 'chat-bottom-anchor';
                setTimeout(() => {
                    this.scrollIntoView = '';
                }, 100);
            });
        },
        goBack() {
            const pages = getCurrentPages();
            if (pages.length > 1) {
                uni.navigateBack();
            } else {
                uni.reLaunch({ url: '/pages/index/index' });
            }
        },
        openSettings() { this.showToast('打开设置') },
        editMode() {
            uni.showToast({ title: '编辑功能开发中', icon: 'none' })
        },
        openNotice(item) {
            if (item.link) {
                uni.showToast({ title: '打开链接', icon: 'none' })
            } else {
                uni.showModal({
                    title: item.title,
                    content: item.content,
                    showCancel: false,
                    confirmText: '知道了'
                })
            }
        },
        showToast(msg) { this.toastMsg = msg; this.toastVisible = true; setTimeout(() => this.toastVisible = false, 2000) }
    }
}
</script>

<style scoped>
/* 全局手写体风格 */
page {
    font-family: 'STKaiti', 'KaiTi', 'Comic Sans MS', cursive, sans-serif;
}

.chat-page {
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;        /* 新增：防止内容溢出 */
  background-color: #f5f6f7;
}

/* 头部 - 修复高度问题，与其他页面统一 */
.detail-header {
  height: 100rpx;
  flex-shrink: 0; 
    /* 增加高度，与其他页面统一 */
    background: #fff;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 24rpx;
    border-bottom: 1rpx solid #e3e5e7;
    position: relative;
    z-index: 10;
}

.back-btn {
    width: 80rpx;
    display: flex;
    align-items: center;
    justify-content: flex-start;
}

.icon-back {
    font-size: 52rpx;
    /* 增大返回图标 */
    color: #18191c;
    font-weight: 300;
}

.header-title-wrapper {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
    gap: 10rpx;
}

.header-title {
    font-size: 36rpx;
    /* 增大标题字体 */
    font-weight: 600;
    color: #18191c;
}

.live-tag {
    display: flex;
    align-items: center;
    gap: 4rpx;
    border: 1rpx solid #fb7299;
    border-radius: 20rpx;
    padding: 4rpx 14rpx;
    background: rgba(251, 114, 153, 0.1);
}

.live-tag text {
    font-size: 20rpx;
    color: #fb7299;
}

.header-right {
    width: 80rpx;
    display: flex;
    align-items: center;
    justify-content: flex-end;
}

.header-right .icon-svg {
    width: 44rpx;
    /* 增大设置图标 */
    height: 44rpx;
    color: #18191c;
    display: block;
}

.live-icon-svg {
    width: 24rpx;
    height: 24rpx;
    color: #3D78FB;
    display: block;
}

/* 内容区 */
.chat-content {
  flex: 1;
  min-height: 0;           /* 关键：允许收缩 */
  padding: 20rpx 30rpx;    /* 移除 padding-bottom */
  box-sizing: border-box;
}

/* 时间分割线 */
.time-divider {
    text-align: center;
    font-size: 24rpx;
    color: #9499a0;
    margin: 30rpx 0;
    font-family: 'STKaiti', serif;
}

/* 隐私提示 */
.privacy-tip {
    text-align: center;
    font-size: 24rpx;
    color: #c0c4cc;
    margin: 20rpx 0;
    font-family: 'STKaiti', serif;
}

/* 顶部操作栏（ID 1 专用）- 优化样式 */
.notice-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #ffffff;
    border-radius: 16rpx;
    padding: 28rpx 32rpx;
    margin-bottom: 24rpx;
    box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.02);
}

.action-group-left,
.action-group-right {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8rpx;
}

.action-group-left {
    flex: 1;
}

.action-group-right {
    flex: 1;
}

.divider-vertical {
    width: 1rpx;
    height: 60rpx;
    background: #e3e5e7;
    margin: 0 20rpx;
}

.action-icon {
    width: 48rpx;
    height: 48rpx;
    color: #18191c;
}

.action-text {
    font-size: 30rpx;
    color: #18191c;
    font-weight: 500;
}

.action-subtext {
    font-size: 24rpx;
    color: #c0c4cc;
}

/* ===== 系统通知样式 (ID 2) ===== */
.action-bar {
    height: 88rpx;
    background: #fff;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 32rpx;
    border-bottom: 1rpx solid #e3e5e7;
    margin-bottom: 16rpx;
}

.action-left {
    font-size: 30rpx;
    color: #18191c;
    font-weight: 500;
}

.action-right {
    font-size: 30rpx;
    color: #fb7299;
    font-weight: 500;
}

.notice-list {
    background: #fff;
}

.notice-item {
    padding: 32rpx;
    border-bottom: 1rpx solid #f1f2f3;
    background: #fff;
}

.notice-content {
    margin-bottom: 16rpx;
}

.notice-title {
    font-size: 32rpx;
    font-weight: 600;
    color: #18191c;
    display: block;
    margin-bottom: 16rpx;
    line-height: 1.4;
}

.notice-body {
    font-size: 28rpx;
    color: #61666d;
    line-height: 1.6;
    display: block;
    white-space: pre-line;
}

.notice-link {
    margin-top: 12rpx;
}

.link-text {
    font-size: 28rpx;
    color: #00a1d6;
    text-decoration: underline;
}

.notice-time {
    font-size: 24rpx;
    color: #c0c4cc;
    text-align: right;
    display: block;
}

/* ===== 系统通知样式结束 ===== */

/* 通知卡片 (ID 1) */
.notice-card {
    background: #fff;
    border-radius: 16rpx;
    padding: 32rpx;
    margin-bottom: 24rpx;
    box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.02);
}

.card-title {
    font-size: 34rpx;
    /* 增大标题字体 */
    font-weight: bold;
    color: #18191c;
    display: block;
    margin-bottom: 24rpx;
}

.card-body {
    font-size: 28rpx;
    color: #61666d;
    line-height: 1.8;
    /* 增加行高 */
    display: block;
    margin-bottom: 32rpx;
}

.card-row {
    display: flex;
    margin-bottom: 20rpx;
    /* 增加间距 */
    font-size: 28rpx;
}

.card-row .label {
    width: 180rpx;
    color: #9499a0;
}

.card-row .value {
    flex: 1;
    color: #18191c;
}

.card-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 32rpx;
    padding-top: 24rpx;
    border-top: 1rpx solid #e3e5e7;
    font-size: 28rpx;
    color: #18191c;
}

.arrow {
    font-size: 44rpx;
    /* 增大箭头 */
    color: #c0c4cc;
}

/* 陌生人消息布局 (ID 7) */
.stranger-layout {
    padding: 20rpx 0;
}

.stranger-item {
    display: flex;
    gap: 20rpx;
    padding: 20rpx;
    background: #fff;
}

.stranger-avatar {
    width: 100rpx;
    height: 100rpx;
    border-radius: 50%;
}

.stranger-info {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.stranger-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10rpx;
}

.stranger-name {
    font-size: 32rpx;
    color: #18191c;
    font-weight: 500;
}

.stranger-time {
    font-size: 24rpx;
    color: #c0c4cc;
}

.stranger-desc {
    font-size: 26rpx;
    color: #9499a0;
}

.divider-line {
    height: 1rpx;
    background: #e3e5e7;
    margin: 40rpx 0;
}

.bottom-line-text {
    text-align: center;
    font-size: 28rpx;
    color: #c0c4cc;
    display: block;
}

/* 标准消息行 */
.msg-row {
    display: flex;
    margin-bottom: 30rpx;
    align-items: flex-start;
}

.msg-row.other {
    justify-content: flex-start;
}

.msg-row.self {
    justify-content: flex-end;
}

.avatar {
    width: 80rpx;
    height: 80rpx;
    border-radius: 50%;
    flex-shrink: 0;
    background: #eee;
}

.bubble-wrapper {
    max-width: 70%;
    display: flex;
    flex-direction: column;
}

.msg-row.other .bubble-wrapper {
    margin-left: 20rpx;
    align-items: flex-start;
}

.msg-row.self .bubble-wrapper {
    margin-right: 20rpx;
    align-items: flex-end;
}

.bubble {
    background: #fff;
    padding: 24rpx 30rpx;
    border-radius: 16rpx;
    font-size: 30rpx;
    color: #18191c;
    line-height: 1.6;
    word-break: break-all;
    box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.03);
}

.msg-row.self .bubble {
    background: #fb7299;
    color: #fff;
}

.auto-reply-tag {
    font-size: 22rpx;
    color: #c0c4cc;
    margin-top: 10rpx;
}

/* 底部输入栏 */
.input-bar {
  /* 移除 position: fixed, bottom, left, right, z-index */
  flex-shrink: 0;          /* 防止被压缩 */
  background: #fff;
  padding: 20rpx 30rpx;
  padding-bottom: calc(20rpx + env(safe-area-inset-bottom));
  display: flex;
  align-items: center;
  gap: 20rpx;
  border-top: 1rpx solid #e3e5e7;
}

.icon-btn {
    width: 60rpx;
    height: 60rpx;
    display: flex;
    align-items: center;
    justify-content: center;
}

.icon-btn .icon-svg {
    width: 48rpx;
    height: 48rpx;
    color: #18191c;
    display: block;
}

.input-box {
    flex: 1;
    background: #f1f2f3;
    border-radius: 40rpx;
    padding: 0 30rpx;
    height: 72rpx;
    display: flex;
    align-items: center;
}

.input-field {
    flex: 1;
    font-size: 28rpx;
    color: #18191c;
}

/* 动画 */
.typing-indicator {
    display: flex;
    gap: 8rpx;
    padding: 20rpx 0;
    margin-left: 100rpx;
}

.dot {
    width: 12rpx;
    height: 12rpx;
    border-radius: 50%;
    background: #c0c4cc;
    animation: bounce 1.4s infinite;
}

.dot:nth-child(2) {
    animation-delay: 0.2s;
}

.dot:nth-child(3) {
    animation-delay: 0.4s;
}

@keyframes bounce {

    0%,
    60%,
    100% {
        transform: translateY(0);
        opacity: 0.4;
    }

    30% {
        transform: translateY(-12rpx);
        opacity: 1;
    }
}

/* 全局变量 */
page {
    --bili-pink: #fb7299;
    --bili-bg: #f1f2f3;
    --bili-white: #ffffff;
    --bili-text: #18191c;
    --bili-text-secondary: #9499a0;
    --bili-text-light: #c0c4cc;
    --bili-border: #e3e5e7;
    font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "HarmonyOS Sans", sans-serif;
    background-color: var(--bili-bg);
    height: 100%;
}
/* 基础重置 */
view, text, image, scroll-view {
    box-sizing: border-box;
}
* {
    margin: 0;
    padding: 0;
}
</style>