<template>
  <div class="player">
    <div class="app-container">
      <h1>选手管理</h1>
      <p>
          {{act.title}}
      </p>
      
      <p>
          {{step.title}}
      </p>
      
      
    </div>

  </div>
</template>
<script>
  export default {
    name: 'Player',
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
        }
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
        },
        getAct(){
            this.$get("/activate/" + this.$route.query.actid).then(response=>{
                console.log("获取活动对象",response);
                this.act = response.data.data;
            }).catch(err=>{

            }).finally(()=>{

            })
        },

        getStep(){
            let params = {
                id:this.$route.query.actid
            }
            this.$postQs("/step/currentStep",params).then(response=>{
                console.log("/step/currentStep" + params,response);
                if(response.data.success){
                    this.step = response.data.data;
                    if(0 == (response.data.data.strone)){
                        console.log("活动未开始，跳转到首页");
                        // this.$message.error("活动未开始")
                    }else if(-1 == parseInt(response.data.data.strone)){
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
