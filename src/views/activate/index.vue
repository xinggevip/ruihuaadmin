<template>
  <div class="app-container">
    <!-- 搜索筛选 -->
    <div class="search">
      <el-input @keyup.enter.native ="fetchData()" v-model="page.value" placeholder="请输入活动名称" size="medium" style="width:250px;margin-right:15px;float:left"></el-input>
      <el-button type="primary" icon="el-icon-search" size="medium" style="margin-right:15px;float:left" @click="fetchData()">搜索</el-button>
    </div>


    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column show-overflow-tooltip align="center" label="ID" width="70">
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="活动名称" width="200">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="活动介绍" width="400">
        <template slot-scope="scope">
          {{ scope.row.profile }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="创建时间" width="180">
        <template slot-scope="scope">
          {{ dateFormatter(scope.row.pushDate) }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="开始时间" width="180">
        <template slot-scope="scope">
          {{ dateFormatter(scope.row.startDate) }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="状态" width="80">
        <template slot-scope="scope">
          {{ statusFormatter(scope.row.strone) }}
        </template>
      </el-table-column>
      <!-- <el-table-column label="操作" min-width="350"  fixed="right" > -->
      <el-table-column label="操作" min-width="350"  >
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            @click="toPlayer(scope.row)"
          >选手管理</el-button>

          <el-button
            v-if="-1 == parseInt(scope.row.strone) || parseInt(scope.row.strone) > 0"
            size="mini"
            type="success"
            @click="toScore(scope.row)"
          >
            成绩查询
          </el-button>

          <el-button
            disabled
            v-if="parseInt(scope.row.strone) == 0"
            size="mini"
            type="success"
            @click="toScore(scope.row)"
          >
           成绩查询
          </el-button>

          <el-button
            size="mini"
            type="warning"
          >首页隐藏</el-button>
          <el-button
            size="mini"
            type="danger"
          >删除</el-button>


        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="block" style="float:right;margin-top:23px;">
      <!-- <span class="demonstration">完整功能</span> -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page.page"
        :page-sizes="pageSetting['page-sizes']"
        :page-size="pageSetting['page-size']"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pageSetting['total']">
      </el-pagination>
    </div>



  </div>
</template>

<script>
  import { Message } from 'element-ui'
  // import { getList } from '@/api/table'

  export default {
    name: 'Activate',
    data() {
      return {
        TimeSubStr:'',
        list: null,
        listLoading: false,
        // 分页配置
        pageSetting:{
          'page-sizes':[5, 10, 15, 20],
          'page-size':10,
          'total':0
        },
        page: {
          page: 1,
          pageCount: 10,
          value: '',
          status: '', // 进行中  已结束
          type:6
        },
        user: JSON.parse(this.$store.state.user),
        loading:false,
        dialogWidth: "0px", // 屏幕宽度

      }
    },
    created() {
      this.fetchData()
      // 设置对话框宽度
      this.setDialogWidth();

    },
    mounted() {
      window.onresize = () => {
        return (() => {
          this.setDialogWidth()
        })()
      }
    },
    methods: {
      fetchData() {
        this.getActList();
      },
      getActList(){
        this.listLoading = true
        this.$post('/activate/getActList', this.page).then((response) => {
          console.log('/activate/getActList' + this.page,response);
          this.list = response.data.data.records;
          this.pageSetting.total = response.data.data.total;
          // console.log(result);
        }).catch((err) => {

        }).finally(() => this.listLoading = false);
      },
      // 设置对话框大小
      setDialogWidth() {
        // console.log(document.body.clientWidth)
        var val = document.body.clientWidth
        const def = 800 // 默认宽度
        if (val < def) {
          this.dialogWidth = '100%'
        } else {
          this.dialogWidth = def + 'px'
        }
      },
      // 设置每页显示多少条
      handleSizeChange(val) {
        // console.log(`每页 ${val} 条`);
        this.page.pageCount = val;
        this.page.page = 1;
        this.fetchData();
      },
      // 页数发生改变
      handleCurrentChange(val) {
        // console.log(`当前页: ${val}`);
        this.page.page = val;
        this.fetchData();
      },
      // 删除
      handleDel(index,row) {
        this.$confirm('此操作将放永久删除这条记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.delete(index, row);

        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });
        });
      },
      delete(index, row) {
        this.$del('/apt/' + row.id).then((result) => {
          if (result.message !== null) {
            Message.success(result.message);
          }
          this.fetchData();
        }).catch((error) => {

        }).finally();
      },
       // 日期格式化
      dateFormatter (value, column) {
        // console.log(row.savedate);
        // let datetime = row.starttime;
        if(value){
          return this.$dateUtil.mToDateStr(value, 'yyyy-MM-dd hh:mm')
        }
        return "空日期"
      },
      // 活动状态格式化
      statusFormatter(value,column){
        let res = parseInt(value);
        if(-1 === res){
          return "已结束"
        }else if(0 === res){ 
          return "未开始"
        }else if(res > 0){
          return "进行中"
        }else{
          return "配置异常，" + value; 
        }
      },
      toPlayer(row){
        console.log(row);
        this.$router.push({path:'/active/player',query: {actid:row.id}});
      },
      toScore(row){
        console.log(row);
        this.$router.push({path:'/active/score',query: {actid:row.id}});
      }





    }
  }
</script>
<style rel="stylesheet/scss" lang="scss">
  .search::after,.search::before{
    content: '';
    display: table;
    clear: both
  }
  .search{
    padding-bottom: 15px;
  }
</style>
