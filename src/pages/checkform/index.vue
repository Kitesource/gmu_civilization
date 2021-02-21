<template>
  <!-- pages/form/form.wxml -->
  <view class="form_container">
    <view class="title">寝室:{{ dormnum }}</view>
    <form class="content">
      <!-- 下拉选择学院 -->
      <view class="selectCollege">
        <view><text class="icon">*</text>请选择学院:</view>
        <uni-combox
          class="combox"
          :candidates="collegeList"
          placeholder="请选择学院信息"
          v-model="college"
        ></uni-combox>
      </view>
      <!-- 查寝时间 -->
      <view class="currentTime">
        <text class="icon">*</text>
        <view class="time_title">查寝时间:</view>
        <text>{{ checkTime }}</text>
      </view>
      <!-- 选择是否合格 -->
      <view class="selection_group">
        <view class="section_title"><text class="icon">*</text>是否合格:</view>
        <radio-group name="evaluation">
          <label>
            <radio :checked="isSelected" @click="qualified" />
            合格
          </label>
          <label>
            <radio :checked="!isSelected" @click="unqualified" />
            不合格
          </label>
        </radio-group>
      </view>
      <!-- 合格区域 -->
      <block v-if="isSelected">
        <!-- 详细描述 -->
        <view class="des_group">
          <view class="des_title">合格描述:</view>
          <view class="textarea_container">
            <textarea
              @input="handleTextarea"
              placeholder="请输入..."
            ></textarea>
          </view>
        </view>
        <!-- 上传图片 -->
        <view class="img_group">
          <view class="img_title">合格图片:</view>
          <view class="img_wrap">
            <button type="default" @click="handleChooseImg">+</button>
            <view
              class="upimgWrap"
              v-for="(item, index) in chooseImgs"
              :key="item"
              @click="previewBigImg"
              @longpress="longpressImg"
              :data-index="index"
            >
              <upload-imgs :src="item"></upload-imgs>
            </view>
          </view>
        </view>
      </block>
      <!-- 不合格区域 -->
      <block v-else>
        <!-- 详细描述 -->
        <view class="des_group">
          <view class="des_title"><text class="icon">*</text>不合格描述:</view>
          <view class="textarea_container">
            <textarea
              :maxlength="50"
              @input="handleTextarea"
              placeholder="请输入..."
            ></textarea>
          </view>
        </view>
        <!-- 上传图片 -->
        <view class="img_group">
          <view class="img_title"><text class="icon">*</text>不合格图片:</view>
          <view class="img_wrap">
            <button type="default" @click="handleChooseImg">+</button>
            <view
              class="upimgWrap"
              v-for="(item, index) in unChooseImgs"
              :key="item"
              @click="previewBigImg"
              @longpress="longpressImg"
              :data-index="index"
            >
              <upload-imgs :src="item"></upload-imgs>
            </view>
          </view>
        </view>
      </block>
      <!-- 提交/返回 -->
      <view class="last_group">
        <button type="default" form-type="reset" @click="reset">重置</button>
        <button type="primary" @click="commit">提交</button>
      </view>
    </form>
  </view>
</template>

