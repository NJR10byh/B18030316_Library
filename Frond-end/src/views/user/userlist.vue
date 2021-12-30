<template>
  <div class="UserList">
    <div class="head-btn">
      <div class="Search">
        <el-select placeholder="请选择需要搜索的类型" v-model="Select">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option
        ></el-select>
        <el-input
          placeholder="请输入需要查询的内容"
          v-model="SearchText"
        ></el-input>
        <el-button icon="el-icon-search" @click="search">搜索</el-button>
      </div>
      <div class="refresh_handle">
        <div class="handle">
          <el-badge :hidden="hidden" :value="handlenumber" :max="99">
            <el-button icon="el-icon-edit-outline" @click="HandleDialog = true"
              >待处理
            </el-button>
          </el-badge>
        </div>
        <div class="refresh">
          <el-button icon="el-icon-refresh" @click="refresh">刷新 </el-button>
        </div>
      </div>
    </div>
    <!-- table -->
    <el-table
      id="outTable"
      ref="multipleTable"
      :data="tableData"
      tooltip-effect="dark"
      stripe
      class="tablestyle"
      style="width: 100%"
    >
      <el-table-column prop="index" label="序号" width="60" />
      <el-table-column prop="username" label="用户名" />
      <el-table-column prop="authorize" label="权限" width="100" />
      <el-table-column prop="name" label="姓名" />
      <el-table-column prop="number" label="学号" width="120" />
      <el-table-column prop="tel" label="联系方式" />
      <el-table-column prop="email" label="电子邮件" width="160" />
      <el-table-column prop="deadline" label="权限有效期" width="120" />
      <el-table-column
        prop="setting"
        label="操作"
        width="180"
        v-if="authorize == 'system'"
      >
        <template slot-scope="scope">
          <el-button
            @click="handleEdit(scope.$index, scope.row)"
            icon="el-icon-edit-outline"
            >修改</el-button
          >
          <el-button
            @click="handleDelete(scope.$index, scope.row)"
            icon="el-icon-delete"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <div class="PagesCurb">
      <el-pagination
        class="pages"
        :currentPage="currentPage"
        :page-size="10"
        layout="total, prev, pager, next"
        :total="total"
        @current-change="handleCurrentPage"
      />
    </div>
    <el-dialog
      class="Putdialog"
      title="修改信息"
      :visible.sync="PutdialogVisible"
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
        <span>学号：</span
        ><el-input v-model="willput.number" placeholder="请输入学号"></el-input>
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
        <el-button @click="PutdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="PutConfirm()">保 存</el-button>
      </span>
    </el-dialog>
    <el-dialog
      class="Deletedialog"
      title="警告"
      :visible.sync="DeletedialogVisible"
      width="30%"
    >
      <span>确定要删除吗？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="DeletedialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="DeleteConfirm()">删 除</el-button>
      </span>
    </el-dialog>
    <el-dialog
      class="handledialog"
      title="待处理"
      :visible.sync="HandleDialog"
      width="60%"
    >
      <el-table :data="WillHandle">
        <el-table-column prop="index" label="序号" width="60" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="name" label="姓名" />
        <el-table-column prop="number" label="学号" />
        <el-table-column prop="tel" label="联系方式" />
        <el-table-column prop="email" label="电子邮件" width="160" />
        <el-table-column
          prop="setting"
          label="操作"
          width="180"
          v-if="authorize == 'system'"
        >
          <template slot-scope="scope">
            <el-button
              @click="Pass(scope.$index, scope.row)"
              icon="el-icon-circle-check"
              >通过</el-button
            >
            <el-button
              @click="Reject(scope.$index, scope.row)"
              icon="el-icon-circle-close"
              >拒绝</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userid: "",
      // 当前用户权限
      authorize: "",

      // 表格数据
      tableData: [],
      total: 0,
      currentPage: 0,
      SearchText: "",
      Select: "",

      options: [
        {
          value: "username",
          label: "用户名",
        },
        {
          value: "authorize",
          label: "权限",
        },
        {
          value: "name",
          label: "姓名",
        },
        {
          value: "number",
          label: "学号",
        },
        {
          value: "tel",
          label: "联系方式",
        },
        {
          value: "email",
          label: "电子邮件",
        },
      ],

      // 编辑用户
      putid: "",
      PutdialogVisible: false,
      willput: {
        username: "",
        authorize: "",
        name: "",
        number: "",
        tel: "",
        email: "",
        deadline: "",
      },

      // 删除用户
      deleteid: "",
      DeletedialogVisible: false,

      // 待处理
      HandleDialog: false,
      hidden: false,
      handlenumber: 0,
      WillHandle: [],
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
      that.Select = "";
      that.SearchText = "";
      // 获取当前登录用户id
      await that.request("Signed/", {}, "GET", {}).then((res) => {
        that.userid = res.data[0].id;
      });
      // 获取当前登录用户权限
      let url = "Sign/" + that.userid;
      await that.request(url, {}, "GET", {}).then((res) => {
        this.authorize = res.data.authorize;
      });
      // 获取用户列表
      await that
        .request("Sign/", {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.username = res.data[i].username;
            obj.authorize = res.data[i].authorize;
            obj.name = res.data[i].name;
            obj.number = res.data[i].number;
            obj.tel = res.data[i].tel;
            obj.email = res.data[i].email;
            obj.deadline = res.data[i].deadline;
            that.tableData.push(obj);
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
      that.currentPage = 1;

      // 获取未处理数据
      await that.request("Registe/", {}, "GET", {}).then((res) => {
        that.handlenumber = res.data.length;
        if (that.handlenumber != 0) {
          that.hidden = false;
        } else {
          that.hidden = true;
        }
        for (let i = 0; i < res.data.length; i++) {
          let obj = {};
          obj.index = i + 1;
          obj.id = res.data[i].id;
          obj.username = res.data[i].username;
          obj.authorize = res.data[i].authorize;
          obj.name = res.data[i].name;
          obj.number = res.data[i].number;
          obj.tel = res.data[i].tel;
          obj.email = res.data[i].email;
          obj.deadline = res.data[i].deadline;
          that.WillHandle.push(obj);
        }
      });
    },
    async handleCurrentPage(val) {
      let that = this;
      that.tableData = [];
      that.total = 0;
      let url = "teacher/queAllInfo?page=" + val;
      await that
        .request(url, {}, "GET", {})
        .then((res) => {
          that.total = res.data.count;
          for (let i = 0; i < res.data.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.username = res.data[i].username;
            obj.authorize = res.data[i].authorize;
            obj.name = res.data[i].name;
            obj.number = res.data[i].number;
            obj.tel = res.data[i].tel;
            obj.email = res.data[i].email;
            obj.deadline = res.data[i].deadline;
            that.tableData.push(obj);
          }
          this.$message({
            message: "数据已更新",
            type: "success",
          });
        })
        .catch((res) => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
      that.currentPage = val;
    },

    // 搜索用户
    async search() {
      let that = this;
      that.tableData = [];
      let url = "Sign?" + that.Select + "_like=" + that.SearchText;
      await that
        .request(url, {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.username = res.data[i].username;
            obj.authorize = res.data[i].authorize;
            obj.name = res.data[i].name;
            obj.number = res.data[i].number;
            obj.tel = res.data[i].tel;
            obj.email = res.data[i].email;
            obj.deadline = res.data[i].deadline;
            that.tableData.push(obj);
          }
          this.$message({
            message: "数据已更新",
            type: "success",
          });
        })
        .catch((res) => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },

    // 编辑用户
    handleEdit(index, row) {
      let that = this;
      console.log(row);
      that.PutdialogVisible = true;
      that.putid = row.id;
      that.willput.username = row.username;
      that.willput.authorize = row.authorize;
      that.willput.name = row.name;
      that.willput.number = row.number;
      that.willput.tel = row.tel;
      that.willput.email = row.email;
      that.willput.deadline = row.deadline;
    },

    // 修改用户
    async PutConfirm() {
      let that = this;
      let url = "Sign/" + that.putid;
      await that.request(url, that.willput, "PUT", {});
      this.PutdialogVisible = false;
      that.refresh();
    },

    // 删除用户
    handleDelete(index, row) {
      this.DeletedialogVisible = true;
      this.deleteid = row.id;
    },

    // 确认用户
    async DeleteConfirm() {
      let that = this;
      that.tableData = [];
      that.total = 0;
      let url = "Sign/" + that.deleteid;
      await that
        .request(url, {}, "DELETE", {})
        .then(() => {
          this.DeletedialogVisible = false;
          that.refresh();
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },

    // 通过预处理
    async Pass(index, row) {
      let that = this;
      let obj = {};
      obj.username = row.username;
      obj.authorize = "reader";
      obj.name = row.name;
      obj.number = row.number;
      obj.tel = row.tel;
      obj.email = row.email;
      obj.username = row.username;
      obj.deadline = "2022-06-30";
      let url = "Registe/" + row.id;
      await that
        .request(url, {}, "DELETE", {})
        .then(() => {
          that
            .request("Sign", obj, "POST", {})
            .then(() => {
              this.$message({
                message: "已通过",
                type: "success",
              });
              that.HandleDialog = false;
              that.refresh();
            })
            .catch(() => {
              this.$message({
                message: "数据请求失败",
                type: "error",
              });
            });
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },

    // 拒绝预处理
    async Reject(index, row) {
      let that = this;
      let url = "Registe/" + row.id;
      await that
        .request(url, {}, "DELETE", {})
        .then(() => {
          this.$message({
            message: "已拒绝",
            type: "success",
          });
          that.HandleDialog = false;
          that.refresh();
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },
  },
};
</script>
<style lang="scss">
.UserList {
  padding: 10px;
  display: flex;
  flex-direction: column;

  .head-btn {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin: 0 0 10px 0;
    // border: 1px solid red;
    .Search {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      width: 500px;
      // border: 1px solid red;

      .el-input {
        width: 200px;
      }
      .el-button {
        padding: 0 10px;
        border-radius: 3px;
        font-size: 14px;
        width: 80px;
        border: 1px solid #409eff;
        color: #ffffff;
        font-weight: bold;
        background-color: #409eff;
      }
    }

    .refresh_handle {
      display: flex;
      justify-content: center;
      align-items: center;
      // border: 1px solid red;
      .handle {
        .el-button {
          padding: 0 10px;
          height: 30px;
          border-radius: 5px;
          font-size: 14px;
          border: 1px solid #409eff;
          color: #409eff;
        }
      }
      .refresh {
        .el-button {
          padding: 0 10px;
          height: 30px;
          border-radius: 5px;
          font-size: 14px;
          border: 1px solid #409eff;
          color: #409eff;
          margin-left: 30px;
        }

        // &:last-child {
        //   margin-right: 5px;
        // }
      }
    }
  }

  .tablestyle {
    // border: 1px solid red;
    &::before {
      display: none;
    }

    .el-table__header {
      th {
        text-align: center;
        background-color: #fafafa;
        border: 1px solid #e9ecf2;

        .cell {
          font-size: 15px;
        }
      }
    }

    .el-checkbox__inner {
      width: 16px;
      height: 16px;
      background: #ffffff;
      border: 1px solid #dddddd;
      border-radius: 4px;
    }

    .el-checkbox__inner::after {
      left: 5px;
    }

    .el-table__body {
      td {
        text-align: center;
        border-right: 1px solid #dddddd;

        &:first-child {
          border-left: 1px solid #dddddd;
        }

        .cell {
          color: #333;
        }
      }
      .el-button {
        border: none;
        padding: 5px 10px;
        background: transparent;
        &:first-child:hover {
          color: #409eff;
        }
        &:nth-child(2):hover {
          color: #f96b6c;
        }
      }
    }
  }

  .PagesCurb {
    margin-top: 20px;

    .pages {
      // border: 1px solid red;
      text-align: center;
    }
  }

  .Putdialog {
    div {
      margin-top: 7px;
      span {
        font-size: 17px;
        font-weight: bold;
        // color: #304156;
      }
    }
  }
  .handledialog {
    .el-button {
      border: none;
      padding: 3px;
      background: transparent;
      &:first-child:hover {
        color: #409eff;
      }
      &:nth-child(2):hover {
        color: #f96b6c;
      }
    }
  }
}
</style>
