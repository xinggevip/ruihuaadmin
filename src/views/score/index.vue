<template>
  <div class="score">
    <div class="app-container">
      <h2>{{act.title}}-成绩查询</h2>
      <!-- <p>
          {{step.title}}
      </p>
      <div v-for="(item,index) in stepList" :key="index">
        {{item.title}}
      </div> -->
        <div class="search">
              <el-select 
                    @change="valueChange"
                    v-model="companyValue" 
                    multiple
                    collapse-tags
                    style="margin-bottom: 10px;"
                    placeholder="请选择单位">
                    <el-option
                    v-for="item in companyOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                    </el-option>
                </el-select>

                <el-select 
                    @change="valueChange"
                    v-model="depValue" 
                    multiple
                    collapse-tags
                    style="margin-left: 20px;margin-bottom: 10px;"
                    placeholder="请选择部门">
                    <el-option
                    v-for="item in depOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                    </el-option>
                </el-select>

                <el-select 
                    @change="valueChange"
                    v-model="stepValue" 
                    multiple
                    collapse-tags
                    style="margin-left: 20px;margin-bottom: 10px;"
                    placeholder="请选择环节">
                    <el-option
                    v-for="item in stepOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                    </el-option>
                </el-select>

                <el-input @keyup.enter.native ="getScoreListPro()" v-model="page.playername" placeholder="请输入选手姓名" size="medium" style="width:180px;margin-left:20px;"></el-input>
                <el-input @keyup.enter.native ="getScoreListPro()" v-model="page.judgename" placeholder="请输入评委姓名" size="medium" style="width:180px;margin-left:20px;"></el-input>
                <el-button type="primary" icon="el-icon-search" size="medium" style=";margin-left:20px;" @click="getScoreListPro()">搜索</el-button>

        </div>

        <el-table
            v-loading="listLoading"
            :data="scoreList"
            element-loading-text="Loading"
            border
            fit
            highlight-current-row
        >
            <el-table-column show-overflow-tooltip align="center" label="单位" width="200">
                <template slot-scope="scope">
                {{ scope.row.company }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="部门" width="200">
                <template slot-scope="scope">
                {{ scope.row.dep }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="选手" width="200">
                <template slot-scope="scope">
                {{ scope.row.player_name }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="环节" width="200">
                <template slot-scope="scope">
                {{ scope.row.step_title }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="平均得分" width="200">
                <template slot-scope="scope">
                {{ scope.row.avg_score_sum }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="合计得分" width="200">
                <template slot-scope="scope">
                {{ scope.row.sum_score }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="得分详情" width="200">
                <template slot-scope="scope">
                {{ scope.row.judge_name }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip align="center" label="评委人数" width="200">
                <template slot-scope="scope">
                {{ scope.row.judge_num }}
                </template>
            </el-table-column>
        </el-table>







      
      
    </div>

  </div>
</template>
<script>
  export default {
    name: 'Score',
    components: {

    },
    props: {

    },
    data(){
      return {
        loading:false,
        act:{
            title:"加载中"
        },
        step:{
            title:"加载中"
        },
        stepList:[],
        scoreList:[],
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
          actid: this.$route.query.actid,
          page: 1,
          pageCount: 10,
          company:'',
          dep:'',
          stepid:null,
          playername:'',
          judgename:''

        },
        user: JSON.parse(this.$store.state.user),
        loading:false,
        dialogWidth: "0px", // 屏幕宽度
        companyOptions: [],
        companyValue: [],
        depOptions: [],
        depValue: [],
        stepOptions:[],
        stepValue:[]

      }
    },
    watch:{

    },
    created(){
        this.fetchData();
        // 设置对话框宽度
        this.setDialogWidth();
    },
    methods:{
        fetchData(){
            this.getStep();
            this.getAct();
            this.getStepList();
            this.getScoreList();
            this.getCompanysAndDepsByActid();
        },
        // 获取该活动下所有人的去重后的单位属性
        getCompanysAndDepsByActid(){
            this.$get("/player/getCompanysAndDepsByActid/"+this.$route.query.actid).then(response=>{
                let companys = response.data.data[0];
                let deps = response.data.data[1];
                this.companyOptions = companys.map((item,index,arr)=>{
                    let json = {
                        value:item,
                        label:item
                    }
                    return json;
                })
                this.depOptions = deps.map((item,index,arr)=>{
                    let json = {
                        value:item,
                        label:item
                    }
                    return json;
                })
            }).catch(err=>{

            }).finally(()=>{

            })
        },
        valueChange(){
            console.log("stepid",this.stepValue);
            this.getScoreListPro();
        },
        getScoreListPro(){
            let companyStr = this.arrToStr(this.companyValue);
            let depStr = this.arrToStr(this.depValue);
            let stepidStr = this.arrToStr(this.stepValue);
            console.log(companyStr);
            console.log(depStr);
            this.page.company = companyStr;
            this.page.dep = depStr;
            this.page.stepid = stepidStr;
            this.getScoreList();
        },

        arrToStr(paramsArr){
            let [...arr] = paramsArr;
            let strRes = "";
            for(let i = 0;i < arr.length;i++){
                if(i === arr.length - 1){
                    strRes = strRes + "'" + arr[i] + "'"
                }else{
                    strRes = strRes + "'" + arr[i] + "',"
                }
            }
            return strRes;
        },

        getAct(){
            this.$get("/activate/" + this.$route.query.actid).then(response=>{
                console.log("获取活动对象",response);
                this.act = response.data.data;
            }).catch(err=>{

            }).finally(()=>{

            })
        },
        getStepList(){
            this.loading = true;
            let params = {
                actid:this.$route.query.actid
            }
            this.$postQs("/step/steplist",params).then(response=>{
                console.log(response);
                if(response.data.success != true){
                return ;
                }
                let list = response.data.data;
                this.stepOptions = list.map((item,index,arr)=>{
                    let json = {
                        value:item.id,
                        label:item.title
                    }
                    return json;
                })
            }).catch(err=>{
            }).finally(()=>{
                this.loading = false;
            })
        },
        getStep(){
            let params = {
                id:this.$route.query.actid
            }
            this.$postQs("/step/currentStep",params).then(response=>{
                console.log(response);
                if(response.data.success){
                    this.step = response.data.data;
                    if(response.data.data.id === 0){
                        console.log("活动未开始，跳转到首页");
                        // this.$message.error("活动未开始")
                    }
                    if(response.data.data.id === -1){
                        console.log("活动已结束，跳转到成绩页面");
                        // this.$message.error("活动已结束")
                    }
                }
            }).catch(err=>{
            }).finally(()=>{
            })
        },
        getScoreList(){
            this.listLoading = true;
            this.$post("/scorevalue/getplayerscore",this.page).then(response=>{
                console.log("/getplayerscore",response);
                this.scoreList = response.data.data;
            }).catch(err=>{

            }).finally(()=>{
                this.listLoading = false;
            })
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
            alert("执行删除");
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
    },
    mounted(){
        window.onresize = () => {
            return (() => {
            this.setDialogWidth()
            })()
        }
    }
  }
</script>
<style lang="scss" scoped>

</style>
