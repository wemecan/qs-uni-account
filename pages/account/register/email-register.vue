<template>
	<view class="account-page">
		<view class="page-header">
			<view class="page-title">{{ title }}</view>
		</view>
		<view class="page-body">
			<view class="form-box">
				<u-form :model="model" ref="uForm" label-position="top">
					<u-form-item class="form-item" label="邮箱" prop="email">
						<u-input v-model="model.email" placeholder="您的常用邮箱" type="text" />
					</u-form-item>
					<u-form-item class="form-item" label="邮件验证码" prop="verifyCode">
						<u-input v-model="model.verifyCode" placeholder="请输入验证码" type="number" />
						<u-button slot="right" type="success" size="mini" @click="getCode">{{ codeTips }}</u-button>
					</u-form-item>
					<u-form-item class="form-item" label="密码" prop="password">
						<u-input v-model="model.password" placeholder="6-20位的数字和字母" type="password" />
					</u-form-item>
					<u-form-item class="form-item" label="确认密码" prop="confirmPassword">
						<u-input v-model="model.confirmPassword" placeholder="请再次输入密码" type="password" />
					</u-form-item>
				</u-form>

				<u-verification-code seconds="60" ref="uCode" @change="codeChange"></u-verification-code>

				<u-gap height="40"></u-gap>

				<u-button :disabled="form.button.loading" type="primary" @click="submit">立即提交</u-button>

				<u-gap height="40"></u-gap>

				<view class="u-flex u-row-between">
					<view @click="openPage('register/mobile-register')">手机号注册</view>
					<view @click="openPage('login/login')">登录</view>
				</view>
			</view>

			<u-gap height="60"></u-gap>
		</view>
	</view>
</template>

<script>
var api = require('@/common/js/account.api.js');

export default {
	data() {
		return {
			title: '邮箱注册',
			desc: '',
			codeTips: '',
			form: {
				button: {
					loading: false
				}
			},
			model: {
				/* 邮箱 */
				email: '',
				/* 验证码 */
				verifyCode: '',
				/* 密码 */
				password: '',
				/* 确认密码 */
				confirmPassword: ''
			},
			rules: {
				email: [
					{
						required: true,
						message: '请输入邮箱',
						trigger: ['change', 'blur']
					},
					{
						validator: (rule, value, callback) => {
							return this.$u.test.email(value);
						},
						message: '邮箱格式不正确',
						trigger: ['change', 'blur']
					}
				],
				verifyCode: [
					{
						required: true,
						message: '请输入验证码',
						trigger: ['change', 'blur']
					},
					{
						type: 'number',
						message: '验证码只能为数字',
						trigger: ['change', 'blur']
					},
					{
						len: 6,
						message: '验证码长度为6位数字',
						trigger: ['change', 'blur']
					}
				],
				password: [
					{
						required: true,
						message: '请输入密码',
						trigger: ['change', 'blur']
					},
					{
						min: 6,
						max: 20,
						message: '密码长度在6到20个字符',
						trigger: ['change', 'blur']
					},
					{
						pattern: /^[0-9a-zA-Z]*$/g,
						transform(value) {
							return String(value);
						},
						message: '密码为字母或数字',
						trigger: ['change', 'blur']
					}
				],
				confirmPassword: [
					{
						required: true,
						message: '请再次输入密码',
						trigger: ['change', 'blur']
					},
					{
						validator: (rule, value, callback) => {
							return value === this.model.password;
						},
						message: '两次输入的密码不相等',
						trigger: ['change', 'blur']
					}
				]
			}
		};
	},
	methods: {
		openPage(path) {
			this.$u.route({
				url: '/pages/account/' + path
			});
		},
		submit() {
			this.$refs.uForm.validate(valid => {
				if (valid) {
					this.form.button.loading = true;
					api.registerByEmail(this.model)
						.then(res => {
							this.form.button.loading = false;
							console.log(res);
							if (res.code == 1) {
								this.$u.vuex('vuex_user.logined', true);
								this.$u.vuex('vuex_user.id', res.data.uid);
								this.$u.vuex('vuex_token', res.data.token);
								uni.navigateBack();
								return this.$u.toast('注册成功');
							} else {
								return this.$u.toast(res.msg);
							}
						})
						.catch(err => {
							this.form.button.loading = false;
							console.log(err);
							return this.$u.toast('出错，请稍后再试');
						});
				} else {
					console.log('验证失败');
				}
			});
		},
		codeChange(text) {
			this.codeTips = text;
		},
		// 获取验证码
		getCode() {
			if (this.$refs.uCode.canGetCode) {
				// 模拟向后端请求验证码
				uni.showLoading({
					title: '正在获取验证码',
					mask: true
				});
				setTimeout(() => {
					uni.hideLoading();
					// 这里此提示会被this.start()方法中的提示覆盖
					this.$u.toast('验证码已发送');
					// 通知验证码组件内部开始倒计时
					this.$refs.uCode.start();
				}, 2000);
			} else {
				this.$u.toast('倒计时结束后再发送');
			}
		}
	},
	onReady() {
		this.$refs.uForm.setRules(this.rules);
	}
};
</script>

<style lang="scss" scoped>
@import '../scss/account.scss';
</style>
