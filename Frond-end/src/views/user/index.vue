<template>
  <div class="UserInfo">
    <div class="Left">
      <div class="MainInfo">
        <div class="Ava">
          <img src="../../assets/images/ava.jpg" alt="" />
        </div>
        <div class="username">{{ userinfo.username }}</div>
        <div class="authorize">{{ userinfo.authorize }}</div>
      </div>
    </div>
    <div class="Right">
      <div class="right_box">
        <div class="title">
          <div class="top">用户信息</div>
          <div class="below">
            管理你的个人信息，其中包括可以联系到您的电话号码和电子邮件地址
          </div>
        </div>
        <div class="Alert">
          <el-button @click="Alert()" icon="el-icon-edit-outline"
            >修改信息</el-button
          >
        </div>
        <div class="InfoDetail">
          <div class="Part_1">
            <div class="part1_1">
              <div class="Info_Menu">姓名</div>
              <div class="Info_Answer">
                <span>{{ userinfo.name }}</span>
              </div>
            </div>
            <div class="part1_1 part1_2">
              <div class="Info_Menu">工号 / 学号</div>
              <div class="Info_Answer">
                <span>{{ userinfo.number }}</span>
              </div>
            </div>
          </div>
          <div class="Part_1">
            <div class="part1_1">
              <div class="Info_Menu">联系方式：</div>
              <div class="Info_Answer">
                <span>{{ userinfo.tel }}</span>
              </div>
            </div>
            <div class="part1_1 part1_2">
              <div class="Info_Menu">电子邮件：</div>
              <div class="Info_Answer">
                <span>{{ userinfo.email }}</span>
              </div>
            </div>
          </div>
          <div class="Part_1">
            <div class="part1_1">
              <div class="Info_Menu">权限有效期</div>
              <div class="Info_Answer">
                <span>{{ userinfo.deadline }}</span>
              </div>
            </div>
            <div
              class="part1_1 part1_2 Manage"
              v-if="userinfo.authorize == '系统管理员'"
              @click="Manager()"
            >
              <div class="Info_Menu">管理用户</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <el-dialog
      class="Alertdialog"
      title="修改信息"
      :visible.sync="AlertdialogVisible"
      width="30%"
    >
      <div>
        <span>用户名：</span
        ><el-input
          v-model="willput.username"
          placeholder="请输入用户名"
        ></el-input>
      </div>
      <div>
        <span>姓名：</span
        ><el-input v-model="willput.name" placeholder="请输入姓名"></el-input>
      </div>
      <div>
        <span>工号 / 学号：</span
        ><el-input
          v-model="willput.number"
          placeholder="请输入工号 / 学号"
        ></el-input>
      </div>
      <div>
        <span>联系方式：</span
        ><el-input
          v-model="willput.tel"
          placeholder="请输入联系方式"
        ></el-input>
      </div>
      <div>
        <span>电子邮件：</span
        ><el-input
          v-model="willput.email"
          placeholder="请输入电子邮件"
        ></el-input>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="AlertdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="AlertConfirm()">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 当前用户数据
      userinfo: {
        id: 0,
        username: "",
        authorize: "",
        name: "",
        number: "",
        tel: "",
        email: "",
        deadline: "",
      },

      putid: "",
      willput: {
        username: "",
        authorize: "",
        name: "",
        number: "",
        tel: "",
        email: "",
        deadline: "",
      },

      AlertdialogVisible: false,
    };
  },
  created() {
    this.refresh();
  },
  methods: {
    // 刷新数据
    async refresh() {
      let that = this;
      that.tableData = [];
      that.total = 0;
      await that.request("Signed", {}, "GET", {}).then((res) => {
        that.userinfo.id = res.data[0].id;
      });
      let url = "Sign/" + that.userinfo.id;
      await that
        .request(url, {}, "GET", {})
        .then((res) => {
          that.userinfo.username = res.data.username;
          that.userinfo.name = res.data.name;
          that.userinfo.number = res.data.number;
          that.userinfo.tel = res.data.tel;
          that.userinfo.email = res.data.email;
          that.userinfo.deadline = res.data.deadline;
          console.log(res.data.authorize);
          switch (res.data.authorize) {
            case "system":
              that.userinfo.authorize = "系统管理员";
              break;
            case "book":
              that.userinfo.authorize = "图书管理员";
              break;
            case "reader":
              that.userinfo.authorize = "读者";
              break;
            default:
              break;
          }
          this.$message({
            message: "数据已更新",
            type: "success",
          });
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },

    // 编辑用户信息
    Alert() {
      let that = this;
      that.AlertdialogVisible = true;
      that.willput.username = that.userinfo.username;
      that.willput.name = that.userinfo.name;
      that.willput.number = that.userinfo.number;
      that.willput.tel = that.userinfo.tel;
      that.willput.email = that.userinfo.email;
    },

    // 修改用户信息
    async AlertConfirm() {
      let that = this;
      that.willput.authorize = that.userinfo.authorize;
      that.willput.deadline = that.userinfo.deadline;
      let url = "Sign/" + that.userinfo.id;
      await that.request(url, that.willput, "PUT", {});
      that.AlertdialogVisible = false;
      that.refresh();
    },

    // 管理用户
    Manager() {
      this.$router.push({
        path: "/userlist",
      });
    },
  },
};
</script>
<style lang="scss">
.UserInfo {
  width: 100vw;
  height: 90vh;
  display: flex;
  justify-content: center;
  align-items: center;
  //   border: 1px solid red;
  .Left {
    width: 25%;
    height: 100%;
    // border: 1px solid red;
    display: flex;
    justify-content: center;
    align-items: center;
    .MainInfo {
      width: 50%;
      height: 50%;
      //   border: 1px solid red;
      .Ava {
        img {
          width: 100px;
          height: 100px;
          border-radius: 8px;
        }
      }
      .username {
        font-size: 30px;
        font-weight: bold;
        margin-top: 20px;
      }
      .authorize {
        font-size: 18px;
        color: #999;
        margin-top: 5px;
      }
    }
  }
  .Right {
    width: 75%;
    height: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    // border: 1px solid red;
    .right_box {
      width: 95%;
      height: 90%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      //   border: 1px solid red;
      .title {
        width: 100%;
        height: 9%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: flex-start;
        // border: 1px solid red;
        .top {
          font-size: 25px;
          font-weight: bold;
          letter-spacing: 1.5px;
        }
        .below {
          font-size: 15px;
          color: #999;
        }
      }
      .Alert {
        width: 100%;
        height: 7%;
        display: flex;
        justify-content: flex-start;
        align-items: flex-end;
        // border: 1px solid red;
        .el-button {
          padding: 0 10px;
          height: 30px;
          border-radius: 5px;
          font-size: 14px;
          border: 1px solid #409eff;
          color: #409eff;
        }
      }
      .InfoDetail {
        width: 100%;
        height: 83%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: flex-start;
        // border: 1px solid red;
        .Part_1 {
          width: 75%;
          height: 33%;
          display: flex;
          justify-content: flex-start;
          align-items: center;
          //   border: 1px solid red;
          .part1_1 {
            width: 40%;
            height: 80%;
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            border-radius: 8px;
            border: 1px solid #cdcdcd;
            box-shadow: 6px 6px 5px #eee;
            .Info_Menu {
              font-size: 23px;
              font-weight: bold;
              margin: 20px 0 0 20px;
              //   border: 1px solid red;
            }
            .Info_Answer {
              margin: 10px 0 0 20px;
              //   border: 1px solid red;
              span {
                color: #999;
              }
            }
          }
          .part1_2 {
            margin-left: 50px;
          }
          .Manage {
            background: #409eff;
            border: 0;
            color: #fff;
            cursor: pointer;
          }
        }
      }
    }
  }
  .Alertdialog {
    div {
      margin-top: 8px;
      span {
        font-size: 17px;
        font-weight: bold;
      }
    }
  }
}
</style>
