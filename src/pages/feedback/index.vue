<template>
  <view class="container">
    <view class="title">
      <text>寝室:{{ checkedInfo.dormNum }}</text>
      <text class="date">查寝时间:{{ checkedInfo.checkTime }}</text>
    </view>
    <view class="content">
      <!-- 查寝记录 -->
      <view class="checkInfo">
        <view class="checked_title">查寝描述:</view>
        <view class="des_content">
          {{ checkedInfo.qualifiedDescribe || checkedInfo.unqualifiedDescribe }}
        </view>
        <view class="checked_title"
          >查寝照片:
          <icon type="info" color="gray" size="12" />
          <text class="tip">点击可预览图片</text>
        </view>
        <view class="pic_content">
          <view
            class="checkedImg_item"
            v-for="item in urlList"
            :key="item"
            @click="PreviewCheckedImg"
          >
            <image v-if="urlList.length" :src="item" />
          </view>
        </view>
      </view>
      <!-- 反馈信息 -->
      <view class="feedbackInfo">
        <view class="fb_title">
          <text class="icon">*</text>
          反馈描述:
        </view>
        <view class="des_content">
          <textarea
            :maxlength="50"
            placeholder="请输入..."
            v-model="value"
          ></textarea>
        </view>
        <view class="fb_title">
          <text class="icon">*</text>
          反馈照片:
          <icon type="info" color="gray" size="12" />
          <text class="tip">长按可选择删除</text>
        </view>
        <view class="fb_imges">
          <button @click="handleAddImg">+</button>
          <view
            class="img_wrap"
            v-for="item in chooseImgs"
            :key="item"
            @click="handlePreviewImg"
            @longpress="longpressImg"
          >
            <upload-imgs v-if="uploadImgs.length" :src="item"></upload-imgs>
          </view>
        </view>
      </view>
      <view class="btn_wrap">
        <button type="primary" @click="commit">提交</button>
      </view>
    </view>
  </view>
</template>

