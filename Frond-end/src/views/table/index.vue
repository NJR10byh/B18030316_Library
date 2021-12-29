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
      <div class="refresh">
        <el-button icon="el-icon-refresh" @click="refresh">刷新 </el-button>
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
      <el-table-column
        prop="setting"
        label="操作"
        width="180"
        v-if="authorize == 'book'"
      >
        <template slot-scope="scope">
          <el-button
            @click="handleEdit(scope.$index, scope.row)"
            icon="el-icon-edit"
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
          label: "书名",
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
      },

      PutdialogVisible: false,
      DeletedialogVisible: false,
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
            obj.id = res.data.data[i].id;
            obj.name = res.data.data[i].name;
            obj.score = res.data.data[i].score;
            obj.counter = res.data.data[i].counter;
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
      let url = "Library?" + that.Select + "=" + that.SearchText;
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
      that.willput.id = row.id;
      that.willput.name = row.name;
      that.willput.author = row.author;
      that.willput.PublishHouse = row.PublishHouse;
      that.willput.PublishDate = row.PublishDate;
      that.willput.Isbn = row.Isbn;
    },

    // 修改书籍
    async PutConfirm() {
      let that = this;
      that.willput.id = that.putid;
      let url = "Library/" + that.putid;
      await that.request(url, that.willput, "PUT", {});
      this.PutdialogVisible = false;
      that.refresh();
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
      await that.request(url, {}, "DELETE", {});
      this.DeletedialogVisible = false;
      that.refresh();
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

    .refresh {
      .el-button {
        padding: 0 10px;
        height: 30px;
        border-radius: 5px;
        font-size: 14px;
        width: 75px;
        border: 1px solid #409eff;
        color: #409eff;
      }

      &:last-child {
        margin-right: 5px;
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
