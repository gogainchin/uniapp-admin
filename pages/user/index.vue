<template>
	<view :class="darkMode?'custom-dark':'custom-light'" class="w-h-100">
		<view class="cu-bar solid-bottom">
			<view class="action">
				<text class="cuIcon-title text-blue"></text>{{$t('BasicInfo')}}
			</view>
		</view>
		<view class="padding-lr padding-tb-xs">
			<view class="uni-flex uni-row">
				<view class="flex-item-30">{{$t('UserName')}}</view>
				<view class="flex-item-70">{{user?user.name:''}}</view>
			</view>
		</view>
		<view class="padding-lr padding-tb-xs">
			<view class="uni-flex uni-row">
				<view class="flex-item-30">{{$t('Name')}}</view>
				<view class="flex-item-70">{{user?user.userName:''}}</view>
			</view>
		</view>
		<view class="padding-lr padding-tb-xs">
			<view class="uni-flex uni-row">
				<view class="flex-item-30">{{$t('Role')}}</view>
				<view class="flex-item-70">{{user?user.roleStr:''}}</view>
			</view>
		</view>
		<view class="padding-lr padding-tb-xs">
			<button class="cu-btn block shadow bg-gradual-pink margin" @tap="tapSettings">{{$t('Settings')}}</button>
			<button class="cu-btn block shadow bg-gradual-blue margin" @tap="modifyPass">{{$t('ChangePassword')}}</button>
			<button class="cu-btn block shadow bg-gradual-orange margin" @tap="appUpgrade">{{$t('Upgrade')}}</button>
			<button class="cu-btn block shadow bg-white margin text-red" @tap="logout">{{$t('SignOut')}}</button>
		</view>
		<view v-if="appInfo.version" class="text-grey text-center">
			{{appInfo.name}} {{appInfo.version}}
		</view>
		<uni-popup ref="modifyPassPopup" type="bottom" :custom="true">
			<view class="uni-share uni-share-padding-bottom">
				<view>
					<form>
						<view class="cu-form-group">
							<view class="title">原密码</view>
							<input password placeholder="原密码" name="oldPassword" v-model="item.oldPassword"></input>
						</view>
						<view class="cu-form-group">
							<view class="title">新密码</view>
							<input password placeholder="新密码" name="newPassword" v-model="item.newPassword"></input>
						</view>
					</form>
					<view class="flex">
						<view class="flex-sub margin-sm">
							<button class="cu-btn block bg-blue margin" @tap="okClick">确认</button>
						</view>
					</view>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import { mapGetters } from 'vuex'
	import uniPopup from '@/components/uni-popup/uni-popup.vue'
	import md5 from '@/common/lib/md5.min.js'
	export default {
		components: {
			uniPopup
		},
		computed: {
			...mapGetters(['user', 'themeBgColor', 'darkMode'])
		},
		data() {
			return {
				appInfo: {
					version: '',
					name: ''
				},
				item: {
					oldPassword: '',
					newPassword: ''
				}
			}
		},
		onShow() {
			this.initTheme()
		},
		onReady() {
			uni.setNavigationBarTitle({
			    title: this.$t('Profile')
			})
			this.initTheme()
		},
		onLoad() {
			//#ifdef APP-PLUS
			plus.runtime.getProperty(plus.runtime.appid, (wgtinfo) => {
				this.appInfo.version = wgtinfo.version
				this.appInfo.name = wgtinfo.name
			})
			//#endif
		},
		methods: {
			initTheme() {
				this.setNavBarColor()
				this.setDarkMode()
			},
			setDarkMode() {
				this.darkMode ? uni.setTabBarStyle({
				  backgroundColor: '#2a2b2d'
				}) : uni.setTabBarStyle({
					backgroundColor: '#ffffff'
				})
			},
			setNavBarColor() {
				// navBar-bg-color
				uni.setNavigationBarColor({
				    frontColor: '#ffffff',
				    backgroundColor: this.themeBgColor,
				    animation: {
				        duration: 400,
				        timingFunc: 'easeIn'
				    }
				})
			},
			okClick() {
				const params = {
					id: this.user.id,
					password: md5(this.item.oldPassword),
					newpw: md5(this.item.newPassword)
				}
				this.$minApi.userPwdModify(params).then(res => {
					if (res.ok()) {
						this.$refs.modifyPassPopup.close()
						uni.showToast({
							title: '修改成功',
							icon: 'none'
						})
						setTimeout(() => {
							this.logout()
						}, 2000)
					} else {
						uni.showToast({
							title: '原密码不正确！',
							icon: 'none'
						})
					}
				})
			},
			modifyPass() {
				this.$refs.modifyPassPopup.open()
			},
			logout() {
				this.$store.dispatch('logout')
			},
			tapSettings() {
				uni.navigateTo({
					url: '/pages/user/setting'
				})
			},
			/**
			 * app整包更新检测
			 */
			appUpgrade() {
				//#ifndef APP-PLUS
				uni.showToast({
					title: '目前只支持Android版本软件更新',
					icon: 'none'
				})
				//#endif
				//#ifdef APP-PLUS
				uni.getSystemInfo({
					success: sysInfo => {
						let platform = sysInfo.platform
						plus.runtime.getProperty(plus.runtime.appid, (wgtinfo) => {
							// this.appInfo.version = wgtinfo.version
							// this.appInfo.name = wgtinfo.name
							let params = {
								appid: plus.runtime.appid,
								version: wgtinfo.versionCode,
								platform: platform
							}
							this.$minApi.findUpgradeApp(params).then(appRes => {
								if (appRes.appid) {
									uni.showModal({
										title: "下载更新提示",
										content: appRes.note,
										showCancel: false,
										confirmText: '确定',
										success: sucRes => {
											if (sucRes.confirm) {
												plus.runtime.openURL(appRes.url)
												// uni.downloadFile({
												//     url: appRes.url,
												//     success: res => {}
												// })
											}
										}
									})
								} else {
									uni.showToast({
										title: '已经是最新版本了。',
										icon: 'none'
									})
								}
							})
						})
					}
				})
				//#endif
			}
		}
	}
</script>
