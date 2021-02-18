<template>
  <view class="container">
    <view class="wrapper">
      <view class="left-top-sign">GMU</view>
      <view class="welcome"> 欢迎登录 </view>
      <view class="input-content">
        <view class="input-item">
          <image src="/static/login/user.png" />
          <input
            type="text"
            placeholder="请输入学号"
            v-model="username"
            @input="handleInUser"
          />
        </view>
        <view class="input-item">
          <image src="/static/login/psd.png" />
          <input
            type="password"
            placeholder="请输入密码"
            v-model="password"
            @input="handleInPwd"
          />
        </view>
      </view>
      <button
        :class="['confirm-btn', (username && password) ? 'btn-hover' : 'btn-default']"
        @click="login"
      >
        登录
      </button>
      <view class="forget-section"> 忘记密码? </view>
    </view>
    <!-- 底部注册区域 -->
    <view class="register-section">
      还没有账号?
      <text>马上注册</text>
    </view>
  </view>
</template>

<script>
import request from "../../utils/request";
export default {
  components: {},
  data() {
    return {
      username: "stucadres",
      password: "123456",
    };
  },
  onLoad() {
    console.log(getApp().globalData.mssage);
  },
  methods: {
    //表单项内容发生改变
    handleInUser(e) {
      this.username = e.detail.value
    },
    handleInPwd(e) {
      this.password = e.detail.value
    },
    //点击登录
    async login() {
      // 1. 收集表单项数据
      //前端验证
      if (!this.username) {
        //提示用户
        uni.showToast({
          title: "学号不能为空",
          icon: "none",
        });
        return;
      }
      if (!this.password) {
        //提示用户
        uni.showToast({
          title: "密码不能为空",
          icon: "none",
        });
        return;
      }
      //后端验证
      let result = await request("/login/login", { username:this.username, password:this.password });
      if (result.data.code === "200") {
        uni.showToast({
          title: "登录成功",
        });
        uni.setStorageSync("JSESSIONID", result.cookies[0]);
        const users = result.data.data.role;
        //登录成功后跳转至选择寝室列表界面,判断角色
        if (users == "ROLE_student") {
          uni.navigateTo({
            url: "/pages/checkedlist/index?username=" + username,
          });
        } else {
          uni.navigateTo({
            url: "/pages/dormlist/index",
          });
        }
      } else if (result.data.code === 404) {
        uni.showToast({
          title: "用户名或密码错误",
          icon: "none",
        });
      } else {
        uni.showToast({
          title: "登录失败，请重新登录",
          icon: "none",
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  height: 100%;
  padding: 200rpx 0;
  box-sizing: border-box;
  .wrapper {
    position: relative;
    z-index: 90;
    padding-bottom: 40rpx;
    .left-top-sign {
      font-size: 120rpx;
      color: #f8f8f8;
      position: relative;
      left: 0;
    }
    .welcome {
      position: relative;
      left: 50rpx;
      top: -90rpx;
      font-size: 46rpx;
      color: #555;
    }
    .input-content {
      padding: 0 60rpx;
      .input-item {
        display: flex;
        flex-direction: row;
        padding: 0 30rpx;
        background: #f8f6fc;
        height: 80rpx;
        line-height: 80rpx;
        border-radius: 10rpx;
        margin-bottom: 50rpx;
        image {
          width: 50rpx;
          height: 50rpx;
          margin-top: 15rpx;
        }
        input {
          flex: 1;
          height: 80rpx;
          line-height: 80rpx;
          font-size: 30rpx;
          color: #303133;
          width: 100%;
        }
        &:last-child {
          margin-bottom: 0;
        }
      }
    }
    .confirm-btn {
      width: 630rpx !important;
      height: 76rpx;
      line-height: 76rpx;
      border-radius: 50rpx;
      margin-top: 70rpx;
      color: #fff;
      font-size: 32rpx;
      padding: 0;
    }
    .forget-section {
      font-size: 28rpx;
      color: #4399fc;
      text-align: center;
      margin-top: 40rpx;
    }
  }
  .register-section {
    position: absolute;
    left: 0;
    bottom: 50rpx;
    width: 100%;
    font-size: 28rpx;
    color: #606266;
    text-align: center;
    text {
      color: #4399fc;
      margin-left: 10rpx;
    }
  }
  // 动态添加登录按钮的背景颜色
  .btn-default {
    background-color: #aaa;
  }
  .btn-hover {
    background-color: #00702b;
  }
}
</style>
