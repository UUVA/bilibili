<template>
	<view class="app">
		<view class="login-page">
			<view class="login-wrapper">
				<view class="login-header">
					<view class="logo">
						<view class="icon-bili">
							<svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="white" stroke-width="2">
								<polygon points="5,3 19,12 5,21 5,3" />
							</svg>
						</view>
						<text>哔哩哔哩</text>
					</view>
					<view class="subtitle">(゜-゜)つロ 干杯~</view>
				</view>

				<view class="login-form">
					<view class="form-group">
						<text class="label">用户名</text>
						<view class="input-wrap">
							<input
								type="text"
								class="input-field"
								v-model="loginForm.username"
								placeholder="请输入用户名"
								maxlength="32"
								@input="errorFields.username = false"
							/>
						</view>
					</view>

					<view class="form-group">
						<text class="label">手机号 / 邮箱</text>
						<view class="input-wrap">
							<input
								type="text"
								class="input-field"
								v-model="loginForm.account"
								placeholder="请输入手机号或邮箱"
								maxlength="64"
								@input="errorFields.account = false"
							/>
						</view>
					</view>

					<view class="form-group">
						<text class="label">密码</text>
						<view class="input-wrap">
							<input
								:type="showPassword ? 'text' : 'password'"
								class="input-field"
								v-model="loginForm.password"
								placeholder="请输入密码"
								maxlength="32"
								@input="errorFields.password = false"
							/>
							<text class="toggle-pwd" @click="showPassword = !showPassword">
								{{ showPassword ? '隐藏' : '显示' }}
							</text>
						</view>
					</view>

					<view class="form-extra">
						<view class="remember" @click="loginForm.remember = !loginForm.remember">
							<view class="check-box" :class="{ checked: loginForm.remember }">
								<text v-if="loginForm.remember">✓</text>
							</view>
							<text>记住密码</text>
						</view>
						<text class="forgot" @click="showToast('正在前往密码找回页面...')">忘记密码？</text>
					</view>

					<button class="btn-login" :class="{ loading: loginLoading }" @click="handleLogin" :disabled="loginLoading">
						<text v-if="!loginLoading">登 录</text>
						<text v-else>登录中...</text>
					</button>
				</view>

				<view class="third-party">
					<view class="divider"><text>其他方式登录</text></view>
					<view class="social-icons">
						<view class="social-btn" @click="showToast('微信登录')">
							<svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor">
								<path d="M8.691 2.188C3.891 2.188 0 5.476 0 9.53c0 2.212 1.17 4.203 3.002 5.55a.59.59 0 0 1 .213.665l-.39 1.48c-.019.07-.048.141-.048.213 0 .163.13.295.29.295a.326.326 0 0 0 .167-.054l1.903-1.114a.864.864 0 0 1 .717-.098 10.16 10.16 0 0 0 2.837.403c.276 0 .543-.027.811-.05-.857-2.578.157-4.972 1.932-6.446 1.703-1.415 3.882-1.98 5.853-1.838-.576-3.583-4.196-6.348-8.596-6.348zM5.785 5.991c.642 0 1.162.529 1.162 1.18a1.17 1.17 0 0 1-1.162 1.178A1.17 1.17 0 0 1 4.623 7.17c0-.651.52-1.18 1.162-1.18zm5.813 0c.642 0 1.162.529 1.162 1.18a1.17 1.17 0 0 1-1.162 1.178 1.17 1.17 0 0 1-1.162-1.178c0-.651.52-1.18 1.162-1.18zm4.269 3.854c-3.623 0-6.56 2.64-6.56 5.895 0 3.256 2.937 5.896 6.56 5.896a7.68 7.68 0 0 0 2.273-.32.551.551 0 0 1 .46.063l1.436.84a.196.196 0 0 0 .1.033.183.183 0 0 0 .183-.183c0-.046-.02-.092-.035-.138l-.287-1.092a.473.473 0 0 1 .17-.53c1.47-1.065 2.423-2.682 2.423-4.419 0-3.255-2.937-5.895-6.56-5.895zm-2.592 3.895c.514 0 .932.425.932.945 0 .52-.418.945-.932.945a.94.94 0 0 1-.932-.945c0-.52.418-.945.932-.945zm5.184 0c.514 0 .932.425.932.945 0 .52-.418.945-.932.945a.94.94 0 0 1-.932-.945c0-.52.418-.945.932-.945z"/>
							</svg>
						</view>
						<view class="social-btn" @click="showToast('QQ登录')">
							<svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor">
								<path d="M12 2C6.477 2 2 6.477 2 12c0 4.991 3.657 9.128 8.438 9.879-.14-.937-.242-1.823-.242-2.254 0-1.022.212-1.701.461-2.43-.526-.13-1.453-.783-1.453-1.83 0-1.032.835-1.506 1.484-1.506.22 0 .481.056.742.131-.38-.545-.622-1.221-.622-1.944 0-1.513.848-2.635 1.922-2.635.18 0 .35.03.518.081-.083-.375-.123-.758-.123-1.144 0-1.005.837-2.037 1.976-2.037.538 0 1.02.216 1.348.559.343-.066.69-.102 1.037-.102.347 0 .694.036 1.037.102.329-.343.811-.559 1.348-.559 1.139 0 1.976 1.032 1.976 2.037 0 .386-.04.769-.123 1.144.168-.051.338-.081.518-.081 1.074 0 1.922 1.122 1.922 2.635 0 .723-.242 1.399-.622 1.944.261-.075.522-.131.742-.131.649 0 1.484.474 1.484 1.506 0 1.047-.927 1.7-1.453 1.83.249.729.461 1.408.461 2.43 0 .431-.102 1.317-.242 2.254C18.343 21.128 22 16.991 22 12c0-5.523-4.477-10-10-10z"/>
							</svg>
						</view>
						<view class="social-btn" @click="showToast('微博登录')">
							<svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor">
								<path d="M17.676 11.183c.262-.264.48-.262.643-.008.188.287.186.628-.054.943-.188.246-.469.376-.728.353-.284-.025-.507-.191-.543-.426-.023-.158.021-.43.682-.862zm.727-3.713c-1.404-1.701-3.534-2.694-5.888-2.693-.621 0-1.238.079-1.843.234-.353.09-.525.149-.525.149s.12-.082.318-.198c.78-.388 1.69-.604 2.62-.615 2.453-.028 4.785 1.005 6.331 2.784.271.312.522.641.75.986-.251-.076-.61-.189-.763-.647zm-2.266-1.619c-1.28-1.455-3.344-2.219-5.481-1.992-.883.094-1.727.358-2.472.778-.247.14-.44.214-.44.214s.247-.102.471-.201c1.461-.559 3.289-.622 4.855-.141 1.632.5 2.981 1.76 3.668 3.407.409-.327.872-.61 1.399-.795-.443-.908-1.198-1.72-2-2.27zm2.87 4.912c-.468-.811-1.346-1.491-2.559-1.684-1.238-.197-2.524.212-3.269 1.022-.767.832-.93 1.812-.825 2.645.08.64.324 1.142.696 1.543-.251-.105-.611-.237-1.001-.352-1.7-.504-4.206-.256-5.196 1.13-1.067 1.497-.501 3.724 1.277 4.603 1.7.842 3.95.646 5.473.019 1.932-.794 3.766-2.62 4.425-4.699.26-.826.213-1.732-.021-2.227zm-6.34 4.359c-1.441.877-3.39.776-4.353-.226-.963-1.002-.582-2.501.859-3.378 1.44-.877 3.39-.776 4.353.226.963 1.002.582 2.501-.859 3.378z"/>
							</svg>
						</view>
						<view class="social-btn" @click="showToast('Apple登录')">
							<svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor">
								<path d="M18.71 19.5c-.83 1.24-1.71 2.45-3.05 2.47-1.34.03-1.77-.79-3.29-.79-1.53 0-2 .77-3.27.82-1.31.05-2.3-1.32-3.14-2.53C4.25 17 2.94 12.45 4.7 9.39c.87-1.52 2.43-2.48 4.12-2.51 1.28-.02 2.5.87 3.29.87.78 0 2.26-1.07 3.8-.91.65.03 2.47.26 3.64 1.98-.09.06-2.17 1.28-2.15 3.81.03 3.02 2.65 4.03 2.68 4.04-.03.07-.42 1.44-1.38 2.83M13 3.5c.73-.83 1.94-1.46 2.94-1.5.13 1.17-.34 2.35-1.04 3.19-.69.85-1.83 1.51-2.95 1.42-.15-1.15.41-2.35 1.05-3.11z"/>
							</svg>
						</view>
					</view>
				</view>
				<view class="register-entry">
					<text>还没有账号？</text>
					<text class="register-link" @click="showToast('正在前往注册页面...')">立即注册</text>
				</view>
			</view>
		</view>

		<!-- Toast -->
		<view class="login-toast" :class="[toastType, { show: toastVisible }]">
			<text>{{ toastMessage }}</text>
		</view>
	</view>
