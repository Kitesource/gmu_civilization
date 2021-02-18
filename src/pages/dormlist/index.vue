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
      <!-- <view class="default" uni:if="{{!dormList.length}}">
      <view>Please select information first...</view>
    </view> -->
      <!-- 寝室信息 -->
      <view class="dormList">
        <view class="title">{{ tung }}栋{{ layer }}层寝室信息</view>
        <scroll-view class="scrollView" scroll-y>
          <navigator
          v-for="item in dormList"
          :key="item.id"
          >{{item.dormNum}}
          </navigator>
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
    uniCombox,
  },
  data() {
    return {
      tung: "", //选取的栋数
      layer: "", //选取的层数
      dormList: [], //请求过来的寝室列表信息
      collegeList: [], //请求返回的学院信息
      dormId: "", //选取的寝室id
      className: "", //请求返回的班级信息
      Tungs: ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12"],
      layers: ["1", "2", "3", "4", "5", "6"],
    };
  },
  onLoad() {
    console.log(getApp().globalData.mssage);
  },
  onShow() {},
  methods: {
    //获取寝室列表信息
    async getDormData() {
      let result = await request("/alldormnum", {
        tung: this.tung,
        layer: this.layer,
      });
      console.log(result);
      if (!result.data.data2.length) {
        uni.showToast({
          title: "该层无学生寝室,请重新选择楼层~",
          icon: "none",
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
          icon: "none",
        });
        return;
      } else if (!this.tung) {
        uni.showToast({
          title: "请先选择栋数",
          icon: "none",
        });
        return;
      } else if (!this.layer) {
        uni.showToast({
          title: "请先选择层数",
          icon: "none",
        });
        return;
      } else {
        //已选择
        this.getDormData();
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.indexContainer {
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
    .default {
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
        height: calc(100vh - 74px);
        navigator {
          height: 120rpx;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
    }
  }
}
</style>
