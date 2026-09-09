<template>
  <div class="wraper">
    <div class="login-intro">
      <div class="brand">
        <span class="brand-mark"><a-icon type="file-text" /></span>
        <span>
          <strong>渐鹿管理系统</strong>
          <small>JIANLU MANAGEMENT</small>
        </span>
      </div>
      <div class="intro-copy">
        <span class="eyebrow">SMART · SIMPLE · EFFICIENT</span>
        <h2>让每一笔订单<br />都清晰可见</h2>
        <p>订单、稿件、人员与数据统计集中管理，让团队协作更简单。</p>
      </div>
    </div>
    <div class="login-card" @keydown.enter="login">
      <a-form-model ref="form" class="login-form" :model="form" :rules="rules">
        <div class="login-heading">
          <span>WELCOME BACK</span>
          <h1 class="title">欢迎登录</h1>
          <p>请输入您的账号信息进入管理系统</p>
        </div>
        <a-form-model-item prop="username">
          <a-input v-model="form.username" placeholder="用户名" allow-clear>
            <a-icon slot="prefix" type="user" class="login-form-icon" />
          </a-input>
        </a-form-model-item>
        <a-form-model-item prop="password">
          <a-input type="password" v-model="form.password" placeholder="密码" allow-clear>
            <a-icon slot="prefix" type="lock" class="login-form-icon" />
          </a-input>
        </a-form-model-item>
        <a-button class="login-submit" size="large" type="primary" block :loading="loading" @click="login"
          >登录系统 <a-icon type="arrow-right" /></a-button
        >
      </a-form-model>
    </div>
    <div class="code">
      <a href="https://beian.miit.gov.cn/" target="_blank">鄂ICP备2024078917号</a>
    </div>
  </div>
</template>

<script>
const rules = {
  username: [
    {
      required: true,
      message: "用户名不能为空！",
      trigger: "blur",
    },
  ],
  password: [
    {
      required: true,
      message: "密码不能为空！",
      trigger: "blur",
    },
  ],
};

export default {
  data() {
    return {
      rules,
      loading: false,
      form: {
        username: "",
        password: "",
      },
    };
  },
  methods: {
    login() {
      this.loading = true;
      this.$refs.form
        .validate()
        .then(() => {
          return this.$auth.login({ ...this.form }).then(() => {
            this.$message.success("登录成功");
            this.$router.push("/");
          });
        })
        .finally(() => (this.loading = false));
    },
  },
};
</script>

<style lang="less" scoped>
.wraper {
  position: relative;
  display: flex;
  height: 100%;
  min-height: 620px;
  align-items: center;
  justify-content: space-between;
  padding: 64px 10vw;
  overflow: hidden;
  background-image: url("~@/assets/login.png");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;

  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(100deg, rgba(7, 36, 83, 0.48) 0%, rgba(12, 93, 184, 0.12) 52%, rgba(4, 29, 72, 0.22) 100%);
  }
}

.login-intro {
  position: relative;
  z-index: 1;
  display: flex;
  width: 46%;
  height: 100%;
  flex-direction: column;
  justify-content: space-between;
  padding: 12px 0 14vh;
  color: #fff;

  .brand {
    display: flex;
    align-items: center;
    gap: 12px;

    &-mark {
      display: flex;
      width: 46px;
      height: 46px;
      align-items: center;
      justify-content: center;
      border: 1px solid rgba(255, 255, 255, 0.38);
      border-radius: 13px;
      background: rgba(255, 255, 255, 0.17);
      box-shadow: 0 10px 25px rgba(9, 52, 112, 0.18);
      backdrop-filter: blur(8px);
      font-size: 23px;
    }

    > span:last-child {
      display: flex;
      flex-direction: column;
    }

    strong {
      font-size: 20px;
      letter-spacing: 1px;
    }

    small {
      margin-top: 4px;
      color: rgba(255, 255, 255, 0.65);
      font-size: 9px;
      letter-spacing: 2px;
    }
  }

  .intro-copy {
    max-width: 470px;

    .eyebrow {
      display: inline-block;
      margin-bottom: 18px;
      padding: 7px 12px;
      border: 1px solid rgba(255, 255, 255, 0.22);
      border-radius: 20px;
      color: rgba(255, 255, 255, 0.78);
      background: rgba(255, 255, 255, 0.08);
      font-size: 10px;
      letter-spacing: 1.8px;
    }

    h2 {
      margin: 0;
      color: #fff;
      font-size: 46px;
      font-weight: 700;
      line-height: 1.25;
      letter-spacing: 2px;
      text-shadow: 0 8px 25px rgba(8, 51, 110, 0.2);
    }

    p {
      max-width: 420px;
      margin: 18px 0 0;
      color: rgba(255, 255, 255, 0.76);
      font-size: 15px;
      line-height: 1.9;
    }
  }
}
.login {
  &-card {
    position: relative;
    z-index: 1;
    width: 420px;
    padding: 46px 42px 42px;
    border: 1px solid rgba(255, 255, 255, 0.72);
    border-radius: 18px;
    background: rgba(255, 255, 255, 0.94);
    box-shadow: 0 28px 80px rgba(9, 48, 103, 0.24);
    backdrop-filter: blur(18px);
  }

  &-form {
    width: 100%;

    .login-heading {
      margin-bottom: 30px;

      > span {
        color: #2f7cf6;
        font-size: 10px;
        font-weight: 700;
        letter-spacing: 2px;
      }

      p {
        margin: 8px 0 0;
        color: #91a0b5;
        font-size: 13px;
      }
    }

    .title {
      margin: 7px 0 0;
      color: #1e3558;
      font-size: 29px;
      font-weight: 700;
    }

    &-icon {
      color: #8799b4;
    }

    /deep/ .ant-form-item {
      margin-bottom: 20px;
    }

    /deep/ .ant-input-affix-wrapper .ant-input {
      height: 46px;
      padding-left: 38px;
      border-color: #dce7f3;
      border-radius: 9px;
      background: #f9fbfe;
    }

    /deep/ .ant-input-prefix {
      left: 14px;
    }
  }

  &-submit {
    height: 46px;
    margin-top: 8px;
    border-radius: 9px;
    font-size: 15px;

    .anticon {
      margin-left: 7px;
    }
  }
}

.code {
  position: absolute;
  left: 50%;
  z-index: 1;
  bottom: 22px;
  transform: translateX(-50%);

  a {
    color: rgba(255, 255, 255, 0.72);
    font-size: 12px;
  }
}

@media (max-width: 980px) {
  .wraper {
    justify-content: center;
    padding: 40px 24px;
  }

  .login-intro {
    display: none;
  }
}

@media (max-width: 520px) {
  .login-card {
    width: 100%;
    padding: 38px 26px 34px;
  }
}
</style>