</template>

<script>
const USER_DATABASE = {
	'test': { account: '111111', password: '123456' },
	'王润姿': { account: '2312505003', password: '123456' },
	'郭瑾涵': { account: '2312505005', password: '123456' },
	'李睿': { account: '2312505039', password: '123456' },
	'李一佳': { account: '2312505047', password: '123456' },
	'袁冶': { account: '2312505002', password: '123456' },
}

export default {
	data() {
		return {
			loginLoading: false,
			showPassword: false,
			loginForm: {
				username: '',
				account: '',
				password: '',
				remember: false,
			},
			errorFields: {
				username: false,
				account: false,
				password: false,
			},
			toastMessage: '',
			toastType: '',
			toastVisible: false,
			toastTimer: null,
		}
	},

	methods: {
		showToast(msg, type = '') {
			clearTimeout(this.toastTimer)
			this.toastMessage = msg
			this.toastType = type
			this.toastVisible = true
			this.toastTimer = setTimeout(() => {
				this.toastVisible = false
			}, 2200)
		},

		handleLogin() {
			const { username, account, password } = this.loginForm
			this.errorFields = { username: false, account: false, password: false }

			if (!username) {
				this.errorFields.username = true
				this.showToast('请填写用户名', 'error')
				return
			}
			if (!account) {
				this.errorFields.account = true
				this.showToast('请填写手机号或邮箱', 'error')
				return
			}
			if (!password) {
				this.errorFields.password = true
				this.showToast('请填写密码', 'error')
				return
			}

			const user = USER_DATABASE[username]
			if (!user) {
				this.errorFields.username = true
				this.showToast('用户名不存在，请检查后重试', 'error')
				return
			}
			if (user.account !== account) {
				this.errorFields.account = true
				this.showToast('手机号/邮箱与用户名不匹配', 'error')
				return
			}
			if (user.password !== password) {
				this.errorFields.password = true
				this.showToast('密码错误，请重新输入', 'error')
				return
			}

			this.loginLoading = true
			setTimeout(() => {
				this.loginLoading = false
				// 清除旧用户的本地状态，防止串号
				uni.removeStorageSync('bili_mine_state')
				// 保存登录状态
				uni.setStorageSync('bili_is_logged_in', true)
				uni.setStorageSync('bili_username', username)
				this.showToast('登录成功！ 🎉', 'success')
				// 跳转到首页
				setTimeout(() => {
					uni.reLaunch({ url: '/pages/index/index' })
				}, 600)
			}, 800)
		},
	},

	mounted() {
		// 每次启动都显示登录页
	},
}
</script>

