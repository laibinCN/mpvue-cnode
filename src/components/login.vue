<template>
<div class='mask' v-if='visible'>
  <div class="container">
    <input class='input' type="text" v-model='accesstoken' placeholder='请输入accesstoken'>
    <button class='button' :disabled='!accesstoken.length' @click.stop="clickLogin">登录</button>
    <button class='button' @click.stop="scale" style='margin-top: 20rpx;'>扫码登陆</button>
    <div class='tips'>登录网页版cnode后在设置中查看accesstoken或二维码</div>
    <button open-type="getUserInfo" @getuserinfo="bindGetUserInfo" @click="getUserInfo1">获取权限</button>
  </div>
</div>

</template>
<script>
import { api } from "../const";
export default {
  props: {
    visible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      accesstoken: ""
    };
  },
  methods: {
    async login(accesstoken) {
      const res = await this.$http.post(`${api}/accesstoken`, {
        accesstoken
      });
      if (res.data.success) {
        wx.setStorageSync("me", res.data.loginname);
        wx.setStorageSync("accesstoken", accesstoken);
        // 触发关闭
        this.$emit("modalClose");
      } else {
        wx.showToast({
          title: "accesstoken错误",
          icon: "none",
          duration: 2000
        });
      }
    },
    clickLogin(){
      this.login(this.accesstoken);
    },
    scale() {
      wx.scanCode({
        success: async res => {

          this.login(res.result);
        }
      });
    },
    mounted(){
      // 一进来看看用户是否授权过
      this.getSetting()
    },
    getSetting(){
      const self = this;
      wx.getSetting({
        success: function(res){
          if (res.authSetting['scope.userInfo']) {
            wx.getUserInfo({
              success: function(res) {
                console.log(res.userInfo)
                //用户已经授权过
                console.log('用户已经授权过')
                wx.setStorageSync("me", res.userInfo.nickName);
                wx.setStorageSync("accesstoken", res.userInfo);
                // 触发关闭
                self.$emit("modalClose");

              }
            })
          }else{
            console.log('用户还未授权过')
          }
        }
      })

    },
    getUserInfo1(){
      console.log('click事件首先触发')
      // 判断小程序的API，回调，参数，组件等是否在当前版本可用。  为false 提醒用户升级微信版本
      // console.log(wx.canIUse('button.open-type.getUserInfo'))
      if(wx.canIUse('button.open-type.getUserInfo')){
        // 用户版本可用
      }else{
        console.log('请升级微信版本')
      }
    },
    bindGetUserInfo(e) {
      // console.log(e.mp.detail.rawData)
      if (e.mp.detail.rawData){
        //用户按了允许授权按钮
        console.log('用户按了允许授权按钮')
        this.getSetting();
      } else {
        //用户按了拒绝按钮
        console.log('用户按了拒绝按钮')
      }
    },
  }
};
</script>
<style lang='scss' scoped>
.mask {
  position: fixed;
  background-color: white;
  height: 100vh;
  width: 100vw;
  z-index: 2333;
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    .input {
      border-bottom: 2rpx solid $borderColor;
      width: 600rpx;
      margin-top: 200rpx;
    }
    .button {
      width: 600rpx;
      margin-top: 150rpx;
    }
    .tips {
      font-size: 22rpx;
      color: #888;
      margin-top: 10rpx;
    }
  }
}
</style>
