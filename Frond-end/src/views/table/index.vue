<template>
  <div class="Table">
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
      <div class="newin_refresh">
        <div class="refresh up" v-if="authorize == 'book'">
          <el-button icon="el-icon-notebook-1" @click="AddDialogVisible = true"
            >新增图书
          </el-button>
        </div>
        <div class="refresh up" v-if="authorize == 'book'">
          <el-button icon="el-icon-top" @click="Borrow_Up()"
            >最后借阅
          </el-button>
        </div>
        <div class="refresh down" v-if="authorize == 'book'">
          <el-button icon="el-icon-bottom" @click="Borrow_Down()"
            >最后借阅
          </el-button>
        </div>
        <div class="refresh newin" v-if="authorize == 'book'">
          <el-button icon="el-icon-collection" @click="NewInKu()"
            >最近入库（12月）
          </el-button>
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
      <el-table-column prop="index" label="序号" width="70" />
      <el-table-column prop="name" label="书籍名称" />
      <el-table-column prop="author" label="作者" />
      <el-table-column prop="PublishHouse" label="出版社" />
      <el-table-column prop="PublishDate" label="出版日期" />
      <el-table-column prop="Isbn" label="ISBN（书号）" />
      <el-table-column prop="InDate" label="入库日期" />
      <el-table-column prop="LastBorrowDate" label="最后借阅日期" />
      <el-table-column
        prop="setting"
        label="操作"
        width="180"
        v-if="authorize == 'book'"
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
      <el-table-column
        prop="setting"
        label="操作"
        width="180"
        v-if="authorize == 'reader'"
      >
        <template slot-scope="scope">
          <el-button
            @click="Ding(scope.$index, scope.row)"
            icon="el-icon-notebook-2"
            :disabled="!scope.row.Ifborrow"
            >预定</el-button
          >
          <el-button
            @click="Jie(scope.$index, scope.row)"
            icon="el-icon-reading"
            :disabled="scope.row.Ifborrow"
            >借阅</el-button
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
        <span>书籍名称：</span
        ><el-input
          v-model="willput.name"
          placeholder="请输入书籍名称"
        ></el-input>
      </div>
      <div>
        <span>作者：</span
        ><el-input v-model="willput.author" placeholder="请输入作者"></el-input>
      </div>
      <div>
        <span>出版社：</span
        ><el-input
          v-model="willput.PublishHouse"
          placeholder="请输入出版社"
        ></el-input>
      </div>
      <div>
        <span>出版日期：</span
        ><el-input
          v-model="willput.PublishDate"
          placeholder="请输入出版日期"
        ></el-input>
      </div>
      <div>
        <span>ISBN（书号）：</span
        ><el-input
          v-model="willput.Isbn"
          placeholder="请输入ISBN（书号）"
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
      title="最近入库（12月入库）"
      :visible.sync="HandleDialog"
      width="60%"
    >
      <el-table :data="NewIn">
        <el-table-column prop="name" label="书籍名称" />
        <el-table-column prop="author" label="作者" />
        <el-table-column prop="PublishHouse" label="出版社" />
        <el-table-column prop="PublishDate" label="出版日期" />
        <el-table-column prop="Isbn" label="ISBN（书号）" />
        <el-table-column prop="InDate" label="入库日期" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="HandleDialog = false">取 消</el-button>
        <el-button type="primary" @click="HandleDialog = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <el-dialog
      class="Putdialog"
      title="新增图书"
      :visible.sync="AddDialogVisible"
      width="30%"
    >
      <div>
        <span>书籍名称：</span
        ><el-input
          v-model="willadd.name"
          placeholder="请输入书籍名称"
        ></el-input>
      </div>
      <div>
        <span>作者：</span
        ><el-input v-model="willadd.author" placeholder="请输入作者"></el-input>
      </div>
      <div>
        <span>出版社：</span
        ><el-input
          v-model="willadd.PublishHouse"
          placeholder="请输入出版社"
        ></el-input>
      </div>
      <div>
        <span>出版日期：</span
        ><el-input
          v-model="willadd.PublishDate"
          placeholder="请输入出版日期"
        ></el-input>
      </div>
      <div>
        <span>ISBN（书号）：</span
        ><el-input
          v-model="willadd.Isbn"
          placeholder="请输入ISBN（书号）"
        ></el-input>
      </div>
      <div>
        <span>入库日期：</span
        ><el-input
          v-model="willadd.InDate"
          placeholder="请输入入库日期"
        ></el-input>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="AddDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="AddConfirm()">新 增</el-button>
      </span>
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
          value: "name",
          label: "书籍名称",
        },
        {
          value: "author",
          label: "作者",
        },
        {
          value: "PublishHouse",
          label: "出版社",
        },
        {
          value: "PublishDate",
          label: "出版日期",
        },
        {
          value: "Isbn",
          label: "ISBN（书号）",
        },
      ],

      putid: "",
      deleteid: "",
      willput: {
        id: "",
        name: "",
        author: "",
        PublishHouse: "",
        PublishDate: "",
        Isbn: "",
        Ifborrow: "",
        InDate: "",
        LastBorrowDate: "",
      },

      PutdialogVisible: false,
      DeletedialogVisible: false,

      // 最近入库
      HandleDialog: false,
      NewIn: [],

      // 新增图书
      AddDialogVisible: false,
      willadd: {
        name: "",
        author: "",
        PublishHouse: "",
        PublishDate: "",
        Isbn: "",
        Ifborrow: "",
        InDate: "",
        LastBorrowDate: "",
      },
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
      await that.request("Signed", {}, "GET", {}).then((res) => {
        that.userid = res.data[0].id;
      });
      let url = "Sign/" + that.userid;
      await that.request(url, {}, "GET", {}).then((res) => {
        this.authorize = res.data.authorize;
      });
      await that
        .request("Library", {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
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
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
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

    // 搜索书籍
    async search() {
      let that = this;
      that.tableData = [];
      let url = "Library?" + that.Select + "_like=" + that.SearchText;
      await that
        .request(url, {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
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

    // 编辑书籍
    handleEdit(index, row) {
      let that = this;
      that.PutdialogVisible = true;
      that.putid = row.id;
      that.willput.name = row.name;
      that.willput.author = row.author;
      that.willput.PublishHouse = row.PublishHouse;
      that.willput.PublishDate = row.PublishDate;
      that.willput.Isbn = row.Isbn;
      that.willput.Ifborrow = row.Ifborrow;
      that.willput.InDate = row.InDate;
      that.willput.LastBorrowDate = row.LastBorrowDate;
    },

    // 修改书籍
    async PutConfirm() {
      let that = this;
      let url = "Library/" + that.putid;
      await that
        .request(url, that.willput, "PUT", {})
        .then(() => {
          this.PutdialogVisible = false;
          that.refresh();
        })
        .catch(() => {});
    },

    // 删除书籍
    handleDelete(index, row) {
      this.DeletedialogVisible = true;
      this.deleteid = row.id;
    },

    // 确认删除
    async DeleteConfirm() {
      let that = this;
      that.tableData = [];
      that.total = 0;
      let url = "Library/" + that.deleteid;
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

    // 新增图书
    async AddConfirm() {
      let that = this;
      that.willadd.Ifborrow = false;
      that.willadd.LastBorrowDate = "2021-12-31";
      console.log(that.willadd);
      await that
        .request("Library", that.willadd, "POST", {})
        .then(() => {
          that.AddDialogVisible = false;
          that.refresh();
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败",
            type: "error",
          });
        });
    },

    // 预定
    async Ding(index, row) {
      let that = this;
      let ding = {};
      ding.name = row.name;
      ding.author = row.author;
      ding.PublishHouse = row.PublishHouse;
      ding.PublishDate = row.PublishDate;
      ding.Isbn = row.Isbn;
      ding.Ifborrow = !row.Ifborrow;
      ding.InDate = row.InDate;
      ding.LastBorrowDate = row.LastBorrowDate;
      let url = "Library/" + row.id;
      await that
        .request(url, ding, "PUT", {})
        .then(() => {
          this.$message({
            message: "预定成功",
            type: "success",
          });
          that.refresh();
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败，预定失败",
            type: "error",
          });
        });
    },
    // 借阅
    async Jie(index, row) {
      let that = this;
      let jie = {};
      jie.name = row.name;
      jie.author = row.author;
      jie.PublishHouse = row.PublishHouse;
      jie.PublishDate = row.PublishDate;
      jie.Isbn = row.Isbn;
      jie.Ifborrow = !row.Ifborrow;
      jie.InDate = row.InDate;
      jie.LastBorrowDate = row.LastBorrowDate;
      let url = "Library/" + row.id;
      await that
        .request(url, jie, "PUT", {})
        .then(() => {
          this.$message({
            message: "借阅成功",
            type: "success",
          });
          that.refresh();
        })
        .catch(() => {
          this.$message({
            message: "数据请求失败，借阅失败",
            type: "error",
          });
        });
    },

    // 借阅历史（升序）
    async Borrow_Up() {
      let that = this;
      that.tableData = [];
      await that
        .request("Library?_sort=LastBorrowDate&_order=asc", {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
            that.tableData.push(obj);
          }
          this.$message({
            message: "已升序排列",
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
    // 借阅历史（降序）
    async Borrow_Down() {
      let that = this;
      that.tableData = [];
      await that
        .request("Library?_sort=LastBorrowDate&_order=desc", {}, "GET", {})
        .then((res) => {
          that.total = res.data.length;
          for (let i = 0; i < res.data.length; i++) {
            let obj = {};
            obj.index = i + 1;
            obj.id = res.data[i].id;
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
            that.tableData.push(obj);
          }
          this.$message({
            message: "已降序排列",
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
    // 最近入库
    async NewInKu() {
      let that = this;
      that.NewIn = [];
      await that.request("Library", {}, "GET", {}).then((res) => {
        for (let i = 0; i < res.data.length; i++) {
          if (
            new Date(res.data[i].InDate).getTime() >
            new Date("2021-12-01").getTime()
          ) {
            let obj = {};
            obj.id = res.data[i].id;
            obj.name = res.data[i].name;
            obj.author = res.data[i].author;
            obj.PublishHouse = res.data[i].PublishHouse;
            obj.PublishDate = res.data[i].PublishDate;
            obj.Isbn = res.data[i].Isbn;
            obj.Ifborrow = res.data[i].Ifborrow;
            obj.InDate = res.data[i].InDate;
            obj.LastBorrowDate = res.data[i].LastBorrowDate;
            that.NewIn.push(obj);
          }
        }
      });
      that.HandleDialog = true;
    },
  },
};
</script>
<style lang="scss">
.Table {
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
    .newin_refresh {
      display: flex;
      justify-content: center;
      align-items: center;
      // border: 1px solid red;
      .refresh {
        margin-left: 10px;
        .el-button {
          padding: 0 10px;
          height: 30px;
          border-radius: 5px;
          font-size: 14px;
          border: 1px solid #409eff;
          color: #409eff;
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
}
</style>
