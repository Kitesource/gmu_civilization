<template>
  <view class="container">
    <uni-search-bar
      class="uniSearch"
      :radius="100"
      :maxlength="10"
      placeholder="请输入合格/不合格"
      @input="handleInput"
      @confirm="handleSearch"
      @cancel="handleCancel"
    ></uni-search-bar>
    <view class="scroll_title">
      <!-- <text>查寝日期</text>
      <text>是否合格</text>
      <text>是否已读</text> -->
      <uni-tag text="查寝日期" type="primary" :inverted="true"></uni-tag>
      <uni-tag text="是否合格" type="primary" :inverted="true"></uni-tag>
      <uni-tag text="是否已读" type="primary" :inverted="true"></uni-tag>
    </view>
    <scroll-view scroll-y class="sroll_content">
      <view
        class="sroll_item"
        hover-class="Active"
        @click="handleToFeedback"
        :data-id="item.id"
        v-for="item in dormInfo"
        :key="item.id"
      >
        <view class="item">
          <uni-tag :text="item.checkTime" type="primary"></uni-tag>
        </view>
        <view class="item">
          <uni-tag
            v-if="item.state == '合格'"
            text="合格"
            type="success"
          ></uni-tag>
          <uni-tag v-else text="不合格" type="error"></uni-tag>
        </view>
        <view class="item">
          <uni-tag
            v-if="item.stuReadIt == 'read'"
            text="已读"
            type="default"
          ></uni-tag>
          <uni-tag v-else text="未读" type="warning"></uni-tag>
        </view>
      </view>
    </scroll-view>
  </view>
</template>

<script>
import uniSearchBar from "@dcloudio/uni-ui/lib/uni-search-bar/uni-search-bar.vue";
import uniTag from "@dcloudio/uni-ui/lib/uni-tag/uni-tag.vue";
import request from "../../utils/request";
export default {
  components: {
    uniSearchBar,
    uniTag
  },
  data() {
    return {
      username: "",
      id:"",
      dateList: [], //查寝日期数组
      dormInfo: [], //查寝信息数组
      queryObj: {}, //修改已读状态所需的参数
      value: "", //搜索框输入内容
      searchObj: {} //搜索参数对象
    };
  },
  //监听页面加载
  onLoad(options) {
    this.username = options.username;
    this.getCheckList();
  },
  onShow() {
    this.getCheckList();
  },
  methods: {
    //获取查寝记录的方法
    async getCheckList() {
      let res = await request("/stuDorm", { stunum: this.username });
      this.dateList = res.data.data;
      this.dormInfo = res.data.data2;
    },
    //监听搜索框的输入
    handleInput(e) {
      let value = e.value;
      if (value.trim() === "合格") {
        this.value = "qualified";
      } else if (value.trim() === "不合格") {
        this.value = "unqualified";
      } else {
        this.value = "";
      }
    },
    //回车确认搜索
    async handleSearch(e) {
      if (!this.value || !this.value.trim()) {
        //输入不合法
        wx.showToast({
          title: "输入不合法",
          icon: "none"
        });
        return;
      }
      this.searchObj.state = this.value;
      this.searchObj.dormNum = this.dormInfo[0].dormNum;
      this.searchObj.check_time = "";
      this.searchObj.currentPage = 1;
      this.searchObj.pageSize = 10;
      let res = await request("/findDorm", { ...this.searchObj });
      this.dormInfo = res.data.data2;
    },
    //点击取消
    async handleCancel() {
      this.getCheckList();
    },
    //点击跳转到反馈页面 发送请求修改是否已读状态
    async handleToFeedback(e) {
      this.id  = e.currentTarget.dataset.id;
      uni.navigateTo({
        url: `/pages/feedback/index?username=${this.username}&id=${this.id}`
      });
      this.dormInfo.some(item => {
        if (item.id == this.id) {
          this.queryObj = item;
        }
      });
      await request("/changeStateStu", { ...this.queryObj });
    }
  }
};
</script>

<style lang="scss" scoped>
.container {
  height: 100%;
  .uniSearch {
    width: 100%;
    height: 80rpx;
  }
  .scroll_title {
    height: 80rpx;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-around;
    border-bottom: 1rpx solid #666;
  }
  .sroll_content {
    flex: 1;
    padding-bottom: 10rpx;
    height: calc(100vh - 192rpx);
    .sroll_item {
      height: 100rpx;
      width: 100%;
      display: flex;
      align-items: center;
      border-bottom: 1px solid #ccc;
      .item {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        .unqualified {
          color: red;
        }
        .readColor {
          color: #666;
        }
      }
    }
  }
}
</style>
