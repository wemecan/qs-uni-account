<template>
	<view class="account-page">
		<view class="page-header">
			<view class="page-title">{{ title }}</view>
		</view>
		<view class="page-body">
			<view class="form-box">
				<u-form :model="model" ref="uForm" label-position="top">
					<u-form-item class="form-item" label="旧密码" prop="oldPassword">
						<u-input v-model="model.oldPassword" placeholder="请输入旧密码" type="password" />
					</u-form-item>
					<u-form-item class="form-item" label="新密码" prop="newPassword">
						<u-input v-model="model.newPassword" placeholder="新密码(6-20位的数字和字母)" type="password" />
					</u-form-item>
					<u-form-item class="form-item" label="确认密码" prop="confirmPassword">
						<u-input v-model="model.confirmPassword" placeholder="请再次输入新密码" type="password" />
					</u-form-item>
				</u-form>

				<u-gap height="40"></u-gap>

				<u-button :disabled="form.button.loading" type="primary" @click="submit">确认修改</u-button>
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
			title: '修改密码',
			desc: '6至20位，由数字或字母组成',
			form: {
				button: {
					loading: false
				}
			},
			model: {
				/* 旧密码 */
				oldPassword: '',
				/* 新密码 */
				newPassword: '',
				/* 确认密码 */
				confirmPassword: ''
			},
			rules: {
				oldPassword: [
					{
						required: true,
						message: '请输入旧密码',
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
		submit() {
			this.$refs.uForm.validate(valid => {
				if (valid) {
					this.form.button.loading = true;
					let params = this.model;
					params.token = this.vuex_token;
					api.updatePassword(params)
						.then(res => {
							this.form.button.loading = false;
							console.log(res);
							if (res.code == 1) {
								return this.$u.toast('修改成功');
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