<script>
import uniCombox from "@dcloudio/uni-ui/lib/uni-combox/uni-combox.vue";
import UploadImgs from "../../components/UploadImgs/UploadImgs";
import util from "../../utils/util";
import request from "../../utils/request";
export default {
  components: { uniCombox, UploadImgs },
  data() {
    return {
      checkTime: "", //当前时间
      dormnum: "", //寝室id
      tung: "", //寝室栋数
      collegeList: [], //dorm页面发送来的学院信息数组
      className: "", //dorm页面发送来的班级信息
      college: "", //用户点击选择的学院
      isSelected: true, //判断是否合格
      state: "合格", //合格状态: '合格'/ '不合格', 默认合格
      qualifiedDescribe: "", //合格描述
      unqualifiedDescribe: "", //不合格描述
      chooseImgs: [], //被选中的合格图片路径数组
      unChooseImgs: [],
      uploadImgs: [], //上传后的合格图片路径
      unUploadImgs: [],
      qualifiedPicture: "", //需提交的图片路径参数
      unqualifiedPicture: ""
    };
  },

  //生命周期函数
  onLoad(options) {
    // 获取当前时间
    this.checkTime = util.formatTime(new Date());

    //接收dorm页发送的信息
    var _this = this;
    const eventChannel = this.getOpenerEventChannel();
    // 监听acceptDataFromOpenerPage事件，获取上一页面通过eventChannel传送到当前页面的数据
    eventChannel.on("acceptDataFromOpenerPage", function(data) {
      _this.tung = data.tung + "栋";
      _this.layer = data.layer;
      _this.dormnum = data.dormnum;
      _this.collegeList = data.collegeList;
      _this.className = data.className;
    });
  },
  methods: {
    //点击选择合格
    qualified() {
      this.isSelected = true;
      this.state = "合格";
      //切换后清空表单数据
      this.undescription = "";
      this.unChooseImgs = [];
      this.unUploadImgs = [];
    },
    //点击选择不合格
    unqualified() {
      this.isSelected = false;
      this.state = "不合格";
      //切换后清空表单数据
      this.description = "";
      this.ChooseImgs = [];
      this.UploadImgs = [];
    },
    //添加描述
    handleTextarea(e) {
      if (this.isSelected) {
        //合格描述可以为空
        this.qualifiedDescribe = e.detail.value;
      } else {
        //不合格描述不能为空
        this.unqualifiedDescribe = e.detail.value.trim();
      }
    },
    //点击+号， 选择图片并自动上传
    handleChooseImg() {
      if (this.chooseImgs.length > 2 || this.unChooseImgs.length > 2) {
        uni.showToast({
          title: "最多上传3张图片哦~",
          duration: 2000,
          icon: "none"
        });
        return;
      }
      const _this = this;
      uni.chooseImage({
        count: 1,
        sizeType: ["original", "compressed"],
        sourceType: ["album", "camera"],
        success: imageRes => {
          //获取本地存储的cookie
          let JSESSIONID = uni
            .getStorageSync("JSESSIONID")
            .split(";")[0]
            .split("=")[1];
          let cookie = "JSESSIONID=" + JSESSIONID;
          //将选择的图片保存在数组中
          if (_this.isSelected) {
            _this.chooseImgs = [..._this.chooseImgs, ...imageRes.tempFilePaths];
          } else {
            _this.unChooseImgs = [
              ..._this.unChooseImgs,
              ...imageRes.tempFilePaths
            ];
          }
          // 上传图片
          const tempFilePaths = imageRes.tempFilePaths;
          uni.uploadFile({
            url: "https://111.75.252.147/score/tupian",
            header: {
              "content-type": "multipart/form-data",
              cookie: cookie
            },
            filePath: tempFilePaths[tempFilePaths.length - 1],
            name: "file",
            success: res => {
              let url = JSON.parse(res.data).data;
              // 将上传图片后得到的路径保存在数组中
              if (_this.isSelected) {
                _this.uploadImgs.push(url);
              } else {
                _this.unUploadImgs.push(url);
              }
            }
          });
        }
      });
    },
    //点击图片预览大图
    previewBigImg(e) {
      const _this = this;
      if (this.isSelected) {
        let path = e.currentTarget.dataset.id;
        uni.previewImage({
          current: path, // 当前显示图片的http链接
          urls: _this.chooseImgs // 需要预览的图片http链接列表
        });
      } else {
        let path = e.currentTarget.dataset.id;
        uni.previewImage({
          current: path, // 当前显示图片的http链接
          urls: _this.unChooseImgs // 需要预览的图片http链接列表
        });
      }
    },
    //长按图片选择是否删除
    longpressImg(e) {
      const _this = this;
      const index = e.currentTarget.dataset.index; //获取当前长按图片下标
      if (this.isSelected) {
        uni.showModal({
          title: "提示",
          content: "确认要删除该图片吗?",
          success: function(res) {
            if (res.confirm) {
              _this.chooseImgs.splice(index, 1);
              _this.uploadImgs.splice(index,1);
            } else if (res.cancel) {
              return false;
            }
          }
        });
      } else {
        uni.showModal({
          title: "提示",
          content: "确认要删除该图片吗?",
          success: function(res) {
            if (res.confirm) {
              _this.unChooseImgs.splice(index, 1);
              _this.unUploadImgs.splice(index, 1)
            } else if (res.cancel) {
              return false;
            }
          }
        });
      }
    },
    //点击重置按钮
    reset() {
      this.college='';
      this.chooseImgs = [];
      this.unChooseImgs = [];
    },
    //点击提交表单数据
    commit() {
      const _this = this;
      // 将添加的图片路径从数组中获取出来
      if (this.uploadImgs.length) {
        this.qualifiedPicture = this.uploadImgs.join(",");
      } else {
        this.unqualifiedPicture = this.unUploadImgs.join(",");
      }
      //是否选择了学院
      if (this.college == "") {
        uni.showToast({
          title: "请先选择学院~",
          icon: "none"
        });
        return;
      }

      // 发送请求
      if (this.isSelected) {
        //合格
        if (this.chooseImgs.length != 0 && this.uploadImgs.length == 0) {
          uni.showToast({
            title: "请先上传添加的图片",
            icon: "none"
          });
          return;
        }
        uni.showModal({
          title: "提示",
          content: "确认提交吗？",
          success: async function(res) {
            if (res.confirm) {
              //用户点击确定
              let result = await request("/insertcheckdorm", {
                tung: _this.tung,
                dormNum: _this.dormnum,
                checkTime: _this.checkTime,
                college: _this.college,
                className: _this.className,
                state: _this.state,
                qualifiedDescribe: _this.qualifiedDescribe,
                unqualifiedDescribe: "",
                qualifiedPicture: _this.qualifiedPicture,
                unqualifiedPicture: ""
              });
              if (result.data.code === 200) {
                uni.showToast({
                  title: "提交成功~",
                  icon: "success"
                });
              } else {
                uni.showToast({
                  title: "提交失败~",
                  icon: "fail"
                });
              }
              //提交成功后跳转到寝室列表页面
              uni.navigateBack();
            } else if (res.cancel) {
              // 用户点击取消
              return;
            }
          }
        });
      } else {
        //不合格，需要描述和上传图片
        if (!this.unqualifiedDescribe) {
          uni.showToast({
            title: "请先添加描述！",
            icon: "none"
          });
          return;
        }
        if (!this.unUploadImgs) {
          uni.showToast({
            title: "请先添加图片！",
            icon: "none"
          });
          return;
        }
        uni.showModal({
          title: "提交",
          content: "确认提交吗？",
          success: async function(res) {
            if (res.confirm) {
              //用户点击确定
              let result = await request("/insertcheckdorm", {
                tung: _this.tung,
                dormNum: _this.dormnum,
                checkTime: _this.checkTime,
                college: _this.college,
                className: _this.className,
                state: _this.state,
                qualifiedDescribe: "",
                unqualifiedDescribe: _this.unqualifiedDescribe,
                qualifiedPicture: "",
                unqualifiedPicture: _this.unqualifiedPicture
              });
              if (result.data.code === 200) {
                uni.showToast({
                  title: "提交成功~",
                  icon: "success"
                });
                //
              } else {
                uni.showToast({
                  title: "提交失败~",
                  icon: "fail"
                });
              }
              //提交成功后跳转到寝室列表页面
              uni.navigateBack();
            } else if (res.cancel) {
              // 用户点击取消
              return;
            }
          }
        });
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.form_container {
  height: 100%;
  display: flex;
  flex-direction: column;
  .title {
    line-height: 80rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: azure;
  }
  .content {
    .icon {
      color: red !important;
    }
    flex: 1;
    padding: 10rpx 20rpx;
    .selectCollege {
      margin-bottom: 20rpx;
      display: flex;
      align-items: center;
      view {
        color: #000;
        flex: 1.4;
      }
      .combox {
        flex: 3;
      }
    }
    .currentTime {
      margin-bottom: 20rpx;
      display: flex;
      .time_title {
        margin-right: 20rpx;
      }
    }
    .selection_group {
      height: 100rpx;
      line-height: 100rpx;
      width: 100%;
      display: flex;
      .section_title {
        margin-right: 10rpx;
      }
      label {
        margin-left: 10rpx;
      }
    }
    /* 描述区域 */
    .des_group {
      width: 100%;
      border-bottom: 20rpx;
      .des_title {
        width: 100%;
        height: 80rpx;
        line-height: 80rpx;
      }
      .textarea_container {
        padding: 30rpx;
        box-sizing: border-box;
        font-size: 32rpx;
        height: 200rpx;
        border: 1px solid #ccc;
        border-radius: 10rpx;
        textarea {
          width: 100%;
          height: 100%;
        }
      }
    }
    /* 上传图片区域 */
    .img_group {
      width: 100%;
      margin-bottom: 20rpx;
      .img_title {
        width: 100%;
        height: 80rpx;
        line-height: 80rpx;
      }
      .img_wrap {
        height: 240rpx;
        border: 1rpx solid #ccc;
        border-radius: 10rpx;
        padding-left: 30rpx;
        display: flex;
        align-items: center;
        button {
          width: 120rpx;
          height: 120rpx;
          color: #ccc;
          font-size: 60rpx;
          text-align: center;
          line-height: 120rpx;
          margin: 0;
          padding: 0;
        }
        .upimgWrap {
          margin-left: 30rpx;
        }
      }
    }
    /* 提交/返回区域 */
    .last_group {
      margin-top: 30rpx;
      font-size: 28rpx;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-around;
      button {
        width: 60%;
        margin: 10rpx 0;
        padding: 0;
        font-size: 32rpx;
      }
    }
  }
}
</style>
