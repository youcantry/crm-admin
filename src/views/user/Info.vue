<template>
  <div>
    <el-dialog title="用户信息" :visible.sync="dialogVisible" width="80%">
      <el-container>
        <!-- 头部 -->
        <el-header>
          <el-row>
            <el-col :span="6">
              <p>ID:{{ user.id }}</p></el-col
            >
            <el-col :span="6">
              <p>用户名:{{ user.username }}</p>
            </el-col>
            <el-col :span="6">
              <p>昵称:{{ user.nickname }}</p>
            </el-col>
            <el-col :span="6">
              <p>手机号:{{ user.phone }}</p>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="6">
              <p>锁定:{{ user.status ? "否" : "是" }}</p>
            </el-col>
            <el-col :span="6">
              <p>会员:{{ user.user_level }}</p>
            </el-col>
            <el-col :span="6">
              <p>会员到期时间{{ user.user_level_expire }}</p>
            </el-col>
            <el-col :span="6">
              <p>注册时间{{ user.created_time }}</p>
            </el-col>
          </el-row>
        </el-header>
        <!-- content -->
        <el-main>
          <el-tabs v-model="activeName" @tab-click="handleClick" v-loading='listLoading'>
            <el-tab-pane
              :key="index"
              :label="item.label"
              :name="item.name"
              v-for="(item, index) in tabs"
            >
              <el-table
                style="width: 100%"
                height="300"
                :data="list"
                v-loading="listLoading"
              >
                <el-table-column
                  :prop="t.prop"
                  :label="t.label"
                  v-for="(t, tI) in item.ths"
                  :key="tI"
                >
                </el-table-column>
              </el-table>
              <!-- 分页处理 -->
              <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :page-size="listQuery.limit"
                layout="total, sizes, prev, pager, next, jumper"
                :total="listQuery.total"
              >
              </el-pagination>
            </el-tab-pane>
          </el-tabs>
        </el-main>
      </el-container>
    </el-dialog>
  </div>
</template>
<script>
import {
  fetchUserDetail,
  fetchUserCourse,
  fetchUserColumn,
  fetchUserOrder,
  fetchUserHistory,
  fetchUserComment,
} from "@/api/user";
let statusOptions = {
    pendding:"待支付",
    success:"成功",
    fail:"失败"
}
let typeOptions = {
    media:"图文",
    audio:"音频",
    video:"视频",
    column:"专栏",
}
export default {
  data() {
    return {
      dialogVisible: false,
      id: null,
      user: {},
      //   tabs默认菜单首选项
      activeName: "course",
      // tabs菜单选项的数据加载显示符
      listLoading: false,
      // tabs菜单循环项
      tabs: [
        {
          label: "课程订阅",
          name: "course",
          ths: [
            {
              prop: "title",
              label: "课程标题",
            },
            {
              prop: "price",
              label: "购买价格",
            },
            {
              prop: "created_time",
              label: "购买时间",
            },
          ],
          request: fetchUserCourse,
        },
        {
          label: "专栏订阅",
          name: "column",
          ths: [
            {
              prop: "title",
              label: "专栏标题",
            },
            {
              prop: "price",
              label: "购买价格",
            },
            {
              prop: "created_time",
              label: "购买时间",
            },
          ],
          request: fetchUserColumn,
        },
        {
          label: "订单记录",
          name: "order",
          ths: [
            {
              prop: "id",
              label: "ID",
            },
            {
              prop: "no",
              label: "订单号",
            },
            {
              prop: "price",
              label: "购买价格",
            },
            {
              prop: "status",
              label: "状态",
            },
            {
              prop: "title",
              label: "商品",
            },
            {
              prop: "created_time",
              label: "购买时间",
            },
          ],
          request: fetchUserOrder,
        },
        {
          label: "观看历史",
          name: "history",
          ths: [
            {
              prop: "title",
              label: "课程标题",
            },
            {
              prop: "type",
              label: "课程类型",
            },
            {
              prop: "total_time",
              label: "学习时长",
            },
          ],
          request: fetchUserHistory,
        },
        {
          label: "用户评论",
          name: "comment",
          ths: [
            {
              prop: "content",
              label: "评论内容",
            },
            {
              prop: "created_time",
              label: "评论时间",
            },
            {
              prop: "title",
              label: "课程标题",
            },
            {
              prop: "type",
              label: "类型",
            },
          ],
          request: fetchUserComment,
        },
      ],
      // 表格的数据
      list: [],
        // 查询分页处理
        listQuery:{
            // 每页的大小
            limit:20,
            // 第几页
            page:1,
            // 总共多少数据
            total:0,
        }
    };
  },
  //   计算属性
  computed: {
    currentTab() {
      return this.tabs.find((item) => item.name == this.activeName);
    },
  },
  mounted() {
    this.getData();
  },
  methods: {
    //打开模态框
    open(id) {
      this.id = id;
      this.dialogVisible = true;
      //   console.log(id);
      fetchUserDetail({
        id,
      }).then((res) => {
        //   console.log(res);
        this.user = res.data.user;
      });
    },
    // 切换tabs标签页
    handleClick(tab, event) {
      // console.log(tab, event);
      this.getData();
    },
    //   请求tab表格的每行数据
    getData() {
    this.listLoading=true
      this.currentTab.request(this.listQuery).then((res) => {
        //    console.log(res);
        this.list = res.data.items.map(item=>{
            if(this.currentTab.name=='order'){
                item.status=statusOptions[item.status]
            }else if(this.currentTab.name=='history'||this.currentTab.name=='comment'){
                item.type=typeOptions[item.type]
            }
            return item
        });
        this.listQuery.total=res.data.total
        this.listLoading=false
      });
    },
    // 每页大小
     handleSizeChange(val) {
        // console.log(`每页 ${val} 条`);
        this.listQuery.limit=val
        this.getData()
      },
    //   第几页
      handleCurrentChange(val) {
        // console.log(`当前页: ${val}`);
        this.listQuery.page=val
        this.getData()
      }
  },
};
</script>