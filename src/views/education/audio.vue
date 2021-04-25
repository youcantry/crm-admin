<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="input"
        placeholder="请输入你要查询的图书"
        style="width=100px"
      />
      <el-select v-model="value" placeholder="请选择">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        >
        </el-option>
      </el-select>
      <el-button>查询</el-button>
      <el-button>增加</el-button>
      <el-button>导出文件</el-button>
    </div>
    <el-table :data="tableList" style="width: 100%"
    :height="300">
      <el-table-column
        type="index"
        :index="indexMethod"
        label="ID"
        width="80px"
      >
      </el-table-column>
      <el-table-column prop="title" label="图书名称" width="280px">
       
      </el-table-column>
      <el-table-column prop="price" label="价格" width="120px">
      </el-table-column>
      <el-table-column prop="status" label="状态" width="120px">
        <template slot-scope="scope">
          {{ scope.row.status ? "已发布" : "未发布" }}
        </template>
      </el-table-column>
      <el-table-column prop="updated_time" label="创建时间" width="150px">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button
          >
        </template> </el-table-column
      >>
    </el-table>
    <div class="block">
  <el-pagination
    layout="prev, pager, next"
    :total="50">
  </el-pagination>
</div>
  </div>
</template>

<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
export default {
  name: "media",
  data() {
    return {
      options: [
        {
          value: "选项1",
          label: "已发布",
        },
        {
          value: "选项2",
          label: "未发布",
        },
      ],
      // 表格的数据源
      tableList: [],
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      fetchList().then((response) => {
        this.tableList = response.data.items;
      });
    },
  },
};
</script>