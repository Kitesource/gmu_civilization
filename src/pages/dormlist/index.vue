<template>
  <view class="indexContainer">
    <!-- 下拉选择输入框选择栋数、层数 -->
    <view class="title">
      <view class="select">
        <uni-combox
          :candidates="Tungs"
          placeholder="请选择栋数"
          v-model="tung"
        ></uni-combox>
        <uni-combox
          :candidates="layers"
          v-model="layer"
          placeholder="请选择层数"
        ></uni-combox>
      </view>
      <view class="btnContainer">
        <button size="mini" @click="handleConfirm">确定</button>
      </view>
    </view>

    <!-- 详细寝室展示区域 -->
    <view class="dorm_detail">
      <!-- 默认信息 -->
      <view class="default" v-if="!dormList.length">
        <p>Please select information first...</p>
      </view>
      <!-- 寝室信息 -->
      <view class="dormList" v-else>
        <view class="title">{{ tung }}栋{{ layer }}层寝室信息</view>
        <scroll-view class="scrollView" scroll-y>
          <view
            :class="['doremItem', item.ischeck == 'check' ? 'checkColor' : '']"
            hover-class="Active"
            v-for="(item, index) in dormList"
            :key="item.id"
            @click="handleToDorm"
            :data-id="index"
            :data-dormNum="item.dormNum"
            :data-ischeck="item.ischeck"
          >
            {{ item.dormNum }}
            <text :class="item.ischeck == 'check' ? 'checked' : ''"></text>
          </view>
        </scroll-view>
      </view>
    </view>
  </view>
</template>

<script>
import uniCombox from "@dcloudio/uni-ui/lib/uni-combox/uni-combox.vue";
import request from "../../utils/request";
export default {
  components: {
    uniCombox
  },
  data() {
    return {
      tung: "6", //选取的栋数
      layer: "5", //选取的层数
      flag: false,
      dormList: [], //请求过来的寝室列表信息
      collegeList: [], //请求返回的学院信息
      dormnum: "", //选取的寝室id
      className: "", //请求返回的班级信息
      Tungs: ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12"],
      layers: ["1", "2", "3", "4", "5", "6"]
    };
  },
  onLoad() {},
  onShow() {
    if (this.tung && this.layer) {
      this.getDormData();
    }
  },
  methods: {
    //获取寝室列表信息
    async getDormData() {
      let result = await request("/alldormnum", {
        tung: this.tung,
        layer: this.layer
      });
      console.log(result);
      if (!result.data.data2.length) {
        uni.showToast({
          title: "查询失败,请重新选择~",
          icon: "none"
        });
        return;
      }
      this.dormList = result.data.data2;
      this.collegeList = result.data.data;
    },
    //点击确定查找寝室列表
    handleConfirm() {
      if (!this.tung && !this.layer) {
        //未选择栋数和楼层
        uni.showToast({
          title: "请先选择栋数和层数",
          icon: "none"
        });
        return;
      } else if (!this.tung) {
        uni.showToast({
          title: "请先选择栋数",
          icon: "none"
        });
        return;
      } else if (!this.layer) {
        uni.showToast({
          title: "请先选择层数",
          icon: "none"
        });
        return;
      } else{
        //已选择
        this.getDormData();
      }
    },
    //点击寝室号跳转到寝室评价表单页面
    handleToDorm(e) {
      let { ischeck, dormnum, id } = e.currentTarget.dataset;
      if (ischeck == "check") {
        wx.showToast({
          title: "该寝室已完成检查！",
          icon: "none"
        });
        return;
      }
      let className = this.dormList[id].className;
      const _this = this;
      wx.navigateTo({
        url: "/pages/checkform/index",
        success: function(res) {
          // 通过eventChannel向被打开页面传送数据
          res.eventChannel.emit("acceptDataFromOpenerPage", {
            tung: _this.tung,
            layer: _this.layer,
            collegeList: _this.collegeList,
            dormnum,
            className
          });
        }
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.indexContainer {
  flex-direction: column;
  .title {
    display: flex;
    .select {
      flex: 8;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
    }
    .btnContainer {
      flex: 2;
      display: flex;
      align-items: center;
    }
  }
  .dorm_detail {
    height: calc(100vh - 40px);
    .default {
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #c2c2c2;
      font-size: 32rpx;
      font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS",
        sans-serif;
    }
    .dormList {
      .title {
        background-color: #e6f3f9;
        height: 80rpx;
        display: flex;
        justify-content: center;
        align-items: center;
      }
      .scrollView {
        height: calc(100vh - 80px);
        .checkColor {
          color: gray;
        }
        .doremItem {
          height: 120rpx;
          display: flex;
          justify-content: center;
          align-items: center;
          /* 完成查寝后的 √ 样式*/
          text {
            height: 15rpx;
            width: 25rpx;
            display: inline-block;
          }
          .checked {
            border: 1rpx solid #09bb07;
            border-width: 0 0 2px 2px;
            transform: rotate(-45deg);
            -ms-transform: rotate(-45deg);
            -moz-transform: rotate(-45deg);
            -webkit-transform: rotate(-45deg);
            -o-transform: rotate(-45deg);
            vertical-align: baseline;
          }
        }
      }
    }
  }
}
</style>
