<template>
  <div class="ali-pay">
    <loading v-if="loading"></loading>
    <div class="form" v-html="this.content"></div>
  </div>
</template>
<script>
  import Loading from './../components/Loading'
  import axios from "axios"
  export default{
    name:'alipay',
    components:{
      Loading
    },
    data(){
      return {
        orderId: this.$route.query.orderId,
        content: '',
        loading: true
      }
    },
    mounted(){
      this.paySubmit();
    },
    methods:{
      paySubmit(){
       var self=this
        // axios.get('http://172.16.217.129:8081/pay?orderId='+this.orderId+'&amount=0.01&payType=ALIPAY_PC')
        axios.get('http://127.0.0.1:8081/pay?orderId='+this.orderId+'&amount=0.01&payType=ALIPAY_PC')
                .then(function (response) {
                  self.content=response.body;
                  setTimeout(()=>{
                    document.forms[0].submit();
                  },100)
                });
      }
    }
  }
</script>
