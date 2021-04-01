<template>
  <div class="indexContainer">
    <img v-if="isShow" class="index_img" :src="userInfo.avatarUrl" alt="">
    <Button class="btn" v-else open-type="getUserInfo" @getuserinfo="getuinfo">点击获取用户信息</Button>
    <p class="userName">hello {{ userInfo.nickName }}</p>
    <div class="goStudy">
      <navigator url="/pages/imagetransfer/main">点击开启小程序之旅</navigator>
    </div>
  </div>
</template>

<script>
export default {
  name: 'index',
  data () {
    return {
      userInfo: {},
      isShow: false // 用户没有授权
    }
  },
  onLoad: function (options) {
    console.log('---onLoad---')
  },
  beforeMount: function (options) {
    console.log('---beforeMount---')
    this.handlerGetUserInfo()
  },
  mounted: function (options) {
    console.log('---mounted---')
  },
  methods: {
    // 获取用户登录信息
    handlerGetUserInfo () {
      wx.getUserInfo({
        success: (data) => {
          console.log(data)
          // 更新data中的数据
          this.userInfo = data.userInfo
          this.isShow = true
        },
        fail: () => {
          console.log('获取失败')
        }
      })
    },
    getuinfo (data) {
      // 判断用户是否允许授权
      if (data.mp.detail.rawData) {
        // 用户授权
        this.handlerGetUserInfo()
      }
    },
    startBtn () {
      const url = '../imagetransfer/main'
      // switchTab navigateTo
      wx.navigateTo({url})
    }
  }
}
</script>

<style>

page {
  background: #009966;
}

.indexContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
}

/*
.index_img {
  overflow: hidden;
  display: block;
  width: 160 rpx;
  height: 160 rpx;
  margin: 20 rpx;
  margin-top: 50 rpx;
  border-radius: 50%;
  border: 2px solid #fff;
  box-shadow: 3px 3px 10px rgba(0, 0, 0, 0.2);
}
*/

.index_img {
  height: 50px;
  width: 50px;
  border-radius: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.userName {
  font-size: 40 rpx;
  font-weight: bold;
  margin: 100 rpx 0;
}

.goStudy {
  width: 220 rpx;
  height: 80 rpx;
  border: 1 rpx solid #eee;
  font-size: 24 rpx;
  line-height: 80 rpx;
  text-align: center;
  border-radius: 10 rpx;
}

.btn {
  width: 100 rpx;
  height: 100 rpx;
  border-radius: 50%;
  margin: 50 rpx 0;
  line-height: 300 rpx;
  text-align: center;
  font-size: 26 rpx;
}
</style>
