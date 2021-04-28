<template>
  <div>
    <!-- 头部 -->
    <el-row style="display: flex; justify-content: space-between">
      <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
        >新增音频</el-button
      >
      <el-select v-model="value" placeholder="请选择" clearable>
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
        placeholder="请输入内容"
        style="width: 120px"
      ></el-input>
      <el-button type="primary" icon="el-icon-search" @click="handleQuery"
        >搜索</el-button
      >
    </el-row>
    <!-- 表格 -->
    <el-row>
      <el-table
        :data="filterTable"
        border
        style="width: 100%"
        v-loading="loading"
      >
        <el-table-column fixed label="ID" width="100" prop="id">
        </el-table-column>
        <el-table-column label="音频内容" width="600">
          <template slot-scope="scope">
            <div style="display: flex; justify-content: flex-start">
              <img
                :src="scope.row.cover"
                alt=""
                style="width: 100px; height: 50px"
              />
              <div>
                <p>{{ scope.row.title }}</p>
                <p style="color: red">￥{{ scope.row.price }}</p>
              </div>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="订阅量" width="100" prop="price">
        </el-table-column>
        <el-table-column label="状态" width="100" prop="status">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status ? 'success' : 'danger'">
              {{ scope.row.status | statusFilter }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="时间" width="150" prop="created_time">
        </el-table-column>
        <el-table-column fixed="right" label="操作" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="small"
              @click="handleUpdate(scope.row)"
              >编辑</el-button
            >
            <el-button v-if="scope.row.status==0" type="primary" size="small" @click="handleStatus(scope.row,1)">上架</el-button>
            <el-button v-if="scope.row.status==1" type="danger" size="small" @click="handleStatus(scope.row,0)">下架</el-button>
            <el-button
              type="primary"
              size="small"
              @click="handleDelete(scope.row, scope.row.index)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-row>
    <!-- 分页操作 -->
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
        <el-form ref="form" :model="temp">
          <el-form-item label="标题">
            <el-input v-model="temp.title"></el-input>
          </el-form-item>
          <el-form-item label="封面">
            <el-upload
              class="avatar-uploader"
              action="https://jsonplaceholder.typicode.com/posts/"
              :show-file-list="false"
              :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload"
            >
              <img v-if="imageUrl" :src="imageUrl" class="avatar" />
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="课程介绍">
            <Tinymce v-model="temp.try"></Tinymce>
          </el-form-item>
          <el-form-item label="课程内容">
            <el-upload
              class="upload-demo"
              action="https://jsonplaceholder.typicode.com/posts/"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :before-remove="beforeRemove"
              multiple
              :limit="3"
              :on-exceed="handleExceed"
              :file-list="fileList"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">
                只能上传MP3、MP4文件，建议流畅,上传大小不超过500M
              </div>
            </el-upload>
          </el-form-item>
          <el-form-item label="课程价格">
            <el-input-number v-model="temp.price" :min="1"></el-input-number>
          </el-form-item>
          <el-form-item label="状态">
            <el-radio-group v-model="temp.status">
              <el-radio label="0">下架</el-radio>
              <el-radio label="1">上架</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
          >提交</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList, createAudio, updateAudio, deleteAudio } from "@/api/audio";
import Tinymce from "@/components/Tinymce";
const statusOptions = {
  0: "已下架",
  1: "已上架",
};

export default {
  components: { Tinymce },
  data() {
    return {
      // 上传图片组件的url
      imageUrl: "",
      // 上传课程内容
      fileList: [
        {
          name: "food.mp3",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100",
        },
        {
          name: "food2.MP4",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100",
        },
      ],
      statusOptions,
      // 模态框提示信息
      textMap: {
        create: "新增",
        update: "修改",
      },
      // 选择上架和下架的列表
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
      // 分页请求数据
      listQuery: {
        // 总数
        total: 0,
        // 一页显示多少数据
        limit: 20,
        // 第几页
        page: 1,
        title: "",
      },
      // 控制加载显示
      loading: false,

      //模态框
      dialogFormVisible: false,
      //  控制提交的状态
      dialogStatus: "",

      // 添加数据的模板
      temp: {
        id: "",
        title: "",
        cover: "",
        content: "",
        price: "",
        status: 1,
        try: "",
      },
    };
  },
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
  mounted() {
    this.getTableData();
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
  methods: {
    // 查询数据源
    async getTableData() {
      this.loading = true;
      // 请求数据
      let res = await fetchList(this.listQuery);
      // console.log(res);
      this.tableData = res.data.items;
      this.listQuery.total = res.data.total;

      this.loading = false;
    },
    // 当页码属改变触发
    handleSizeChange(val) {
      this.listQuery.limit = val;
      this.getTableData();
    },
    // 当页码发生改变
    handleCurrentChange(val) {
      this.listQuery.page = val;
      this.getTableData();
    },

    // 模态框的方法
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    resetTemp() {
      this.temp = {
        id: "",
        title: "",
        cover: "",
        content: "",
        price: "",
        status: 1,
        try: "",
      };
    },
    // 增加数据
    handleCreate() {
      this.resetTemp();
      this.dialogFormVisible = true;
      this.dialogStatus = "create";
    },
    createData() {
      this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
      this.temp.author = "vue-element-admin";
      createAudio(this.temp).then(() => {
        this.tableData.unshift(this.temp);
        this.dialogFormVisible = false;
        this.$notify({
          title: "Success",
          message: "Created Successfully",
          type: "success",
          duration: 2000,
        });
      });
    },

    // 编辑内容
    handleUpdate(row) {
      this.temp = Object.assign({}, row);
      (this.dialogStatus = "update"), (this.dialogFormVisible = true);
    },
    updateData() {
      const tempData = Object.assign({}, this.temp);
      updateAudio(tempData).then(() => {
        const index = this.tableData.findIndex(
          (item) => item.id == this.temp.id
        );
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

    // 删除数据
    handleDelete(row, index) {
      deleteAudio(row).then(() => {
        this.tableData.splice(index, 1);
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success",
          duration: 2000,
        });
      });
    },

    // 查询数据
    handleQuery() {
      (this.listQuery.page = 1), this.getTableData();
    },
    // 改变商品商家下架状态
    handleStatus(row,id){
      row.status=id
    },
    //上传课程内容组件的钩子函数
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${
          files.length + fileList.length
        } 个文件`
      );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    // 上传图片组件的钩子函数
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
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