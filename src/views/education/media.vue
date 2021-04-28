<template>
  <div class="app-container">
    <div class="filter-container">
      <!-- 选择界面 -->
      <el-row style="display: flex; justify-content: space-between">
        <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
          >新增图书</el-button
        >
        <el-select v-model="value" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
        <el-input
          v-model="listQuery.title"
          placeholder="请输入图书名称"
          style="width: 150px"
        ></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handQuery"
          >搜索</el-button
        >
      </el-row>
      <!-- 列表页面 -->
      <el-row>
        <el-table
          :data="filterTable"
          border
          style="width: 100%"
          v-loading="loading"
        >
          <el-table-column prop="id" label="ID" width="120"> </el-table-column>
          <el-table-column label="图书内容" width="350">
            <template slot-scope="scope">
              <div style="display: flex">
                <img
                  style="width: 100px; height: 50px"
                  :src="scope.row.cover"
                  alt=""
                />
                <div>
                  <p>{{ scope.row.title }}</p>
                  <p style="color: red">￥{{ scope.row.price }}</p>
                </div>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="t_price" label="订阅量" width="120">
          </el-table-column>
          <el-table-column prop="status" label="状态" width="120">
            <template slot-scope="scope">
              <el-button type="info">
                {{ scope.row.status ? "已上架" : "未上架" }}</el-button
              >
            </template>
          </el-table-column>
          <el-table-column prop="created_time" label="创建时间" width="200">
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="{ row, $index }">
              <el-button
                type="primary"
                icon="el-icon-edit"
                @click="handUpdata(row)"
                >编辑</el-button
              >

              <el-button
                v-if="row.status == 0"
                size="mini"
                type="success"
                @click="handleModifyStatus(row, 1)"
              >
                上架
              </el-button>
              <el-button
                v-if="row.status == 1"
                size="mini"
                @click="handleModifyStatus(row, 0)"
              >
                下架
              </el-button>

              <el-button
                type="danger"
                icon="el-icon-delete"
                @click="handDelete(row, $index)"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </el-row>
      <!-- 分页页面 -->
      <el-row>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :page-size="listQuery.limit"
          layout="total, sizes, prev, pager, next, jumper"
          :total="listQuery.total"
        >
        </el-pagination>
      </el-row>
      <!-- 模态框 -->
      <el-dialog
        :title="textMap[dialogStatus]"
        :visible.sync="dialogFormVisible"
        width="100%"
        :before-close="handleClose"
      >
        <span slot="footer" class="dialog-footer">
          <!-- 提交表单 -->
          <el-form ref="dataForm" :model="temp">
            <el-form-item label="标题">
              <el-input v-model="temp.title"></el-input>
            </el-form-item>
            <el-form-item label="封面">
              <el-upload
                action="https://jsonplaceholder.typicode.com/posts/"
                list-type="picture-card"
                :on-preview="handlePictureCardPreview"
                :on-remove="handleUploadRemove"
                :on-success="handleUploadSuccess"
              >
                <i class="el-icon-plus"></i>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible">
                <img width="100%" :src="dialogImageUrl" alt="" />
              </el-dialog>
            </el-form-item>
            <el-form-item label="内容">
              <Tinymce v-model="temp.content"></Tinymce>
            </el-form-item>
            <el-form-item label="课程价格">
              <el-input-number v-model="temp.price" :min="0"></el-input-number>
            </el-form-item>
            <el-form-item label="状态">
              <el-radio-group v-model="temp.status">
                <el-radio :label="0">下架</el-radio>
                <el-radio :label="1">上架</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-form>
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="dialogStatus === 'create' ? createData() : updateData()"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
import Tinymce from "@/components/Tinymce";
export default {
  components: {
    Tinymce,
  },
  data() {
    return {
      textMap: {
        update: "修改",
        create: "新增",
      },
      dialogImageUrl: "",
      dialogVisible: false,
      options: [
        {
          value: "选项1",
          label: "已上架",
        },
        {
          value: "选项2",
          label: "未上架",
        },
      ],
      value: "",

      // 数据源
      tableData: [],

      listQuery: {
        //   分页总数
        total: 0,
        // 每页的数量
        limit: 20,
        // 第几页
        page: 1,
        // 查询的标题
        title:undefined
      },
      //  页面加载控制开关
      loading: false,

      //   模态框开关
      dialogFormVisible: false,
      dialogStatus: "",
      // 表单内容
      temp: {
        id: undefined,
        title: "",
        cover: "",
        content: "",
        price: 0,
        status: 1,
      },
    };
  },
  mounted() {
    this.getTableData();
  },
  methods: {
    getTableData() {
      fetchList(this.listQuery).then((response) => {
        console.log(response);
        this.tableData = response.data.items;
        this.listQuery.total = response.data.total;
      });
    },
    // 查询数据
    handQuery() {
      this.listQuery.page=1
      this.getTableData();
    },
    // 改变上架下架状态
     handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = status;
    },
    // 页码改变
    handleCurrentChange(val) {
      //   console.log(val);
      this.listQuery.page = val;
      this.getTableData();
    },
    // 每页显示数据多少
    handleSizeChange(val) {
      //   console.log(val);
      this.listQuery.limit = val;
      this.getTableData();
    },
    // 模态框跳出开关
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },

    resetTemp() {
      this.temp = {
        id: undefined,
        title: "",
        status: 1,
        price: 0,
        content: "",
        cover: "",
      };
    },
    // 增加数据
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
          this.temp.author = "vue-element-admin";
          createMedia(this.temp).then(() => {
            this.tableData.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Created Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },

    // 编辑数据
    handUpdata(row) {
      this.temp = Object.assign({}, row);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
    },
    updateData() {
      const tempData = Object.assign({}, this.temp);
      updateMedia(tempData).then(() => {
        const index = this.tableData.findIndex((v) => v.id == this.temp.id);
        this.tableData.splice(index, 1, this.temp);
        this.dialogFormVisible = false;
        this.$notify({
          title: "Success",
          message: "Update Successfully",
          type: "success",
          duration: 2000,
        });
      });
    },

    //删除数据
    handDelete(row, index) {
      deleteMedia(row).then((response) => {
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success",
          duration: 2000,
        });
        this.tableData.splice(index, 1);
      });
    },

    // 上传组件的钩子函数
    handleUploadRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleUploadSuccess(response, file, fileList) {
      console.log(response, file, fileList);
    },
  },

  computed: {
    filterTable() {
      if (this.value == "") {
        return this.tableData;
      } else if (this.value == "选项1") {
        return this.tableData.filter((item) => item.status == 1);
      } else if (this.value == "选项2") {
        return this.tableData.filter((item) => item.status == 0);
      }
    },
  },
};
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>