<template>
  <div class="login-container">
    <el-form
      ref="loginForm"
      :model="loginForm"
      :rules="loginRules"
      class="login-form"
      auto-complete="on"
      label-position="left"
    >
      <div class="title-container">
        <h1 class="title">图书管理系统</h1>
      </div>

      <el-form-item prop="userName">
        <span class="svg-container"><svg-icon icon-class="user" /></span>
        <el-input
          ref="userName"
          v-model="loginForm.userName"
          placeholder="用户名"
          name="userName"
          type="text"
          tabindex="1"
          auto-complete="on"
        />
      </el-form-item>

      <el-form-item prop="passWord">
        <span class="svg-container"><svg-icon icon-class="password" /></span>
        <el-input
          :key="passwordType"
          ref="password"
          v-model="loginForm.passWord"
          :type="passwordType"
          placeholder="密码"
          name="password"
          tabindex="2"
          auto-complete="on"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="showPwd"
          ><svg-icon
            :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"
        /></span>
      </el-form-item>
      <el-button
        :loading="loading"
        type="primary"
        style="
          width: 100%;
          margin-bottom: 30px;
          font-size: 20px;
          font-weight: bold;
          letter-spacing: 1px;
          <!-- border: 1px solid red; -->
        "
        @click="handleLogin"
      >
        登录
      </el-button>
    </el-form>
    <div class="register">还没账号？注册一个</div>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    const validateUsername = (rule, value, callback) => {
      if (value == "") {
        callback(new Error("请填写用户名！"));
      } else {
        callback();
      }
    };
    const validatePassword = (rule, value, callback) => {
      if (value.length == 0) {
        callback(new Error("请填写密码！"));
      } else {
        callback();
      }
    };
    return {
      loginForm: {
        userName: "",
        passWord: "",
      },
      loginRules: {
        userName: [
          {
            required: true,
            trigger: "blur",
            validator: validateUsername,
          },
        ],
        passWord: [
          {
            required: true,
            trigger: "blur",
            validator: validatePassword,
          },
        ],
      },
      loading: false,
      passwordType: "password",
      redirect: undefined,
    };
  },
  methods: {
    showPwd() {
      if (this.passwordType === "password") {
        this.passwordType = "";
      } else {
        this.passwordType = "password";
      }
      this.$nextTick(() => {
        this.$refs.password.focus();
      });
    },
    handleLogin() {
      if (this.loginForm.userName != "" && this.loginForm.passWord != "") {
        const that = this;
        let url = "Sign?username=" + this.loginForm.userName;
        that
          .request(url, "", "GET", {})
          .then((res) => {
            if (res.data == "") {
              this.$message({
                message: "当前用户尚未注册！",
                type: "warning",
              });
            } else if (res.data[0].number == this.loginForm.passWord) {
              this.$message({
                message: "登录成功，Welcome！",
                type: "success",
              });
              let obj = {
                id: res.data[0].id,
                username: res.data[0].username,
                authorize: res.data[0].authorize,
                name: res.data[0].name,
                number: res.data[0].number,
                tel: res.data[0].tel,
                email: res.data[0].email,
                deadline: res.data[0].deadline,
              };
              that.request("Signed", obj, "POST");
              this.$router.push({
                path: "/Home",
              });
            } else if (res.data[0].number != this.loginForm.passWord) {
              this.$message({
                message: "用户名或密码错误！",
                type: "error",
              });
            }
          })
          .catch((res) => {
            this.$message({
              message: res.data.errMessage,
              type: "error",
            });
          });
      } else {
        this.$message({
          message: "请填写用户名或密码",
          type: "warning",
        });
      }
    },
  },
};
</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg: #283443;
$light_gray: #fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  .el-input {
    display: inline-block;
    height: 50px;
    width: 85%;
    font-size: 16px;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      // padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 50px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

.login-container {
  // border: 1px solid red;
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;

  .login-form {
    // border: 1px solid red;
    width: 520px;
    max-width: 100%;
    margin: 0 auto;
    overflow: hidden;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      color: $light_gray;
      margin: 0px auto 20px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }
}
</style>
