<template>
	<view class="account-page">
		<view class="page-header">
			<view class="page-title">{{ title }}</view>
		</view>
		<view class="page-body">
			<view class="form-box">
				<u-form :model="model" ref="uForm" label-position="top">
					<u-form-item class="form-item" label="手机号" prop="mobile" :label-position="form.labelPosition">
						<u-input v-model="model.mobile" placeholder="已验证的手机号" type="text" />
					</u-form-item>
					<u-form-item class="form-item" label="短信验证码" prop="verifyCode" :label-position="form.labelPosition">
						<u-input v-model="model.verifyCode" placeholder="请输入验证码" type="number" />
						<u-button slot="right" type="success" size="mini" @click="getCode">{{ codeTips }}</u-button>
					</u-form-item>
					<u-form-item class="form-item" label="新密码" prop="newPassword" :label-position="form.labelPosition">
						<u-input v-model="model.newPassword" placeholder="新密码(6-20位的数字和字母)" type="password" />
					</u-form-item>
					<u-form-item class="form-item" label="确认密码" prop="confirmPassword" :label-position="form.labelPosition">
						<u-input v-model="model.confirmPassword" placeholder="请再次输入新密码" type="password" />
					</u-form-item>
				</u-form>

				<u-verification-code seconds="60" ref="uCode" @change="codeChange"></u-verification-code>

				<u-gap height="40"></u-gap>

				<u-button :disabled="form.button.loading" type="primary" @click="submit">立即提交</u-button>

				<u-gap height="40"></u-gap>

				<view class="u-flex u-row-between">
					<view @click="openPage('findPassword/email-findPassword')">邮箱找回密码</view>
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
			title: '通过手机号找回密码',
			desc: '',
			form: {
				button: {
					loading: false
				}
			},
			model: {
				/* 手机号 */
				mobile: '',
				/* 验证码 */
				verifyCode: '',
				/* 新密码 */
				newPassword: '',
				/* 确认密码 */
				confirmPassword: ''
			},
			rules: {
				mobile: [
					{
						required: true,
						message: '请输入手机号',
						trigger: ['change', 'blur']
					},
					{
						validator: (rule, value, callback) => {
							return this.$u.test.mobile(value);
						},
						message: '手机号格式不正确',
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
				newPassword: [
					{
						required: true,
						message: '请设置新密码',
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
						message: '请再次输入新密码',
						trigger: ['change', 'blur']
					},
					{
						validator: (rule, value, callback) => {
							return value === this.model.newPassword;
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
					return this.$u.toast('验证通过');
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