<style scoped>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}

.app {
	width: 100%;
	height: 100vh;
	background: #f4f5f7;
	position: relative;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Helvetica Neue", Arial, sans-serif;
}

.login-page {
	flex: 1;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 20px;
	background: rgba(244, 245, 247, 0.96);
}

.login-wrapper {
	width: 100%;
	max-width: 380px;
	background: #fff;
	border-radius: 16px;
	box-shadow: 0 8px 40px rgba(0,0,0,0.08);
	padding: 32px 28px 28px;
}

.login-header {
	text-align: center;
	margin-bottom: 24px;
}

.login-header .logo {
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 8px;
	font-size: 24px;
	font-weight: 700;
	color: #fb7299;
}

.login-header .logo .icon-bili {
	width: 36px;
	height: 36px;
	background: linear-gradient(135deg, #fb7299, #ff5e85);
	border-radius: 10px;
	display: flex;
	align-items: center;
	justify-content: center;
}

.login-header .subtitle {
	margin-top: 4px;
	font-size: 13px;
	color: #9499a0;
}

.login-form {
	margin-top: 8px;
}

.form-group {
	margin-bottom: 14px;
}

.form-group .label {
	display: block;
	font-size: 13px;
	font-weight: 500;
	color: #18191c;
	margin-bottom: 4px;
}

.input-wrap {
	position: relative;
}

.input-field {
	width: 100%;
	height: 44px;
	padding: 0 14px;
	font-size: 14px;
	color: #18191c;
	background: #f4f5f7;
	border: 1px solid transparent;
	border-radius: 8px;
	outline: none;
	transition: all 0.2s;
}

.input-field:focus {
	background: #fff;
	border-color: #fb7299;
}

.input-field::placeholder {
	color: #c0c4cc;
}

.input-wrap .toggle-pwd {
	position: absolute;
	right: 12px;
	top: 50%;
	transform: translateY(-50%);
	color: #9499a0;
	cursor: pointer;
	font-size: 13px;
}

.form-extra {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 18px;
	font-size: 13px;
}

.form-extra .remember {
	display: flex;
	align-items: center;
	gap: 6px;
	color: #9499a0;
	cursor: pointer;
}

.form-extra .remember .check-box {
	width: 16px;
	height: 16px;
	border: 1.5px solid #c0c4cc;
	border-radius: 3px;
	display: flex;
	align-items: center;
	justify-content: center;
	transition: all 0.2s;
}

.form-extra .remember .check-box.checked {
	background: #fb7299;
	border-color: #fb7299;
	color: #fff;
	font-size: 11px;
	font-weight: 700;
}

.form-extra .forgot {
	color: #9499a0;
	cursor: pointer;
}

.btn-login {
	width: 100%;
	height: 46px;
	border: none;
	border-radius: 8px;
	background: linear-gradient(135deg, #fb7299, #ff5e85);
	color: #fff;
	font-size: 16px;
	font-weight: 600;
	cursor: pointer;
	transition: all 0.25s;
	box-shadow: 0 4px 14px rgba(251,114,153,0.3);
	display: flex;
	align-items: center;
	justify-content: center;
}

.btn-login:active {
	transform: scale(0.97);
}

.btn-login.loading {
	opacity: 0.75;
}

.third-party {
	margin-top: 18px;
	text-align: center;
}

.third-party .divider {
	display: flex;
	align-items: center;
	gap: 14px;
	margin-bottom: 14px;
}

.third-party .divider::before,
.third-party .divider::after {
	content: "";
	flex: 1;
	height: 1px;
	background: #e3e5e7;
}

.third-party .divider text {
	font-size: 12px;
	color: #9499a0;
	flex-shrink: 0;
}

.social-icons {
	display: flex;
	justify-content: center;
	gap: 16px;
}

.social-icons .social-btn {
	width: 40px;
	height: 40px;
	border-radius: 50%;
	border: 1px solid #e3e5e7;
	background: #fff;
	display: flex;
	align-items: center;
	justify-content: center;
	cursor: pointer;
	color: #666;
	transition: all 0.2s;
}

.social-icons .social-btn:active {
	border-color: #fb7299;
	color: #fb7299;
}

.register-entry {
	text-align: center;
	margin-top: 18px;
	font-size: 13px;
	color: #9499a0;
}

.register-entry .register-link {
	color: #fb7299;
	font-weight: 500;
	margin-left: 4px;
	cursor: pointer;
}

/* Toast */
.login-toast {
	position: fixed;
	top: 24px;
	left: 50%;
	transform: translateX(-50%) translateY(-120px);
	background: #18191c;
	color: #fff;
	padding: 10px 24px;
	border-radius: 20px;
	font-size: 14px;
	z-index: 9999;
	pointer-events: none;
	white-space: nowrap;
	transition: transform 0.35s cubic-bezier(0.16,1,0.3,1);
	box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}

.login-toast.show {
	transform: translateX(-50%) translateY(0);
}

.login-toast.success {
	background: #52c41a;
}

.login-toast.error {
	background: #ff4d4f;
}
</style>
