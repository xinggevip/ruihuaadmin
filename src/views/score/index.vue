<template>
  <div class="score">
    <div class="app-container">
      <h1>成绩查询</h1>
      <p>
          {{act.title}}
      </p>
      
      <p>
          {{step.title}}
      </p>
      <div v-for="(item,index) in stepList" :key="index">
        {{item.title}}
      </div>
      
      
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
        stepList:[]
      }
    },
    watch:{

    },
    created(){
        this.fetchData();
    },
    methods:{
        fetchData(){
            this.getStep();
            this.getAct();
            this.getStepList();
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
                let neweList = list.map((item,index,arr)=>{
                item.name = item.title
                return item;
                });
                this.stepList = neweList;
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
    },
    mounted(){
        this.timer = setInterval(() => {
        // 某些操作
        // console.log("访问活动版本");
        this.$get("/activate/" + this.$route.query.actid).then(response=>{
            console.log("活动版本",response.data.data.numone);
            if(this.localActVersion != response.data.data.numone){
                console.log("版本不一致");
                this.localActVersion = response.data.data.numone;
                this.fetchData();
            }
            }).catch(err=>{
            }).finally(()=>{
            })
        }, 5000)
    },
    beforeDestroy(){
        this.$once('hook:beforeDestroy', () => {            
            clearInterval(this.timer);                                    
        })
    }
  }
</script>
<style lang="scss" scoped>

</style>
