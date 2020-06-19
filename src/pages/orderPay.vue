<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div class="order-pay">
    <order-header title="订单支付">
      <template v-slot:tip>
        <span>请谨防钓鱼链接或诈骗电话，了解更多</span>
      </template>
    </order-header>
    <div class="wrapper">
      <div class="container">
        <div class="order-wrap">
          <div class="item-order">
            <div class="icon-succ"></div>
            <div class="order-info">
              <h2>订单提交成功！去付款咯～</h2>
              <p>请在<span>30分</span>内完成支付, 超时后将取消订单</p>
              <p>收货信息：{{addressInfo}}</p>
            </div>
            <div class="order-total">
              <p>应付总额：<span>{{payment}}</span>元</p>
              <p>订单详情<em class="icon-down" :class="{'up':showDetail}" @click="showDetail=!showDetail"></em></p>
            </div>
          </div>
          <div class="item-detail" v-if="showDetail">
            <div class="item">
              <div class="detail-title">订单号：</div>
              <div class="detail-info theme-color">{{orderId}}</div>
            </div>
            <div class="item">
              <div class="detail-title">收货信息：</div>
              <div class="detail-info">{{addressInfo}}</div>
            </div>
            <div class="item good">
              <div class="detail-title">商品名称：</div>
              <div class="detail-info">
                <ul>
                  <li v-for="(item,index) in orderDetail" :key="index">
                    <img v-lazy="item.productImage"/>{{item.productName}}
                  </li>
                </ul>
              </div>
            </div>
            <div class="item">
              <div class="detail-title">发票信息：</div>
              <div class="detail-info">电子发票 个人</div>
            </div>
          </div>
        </div>
        <div class="item-pay">
          <h3>选择以下支付方式付款</h3>
          <div class="pay-way" >
            <p>支付平台</p>
            <div class="pay pay-ali" :class="{'checked':payType==='ALIPAY_PC'}" @click="paySubmit(1)"></div>
            <div class="pay pay-wechat"  :class="{'checked':payType==='WXPAY_NATIVE'}" @click="paySubmit(2)"></div>

            <!--<div id="qrcode" style="margin-left: 220px"></div>-->
          </div>
        </div>
      </div>
    </div>
    <scan-pay-code v-if="showPay" @close="closePayModal" :img="payImg"></scan-pay-code>
    <modal
      title="支付确认"
      btnType="3"
      :showModal="showPayModal"
      sureText="查看订单"
      cancelText="未支付"
      @cancel="showPayModal=false"
      @submit="goOrderList"
    >
      <template v-slot:body="">
        <p>您确认是否完成支付？</p>
      </template>
    </modal>
  </div>