<script>
import uniCard from "@dcloudio/uni-ui/lib/uni-card/uni-card.vue";
import UploadImgs from "../../components/UploadImgs/UploadImgs";
import request from "../../utils/request";
export default {
  components: {
    uniCard,
    UploadImgs
  },
  data() {
    return {
      stunum: "", //学生登录账号
      checkedInfo: {}, //当天查寝信息
      value:'', //反馈描述
      urlList: [], //查寝照片
      chooseImgs: [],
      uploadImgs: []
    };
  },
  //监听页面加载
  onLoad() {
    let _this = this;
    let id = null;
    // 监听acceptDataFromOpenerPage事件，获取上一页面通过eventChannel传送到当前页面的数据
    const eventChannel = this.getOpenerEventChannel();
    eventChannel.on("acceptDataFromOpenerPage", function(data) {
      (_this.stunum = data.username), (id = data.id);
    });
    this.getCheckList(id);
  },
  methods: {
    //获取查寝记录的方法
    async getCheckList(id) {
      let res = await request("/stuDorm", { stunum: this.stunum });
      let dormInfo = res.data.data2;
      console.log(dormInfo);

      dormInfo.some(item => {
        if (item.id == id) {
          this.checkedInfo = item;
          console.log(this.checkedInfo);
        }  
      });
      // 分割图片路径字符串
      let url =
        this.checkedInfo.qualifiedPicture ||
        this.checkedInfo.unqualifiedPicture;
      this.urlList = url.split(",");
    },
    //点击查寝图片预览大图
    PreviewCheckedImg(event) {
      const _this = this;
      let { url } = event.currentTarget.dataset;
      uni.previewImage({
        current: url, // 当前显示图片的http链接
        urls: _this.urlList // 需要预览的图片http链接列表
      });
    },
    //点击添加并上传图片
    handleAddImg() {
      if (this.uploadImgs.length > 2) {
        uni.showToast({
          title: "最多上传3张图片哦~",
          icon: "none",
          mask: true
        });
        return;
      }
      const _this = this;
      uni.chooseImage({
        count: 1,
        sizeType: ["original", "compressed"],
        sourceType: ["album", "camera"],
        success: res => {
          //获取本地存储的cookie
          let JSESSIONID = uni
            .getStorageSync("JSESSIONID")
            .split(";")[0]
            .split("=")[1];
          let cookie = "JSESSIONID=" + JSESSIONID;
          // tempFilePath可以作为img标签的src属性显示图片
          _this.chooseImgs = [..._this.chooseImgs, ...res.tempFilePaths];
          //上传图片
          const tempFilePaths = res.tempFilePaths;
          uni.uploadFile({
            url: "https://111.75.252.147/score/tupian",
            filePath: tempFilePaths[tempFilePaths.length - 1],
            name: "file",
            header: {
              "content-type": "multipart/form-data",
              cookie: cookie
            },
            success(res) {
              let url = JSON.parse(res.data).data;
              _this.uploadImgs.push(url);
            }
          });
        }
      });
    },
    //点击上传的图片预览大图
    handlePreviewImg(event) {
      const _this = this;
      let { path } = event.currentTarget.dataset;
      uni.previewImage({
        current: path, // 当前显示图片的http链接
        urls: _this.uploadImgs // 需要预览的图片http链接列表
      });
    },
    //长按图片选择是否删除
    longpressImg(event) {
      const _this = this;
      //获取当前点击图片下标
      let { index } = event.currentTarget.dataset;
      uni.showModal({
        title: "提示",
        content: "确认要删除该图片吗?",
        success: function(res) {
          if (res.confirm) {
            _this.chooseImgs.splice(index, 1);
            _this.uploadImgs.splice(index, 1);
          } else if (res.cancel) {
            return false;
          }
        }
      });
    },
    //点击提交按钮， 提交成功返回上一页
    async commit() {
      if (!this.value.trim()) {
        uni.showToast({
          title: "请输入反馈信息",
          icon: "none"
        });
        return;
      }
      if (!this.uploadImgs.length) {
        uni.showToast({
          title: "请上传反馈图片",
          icon: "none"
        });
        return;
      }
      // 发送请求
      this.checkedInfo.feedbackPicture = this.uploadImgs.join(",");
      this.checkedInfo.feedbackDescribe = this.value;
      let res = await request("/sendfeedback", { ...this.checkedInfo });
      if (res.data.code == 200) {
        uni.showToast({
          title: "提交成功",
          icon: "success"
        });
        uni.navigateBack({
          delta: 1
        });
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.container {
  height: 100%;
  display: flex;
  flex-direction: column;
  .title {
    height: 80rpx;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #e6f3f9;
    .date {
      font-size: 28rpx;
    }
  }
  .content {
    width: 100%;
    flex: 1;
    padding: 0 10rpx;
    display: flex;
    flex-direction: column;
    .checkInfo {
      flex: 4;
      width: 100%;
      .checked_title {
        margin: 20rpx 0;
        .icon {
          color: red;
        }
        .tip {
          color: #666;
          font-size: 26rpx;
        }
      }
      .des_content {
        height: 25%;
        padding: 20rpx;
        font-size: 30rpx;
        border: 1rpx solid #ccc;
        border-radius: 10rpx;
        display: flex;
        flex-wrap: wrap;
      }
      .pic_content {
        width: 100%;
        height: 35%;
        border: 1rpx solid #ccc;
        border-radius: 10rpx;
        display: flex;
        justify-content: space-evenly;
        align-items: center;
        .checkedImg_item {
          width: 120rpx;
          height: 120rpx;
          image {
            width: 100%;
            height: 100%;
            border-radius: 15rpx;
          }
        }
      }
    }
    .feedbackInfo {
      flex: 4;
      width: 100%;
      .fb_title {
        margin: 20rpx 0;
        .icon {
          color: red;
        }
        .tip {
          color: #666;
          font-size: 26rpx;
        }
      }
      .des_content {
        width: 100%;
        height: 25%;
        border: 1rpx solid #ccc;
        border-radius: 10rpx;
        font-size: 30rpx;
        padding: 20rpx;
        textarea {
          width: 100%;
          height: 100%;
        }
      }
      .fb_imges {
        width: 100%;
        height: 35%;
        display: flex;
        align-items: center;
        padding-left: 60rpx;
        border: 1rpx solid #ccc;
        border-radius: 10rpx;
        button {
          width: 120rpx;
          height: 120rpx;
          padding: 0;
          margin: 0;
          display: flex;
          justify-content: center;
          align-items: center;
          font-size: 60rpx;
          color: #ccc;
        }
        .img_wrap {
          margin-left: 40rpx;
        }
      }
    }
    .btn_wrap {
      margin-top: 20rpx;
      flex: 1;
      width: 100%;
      display: flex;
      justify-content: center;
      button {
        width: 60%;
        height: 90rpx;
        font-size: 32rpx;
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 0;
      }
    }
  }
}
</style>