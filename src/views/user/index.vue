<template>
  <div class="app-container" icon="404">
    <!-- 头部 -->
    <div style="display: flex; justify-content: space-between">
      <el-dropdown>
        <el-button type="danger">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="changeCommentStatus(true)"
            >禁止评论</el-dropdown-item
          >
          <el-dropdown-item @click.native="changeAccessStatus(true)"
            >禁止访问</el-dropdown-item
          >
        </el-dropdown-menu>
      </el-dropdown>
      <div>
        <el-input
          v-model="listQuery.title"
          placeholder="用户名"
          style="width: 180px"
        ></el-input>
        <el-button icon="el-icon-search" type="primary" @click="handQuery"
          >搜索</el-button
        >
      </div>
    </div>
    <!-- 表格 -->
    <el-row>
      <el-table
        :data="tableData"
        style="width: 100%"
        v-loading="isloading"
        @selection-change="handleSelectionChange"
        ref="multipleTable"
      >
        <el-table-column type="selection" width="55"> </el-table-column>
        <el-table-column label="用户" width="280px">
          <template slot-scope="scope">
            <div style="display: flex">
              <el-avatar :size="50" :src="scope.row.user.avatar"></el-avatar>
              <p>{{ scope.row.user.username }}</p>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="total_consume" label="消费总额" width="120px">
        </el-table-column>
        <el-table-column prop="updated_time" label="创建时间" width="150px">
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="small"
              @click="handleOpenInfo(scope.row.user.id)"
              >详情</el-button
            >
            <el-button
              type="success"
              size="small"
              @click="handleEdit(scope.$index, scope.row)"
              >联系用户</el-button
            >
            <el-dropdown size="mini">
              <el-button type="danger">
                黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
              </el-button>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item
                  @click.native="changeCommentStatus(scope.row.user.id)"
                  >禁止评论</el-dropdown-item
                >
                <el-dropdown-item
                  @click.native="changeAccessStatus(scope.row.user.id)"
                  >禁止访问</el-dropdown-item
                >
              </el-dropdown-menu>
            </el-dropdown>
          </template> </el-table-column
        >>
      </el-table>
    </el-row>
    <!-- 分页处理 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :page-size="listQuery.limit"
      layout="total, sizes, prev, pager, next, jumper"
      :total="listQuery.total"
    >
    </el-pagination>
    <!-- 个人信息组件 -->
    <Info ref="info"></Info>
  </div>
</template>

<script>
import { fetchList, changeAccessStatus, changeCommentStatus } from "@/api/user";
import Info from './Info'
export default {
  name: "Index",
  components:{
    Info
  },
  data() {
    return {
      // 数据源
      tableData: [],

      // 控制加载开关
      isloading: false,
      // 查询分页请求后端的参数
      listQuery: {
        //  数据总数
        total: 0,
        // 每页显示多少数据
        limit: 20,
        //当前第几页
        page: 1,
        // 查询的信息的头部
        title: "",
      },
      // 收集表单选项变化的数组
      multipleSelection: [],
    };
  },
  mounted() {
    this.getTableData();
  },
  methods: {
    // 查询数据
    async getTableData() {
      this.isloading = true;
      let res = await fetchList(this.listQuery);
      // console.log(res);
      this.tableData = res.data.items;
      this.listQuery.total = res.data.total;
      setTimeout(() => {
        this.isloading = false;
      }, 1000);
    },
    // 搜索数据
    handQuery() {
      this.listQuery.page = 1;
      this.getTableData();
    },

    // 下面的是element配置

    // 分页每页大小发生改变时触发
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`);
      this.listQuery.limit = val;
      this.getTableData();
    },
    //页码发生改变时触发
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.listQuery.page = val;
      this.getTableData();
    },
    // element中收集表单选项变化
    handleSelectionChange(val) {
      this.multipleSelection = val;
      // console.log(val);
    },
    // 改变用户禁止评论状态
    changeCommentStatus(ids) {
      // console.log(id);
      if (typeof ids == "boolean" && !this.multipleSelection.length) {
        return this.$message.error("请先选择用户");
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      let alretText = typeof ids == "boolean" ? "选中" : "当前";
      this.$confirm("是否要禁止" + alretText + "用户评论?", "提示", {
        distinguishCancelAndClose: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then((action) => {
          this.isloading = true;
          changeCommentStatus(id, (status = 0));
        })
        .then(() => {
          this.$message({
            type: "success",
            message: "已成功禁止用户评论!",
          });
          this.isloading = false;
          this.$refs.multipleTable.clearSelection();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消禁止用户评论",
          });
        });
    },
    // 改变用户禁止访问状态
    changeAccessStatus(ids) {
      // console.log(typeof ids);
      if (typeof ids == "boolean" && !this.multipleSelection.length) {
        return this.$message({
          message: "请先选择用户",
          type: "warning",
        });
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      let alertText = typeof ids == "boolean" ? "选中" : "当前";
      this.$confirm("此操作将禁止" + alertText + "用户访问", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action)=>{
        this.isloading=true
        changeAccessStatus(
          id,
          status=0
        )
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "用户禁止访问成功!",
          });
          this.$refs.multipleTable.clearSelection()
          this.isloading=false
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消用户禁止访问",
          });
        });
    },
    // 查看个人信息
    handleOpenInfo(id){
      // console.log(id);
      this.$refs.info.open(id)
    }
  },
};
</script>