</template>
<script>
import QRCode from 'qrcode'
import OrderHeader from './../components/OrderHeader'
import ScanPayCode from './../components/ScanPayCode'
import Modal from './../components/Modal'
import axios from "axios"
// import QRCode from 'qrcodejs2'
export default{
  name:'order-pay',
  data(){
    return {
      orderId:this.$route.query.orderNo,
      addressInfo:'',//收货人地址
      orderDetail:[],//订单详情，包含商品列表
      showDetail:false,//是否显示订单详情
      payType:'',//支付类型
      showPay:false,//是否显示微信支付弹框
      payImg:'',//微信支付的二维码地址
      showPayModal:false,//是否显示二次支付确认弹框
      payment:0,//订单总金额
      T:'',//定时器ID
      // link: ''
    }
  },
  components:{
    OrderHeader,
    ScanPayCode,
    Modal
  },
  mounted(){
    this.getOrderDetail();
  },
  methods:{
    getOrderDetail(){
      this.axios.get(`/orders/${this.orderId}`).then((res)=>{
        let item = res.shippingVo;
        this.addressInfo = `${item.receiverName} ${item.receiverMobile} ${item.receiverProvince} ${item.receiverCity} ${item.receiverDistrict} ${item.receiverAddress}`;
        this.orderDetail = res.orderItemVoList;
        this.payment = res.payment;
      })
    },
    paySubmit(payType){
      var slef=this
      if(payType == 1){
         window.open('/#/order/alipay?orderId='+this.orderId,'_blank');
      }else{
        // axios.get('http://172.16.217.129:8081/pay?orderId='+this.orderId+'&amount=0.01&payType=WXPAY_NATIVE')
        axios.get('http://127.0.0.1:8081/pay?orderId='+this.orderId+'&amount=0.01&payType=WXPAY_NATIVE')
                .then(function (response) {
                  //成功
                  // console.log(response.codeUrl)
                  // slef.link=response.codeUrl
                  // new QRCode('qrcode', {
                  //   width: 132,
                  //   height: 132,
                  //   text: slef.link, // 二维码地址
                  // })

                  //原版
                  QRCode.toDataURL(response.codeUrl)
                          .then(url => {
                            slef.showPay = true;
                            slef.payImg = url;
                            slef.loopOrderState();
                          })
                          .catch(() => {
                            slef.$message.error('微信二维码生成失败，请稍后重试');
                          })

                }).catch(function (error) {
                  console.log(error.codeUrl)
                  // console.log(error);
                });
      }
    },
    // 关闭微信弹框
    closePayModal(){
      this.showPay = false;
      this.showPayModal = true;
      clearInterval(this.T);
    },
    // 轮询当前订单支付状态
    loopOrderState(){
      this.T = setInterval(()=>{
        this.axios.get(`/orders/${this.orderId}`).then((res)=>{
          if(res.status == 20){
            clearInterval(this.T);
            this.goOrderList();
          }
        })
      },1000);
    },
    goOrderList(){
      this.$router.push('/order/list');
    }
  }
}
</script>
<style lang="scss">
  .order-pay{
    .wrapper{
      background-color:#F5F5F5;
      padding-top:30px;
      padding-bottom:61px;
      .order-wrap{
        padding: 62px 50px;
        background-color: #fff;
        font-size:14px;
        margin-bottom:30px;
        .item-order{
          display: flex;
          align-items: center;
          .icon-succ{
            width: 90px;
            height: 90px;
            border-radius: 50%;
            background:url('/imgs/icon-gou.png') #80c58a no-repeat center;
            background-size:60px;
            margin-right:40px;
          }
          .order-info{
            margin-right: 248px;
            h2{
              font-size:24px;
              color:#333333;
              margin-bottom:20px;
            }
            p{
              color:#666666;
              span{
                color:#FF6700;
              }
            }
          }
          .order-total{
            &>p:first-child{
              margin-bottom:30px;
            }
            span{
              font-size:28px;
              color:#FF6700;
            }
            .icon-down{
              display:inline-block;
              width:14px;
              height:10px;
              background:url('/imgs/icon-down.png') no-repeat center;
              background-size:contain;
              margin-left:9px;
              transition:all .5s;
              cursor:pointer;
              &.up{
                transform: rotate(180deg);
              }
            }
            .icon-up{
              transform: rotate(180deg);
            }
          }
        }
        .item-detail{
          border-top: 1px solid #D7D7D7;
          padding-top: 47px;
          padding-left: 130px;
          font-size: 14px;
          margin-top: 45px;
          .item{
            margin-bottom:19px;
            .detail-title{
              float:left;
              width:100px;
            }
            .detail-info{
              display:inline-block;
              img{
                width: 30px;
                vertical-align: middle;
              }
            }
          }
        }
      }
      .item-pay{
        padding:26px 50px 72px;
        background-color:#ffffff;
        color: #333333;
        h3{
          font-size: 20px;
          font-weight: 200;
          color: #333333;
          padding-bottom: 24px;
          border-bottom: 1px solid #D7D7D7;
          margin-bottom: 26px;
        }
        .pay-way{
          font-size:18px;
          .pay{
            display:inline-block;
            width:188px;
            height:64px;
            border:1px solid #D7D7D7;
            cursor:pointer;
            &:last-child{
              margin-left:20px;
            }
            &.checked{
              border:1px solid #FF6700;
            }
          }
          .pay-ali{
            background:url('/imgs/pay/icon-ali.png') no-repeat center;
            background-size:103px 38px;
            margin-top:19px;
          }
          .pay-wechat{
            background:url('/imgs/pay/icon-wechat.png') no-repeat center;
            background-size:103px 38px;
          }
        }
      }
    }
  }
</style>
