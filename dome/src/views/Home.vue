<template>
  <el-container class="wrap">
    <el-header>Header</el-header>
    <el-container>
      <el-aside width="200px">Aside</el-aside>
      <el-main>
        <el-button type="primary" @click="dialogFormVisible=true">添加</el-button>
        <el-table :data="tableData" style="width: 100%">
          <el-table-column label="title" prop="title"></el-table-column>
          <el-table-column label="type" prop="type"></el-table-column>
          <el-table-column label="time" prop="time"></el-table-column>
          <el-table-column align="right">
            <template slot="header">操作</template>
            <template slot-scope="scope">
              <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">Edit</el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDelete(scope.$index, scope.row)"
              >Delete</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          background
          layout="sizes,prev, pager, next"
          :total="total"
          :page-size="limit"
          @current-change="changePage"
          @size-change="handleSizeChange"
          :page-sizes="[1, 2, 4, 6]"
        ></el-pagination>
      </el-main>
      <el-dialog title="添加成员" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="title" :label-width="formLabelWidth">
            <el-input v-model="form.title" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="type" :label-width="formLabelWidth">
            <el-input v-model="form.type" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="time" :label-width="formLabelWidth">
            <el-input v-model="form.time" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="curFun">确 定</el-button>
        </div>
      </el-dialog>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      tableData: [], //当前页数据
      pagenum: 1, //当前的页码
      limit: 1, //每页显示的条数
      total: 0, //总条数
      dialogFormVisible: false,
      form: {
        title: "",
        type: "",
        time: ""
      },
      formLabelWidth: "120px",
      id: null //修改ID
    };
  },
  created() {
    this.getCurData();
  },
  methods: {
    getCurData() {
      this.$http
        .get("/api/list", {
          params: { pagenum: this.pagenum, limit: this.limit }
        })
        .then(res => {
          if (res.data.code === 1) {
            this.tableData = res.data.data;
            this.total = res.data.total;
          } else {
            alert(res.data.msg);
          }
        });
    },
    handleEdit(index, row) {
      //第一步：把弹窗打开
      this.dialogFormVisible = true;
      //第二步：id赋值
      this.id = row.id;
      //第三步：给弹窗的表单赋值
      this.form = {
        title: row.title,
        type: row.type,
        time: row.time
      };
    },
    handleDelete(index, row) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          //删除接口
          this.$http.get("/api/del", { params: { id: row.id } }).then(res => {
            if (res.data.code === 1) {
              this.$message({
                type: "success",
                message: "删除成功!"
              });
              this.getCurData();
            } else {
              this.$message({
                type: "info",
                message: res.data.msg
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //改变页码
    changePage(cur) {
      console.log(cur);
      this.pagenum = cur;
      this.getCurData();
    },
    //添加  修改
    curFun() {
      let url = "";
      if (this.id) {
        //修改
        url = "/api/edit";
      } else {
        //添加
        url = "/api/add";
      }
      this.$http.post(url, { ...this.form, id: this.id }).then(res => {
        if (res.data.code === 1) {
          this.getCurData();
        }
        this.reset();
        this.$message({
          type: "info",
          message: res.data.msg
        });
        this.dialogFormVisible = false;
      });
    },
    reset() {
      this.form = {
        title: "",
        type: "",
        time: ""
      };
      this.id = null;
    },
    //每页条数发生改变
    handleSizeChange(num) {
      this.limit = num;
      this.getCurData();
    }
  }
};
</script>

<style>
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #d3dce6;
}
.wrap {
  height: 100%;
  overflow: hidden;
}
</style>