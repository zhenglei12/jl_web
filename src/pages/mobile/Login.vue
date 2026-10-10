<template>
  <div class="mobile-login">
    <div class="mobile-login__brand">
      <span class="mobile-login__logo"><a-icon type="file-text" /></span>
      <div>
        <strong>渐鹿管理系统</strong>
        <small>DeerVison</small>
      </div>
    </div>
    <div class="mobile-login__card">
      <h1>欢迎登录</h1>
      <p>登录后上传订单稿件</p>
      <a-form-model ref="form" :model="form" :rules="rules" @submit.native.prevent="login">
        <a-form-model-item prop="username">
          <a-input v-model="form.username" size="large" placeholder="用户名" allow-clear>
            <a-icon slot="prefix" type="user" />
          </a-input>
        </a-form-model-item>
        <a-form-model-item prop="password">
          <a-input v-model="form.password" size="large" type="password" placeholder="密码" allow-clear>
            <a-icon slot="prefix" type="lock" />
          </a-input>
        </a-form-model-item>
        <a-button block size="large" type="primary" html-type="submit" :loading="loading">登录并继续</a-button>
      </a-form-model>
    </div>
  </div>
</template>

<script>
const rules = {
  username: [{ required: true, message: "请输入用户名" }],
  password: [{ required: true, message: "请输入密码" }],
};

export default {
  data() {
    return { rules, loading: false, form: { username: "", password: "" } };
  },
  methods: {
    login() {
      this.loading = true;
      this.$refs.form.validate().then(() => this.$auth.login({ ...this.form })).then(() => {
        const redirect = this.$route.query.redirect || "/mobile/order-add";
        this.$router.replace(redirect);
      }).finally(() => (this.loading = false));
    },
  },
};
</script>

<style lang="less" scoped>
.mobile-login {
  display: flex;
  min-height: 100vh;
  box-sizing: border-box;
  flex-direction: column;
  justify-content: center;
  padding: 28px 20px;
  background: linear-gradient(145deg, #eef6ff, #f8fbff 52%, #e8f2ff);

  &__brand { display: flex; align-items: center; justify-content: center; gap: 10px; margin-bottom: 28px; color: #163d74; }
  &__brand strong { display: block; font-size: 20px; }
  &__brand small { display: block; margin-top: 3px; color: #7c91ad; letter-spacing: 1.5px; }
  &__logo { display: flex; width: 44px; height: 44px; align-items: center; justify-content: center; border-radius: 13px; color: #fff; background: #2f7cf6; font-size: 22px; box-shadow: 0 8px 18px rgba(47, 124, 246, .25); }
  &__card { padding: 25px 20px 20px; border: 1px solid #dce9f8; border-radius: 16px; background: rgba(255,255,255,.94); box-shadow: 0 15px 40px rgba(37, 83, 139, .12); }
  h1 { margin: 0 0 6px; color: #183b6b; font-size: 25px; }
  p { margin: 0 0 24px; color: #91a0b3; }
}
</style>
