<template>
  <view class="container">
    <!-- 下拉框选择 -->
    <view class="title">
      <view class="select">
        <uni-combox :candidates="classes" placeholder="请选择班级" v-model="className"></uni-combox>
        <uni-combox :candidates="states" placeholder="请选择状态" v-model="state"></uni-combox>
      </view>
      <view class="btnContainer"><button size="mini" @click="handleConfirm">确认</button></view>
    </view>
    <!-- 寝室列表 -->
    <view class="content">
      <!-- 默认信息 -->
      <view class="default" v-if="!recordList.length">
        <p>Please select information first...</p>
      </view>
      <!-- 寝室信息 -->
      <view class="dormList" v-else>
        <view class="title">按{{state}}降序排列</view>
        <scroll-view class="scrollView" scroll-y>
          <view
            class='dormItem'
            hover-class="Active"
            v-for="item in recordList"
            :key="item.code"
            @click="handleToRecord"
            :data-dormnum = "item.data2"
          >
            {{ item.data2}}
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
      states: ["优秀", "良好", "一般", "较差", "脏乱差"],
      college:'信息工程学院',
      classes:[],
      state: '',
      className:'',
      recordList:[], //查寝记录数组
    };
  },
  onLoad(args) {
    this.getClasses();
  },
  methods: {
    // 获取学院所有班级
    async getClasses() {
      let result = await request("/findBytime", {
        college:'信息工程学院',
        position:'辅导员'
      })
      this.classes = result.data.data;
    },
    // 点击确定获取查寝信息
    async handleConfirm() {
      if(this.className || this.state){
      let result = await request("/getStateAndCount", {
        college:this.college,
        className:this.className,
        state:this.state
      })
      if(!result.data.data2){
        uni.showToast({
          title: '暂无数据',
          icon: 'none'
        })
        this.recordList = [];
        return;
      }
      const arr = result.data.data2;
      this.recordList = arr.sort(this.handleSort('message'));
      }else{
        uni.showToast({
          title: '请至少选择一个条件',
          icon: 'none'
        })
        return;
      }
    },
    // 降序函数
    handleSort(property){
      return function(a,b) {
        const val1 = a[property];
        const val2 = b[property];
        return val2 - val1;
      }
    },
    handleToRecord(e) {
      const {dormnum} = e.currentTarget.dataset;
      uni.navigateTo({
         url: `/pages/insRecordMsg/index?dormNum=${dormnum}&state=${this.state}&className=${this.className}`
      });
    }

  }
};
</script>

<style scoped lang="scss"> 
.container{
  .title{
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
  .content {
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
        .dormItem {
          height: 100rpx;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
    }
  }
}
</style>