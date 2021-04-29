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
      <el-table-column show-overflow-tooltip align="center" label="ID" width="95">
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="名称" width="250">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="介绍" width="400">
        <template slot-scope="scope">
          {{ scope.row.profile }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="创建时间" width="200">
        <template slot-scope="scope">
          {{ scope.row.pushDate }}
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip align="center" label="开始时间" width="200">
        <template slot-scope="scope">
          {{ scope.row.startDate }}
        </template>
      </el-table-column>
      
      <el-table-column label="操作" min-width="290"  fixed="right" >
        <template>
          <el-button
            size="mini"
            type="primary"
          >随机上台</el-button>
          <el-button
            size="mini"
            type="success"
          >查看成绩</el-button>
          <el-button
            size="mini"
            type="warning"
          >隐藏</el-button>
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
          status: '进行中',
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
          console.log(response);
          this.list = response.data.data.records;
          this.pageSetting.total = result.data.data.total;
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
