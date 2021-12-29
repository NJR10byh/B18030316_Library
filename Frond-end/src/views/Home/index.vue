<template>
  <div class="Box">
    <div class="userinfo">
      <div class="user">
        Welcome，<span>{{ username }}</span>
      </div>
      <!-- <div class="authorize">权限：系统管理员</div> -->
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userid: 0,
      username: "",
    };
  },
  created() {
    this.getuesrinfo();
  },
  methods: {
    async getuesrinfo() {
      let that = this;
      await that.request("Signed", {}, "GET", {}).then((res) => {
        that.userid = res.data[0].id;
      });
      let url = "Sign/" + that.userid;
      await that.request(url, {}, "GET", {}).then((res) => {
        console.log(res.data);
        that.username = res.data.username;
      });
    },
  },
};
</script>

<style lang="scss" >
.Box {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  width: 100vw;
  // border: 1px solid red;
  .userinfo {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    height: 60vh;
    margin-left: 20px;
    // border: 1px solid red;

    .user {
      margin-top: 100px;
      font-size: 50px;
      font-weight: bold;
      // border: 1px solid red;
      span {
        color: #409eff;
      }
    }
  }
}
</style>